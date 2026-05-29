# Schema

Contact: <vincent.gardeux@epfl.ch>

Document Status: *Drafting*

Version: 7.1.0+scfair1.0

Current schema: **analysis_json**

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED" "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14), [RFC2119](https://www.rfc-editor.org/rfc/rfc2119.txt), and [RFC8174](https://www.rfc-editor.org/rfc/rfc8174.txt) when, and only when, they appear in all capitals, as shown here.

## Schema split

The scFAIR schema is split into multiple parts, to differentiate the metadata specific to certain modalities:

- The **core** schema ['schema.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md) is the main schema for all types of single-cell data (scRNA-seq, scATAC-seq, perturbation, spatial, ...). It describes the core metadata between all modalities.
- This **analysis_json** schema ['schema_analysis_json.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md) describes the analysis pipeline, including software and versioning (cellranger, seurat, scanpy, ...), methods and parameters (umap, louvain, ...), and Docker/conda environments for reproducibility.
- The **spatial** schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md) describes the additional metadata that are specific to spatial datasets (Visium)
- The **perturb** schema ['schema_perturb.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md) describes the additional metadata that are specific to perturbation datasets (CRISPR screens, perturb-seq, ...)
- The **atac** schema ['schema_atac.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md) describes the additional metadata that are specific to scATAC datasets (scATAC-seq, multiomics)


## Overview

This schema describes the analysis pipeline applied to a single-cell dataset. It describes, in execution order, all the tools, parameters, and versions that were used to generate the results. For each step, a Docker image or a conda environment can be referenced to maximize reproducibility.

This JSON is stored as a UTF-8-encoded string in `uns["analysis_pipeline"]` of the h5ad file, as described in the [core schema](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#analysis_pipeline).

This document is organized by:

- [General requirements](#general-requirements)
- [Top-level fields](#top-level-fields), which describe the pipeline as a whole
- [`steps` (Pipeline steps)](#steps-pipeline-steps), an ordered array of individual analysis steps, each containing:
  - [Step identification](#step-identification) — label, category, and human-readable description
  - [Step execution](#step-execution) — method name, exact command, tool version, and programming language
  - [Step environment](#step-environment) — Docker container image or conda environment for reproducibility
  - [`parameters`](#step-parameters) — ordered list of arguments and named settings passed to the method
  - [`inputs`](#step-inputs) — list of input files and/or h5ad metadata slots consumed by this step
  - [`outputs`](#step-outputs) — list of output files and/or h5ad metadata slots produced by this step
  - [Step resources](#step-resources) — computational resources used (CPU, RAM, GPU)
  - [Execution metadata](#execution-metadata) — timestamp, duration, and random seed
- [Appendix A: Full example JSON](#appendix-a-full-example-json)
- [Appendix B: Changelog](#appendix-b-changelog)


## General Requirements

**JSON.** The canonical format adopted by scFAIR for storing all analyses is JSON (RFC 8259). The document MUST be valid JSON, encoded in UTF-8 without BOM. When embedded in an h5ad file, it is stored as a single UTF-8 string in `uns["analysis_pipeline"]`.

**Reproducibility flag.** The `command` field of each step is the primary indicator of whether a step is reproducible. If `command` is `null`, the step is considered **non-reproducible**, and all environment and execution fields (`docker_image_name`, `docker_image_url`, `docker_image_digest`, `conda_env_url`, `conda_env_file`, `parameters`, `inputs`, `outputs`, etc.) MAY also be `null`. If `command` is a non-null string, all environment fields SHOULD be filled to maximize reproducibility.

**Ordering.** The `steps` array is ordered: steps MUST be listed in the chronological order they were applied to the data, from first to last.

**Reserved names.** The names of fields specified by the schema are **reserved**. Custom fields MAY be added to any object as additional key-value pairs, but MUST NOT start with `"__"` and MUST NOT conflict with reserved field names.

**Types.** All types are standard JSON types: `string`, `number`, `integer`, `boolean`, `array`, `object`, `null`.

**Dates.** All date/time values MUST follow ISO 8601 format: `YYYY-MM-DDTHH:MM:SSZ` (UTC preferred) or with an explicit timezone offset (e.g. `YYYY-MM-DDTHH:MM:SS+01:00`).

**Unique step labels.** The `step_label` value SHOULD be unique within the `steps` array, to allow unambiguous cross-referencing between steps (e.g. in `inputs[*].location`).


## Top-level fields

The top-level JSON object MUST be an `object` containing the following fields:

### schema_version

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>schema_version</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The version of the scFAIR <code>analysis_json</code> schema used to generate this document. MUST match the version string of the core schema. Example: <code>"7.1.0+scfair1.0"</code>.</td>
    </tr>
</tbody></table>

### pipeline_name

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>pipeline_name</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A short, human-readable name for the pipeline. Example: <code>"ASAP"</code>, <code>"Seurat"</code> or <code>"Scanpy"</code>.</td>
    </tr>
</tbody></table>

### pipeline_version

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>pipeline_version</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The version of the pipeline that was run (if existing). SHOULD follow <a href="https://semver.org/">Semantic Versioning</a>. Should be <code>null</code> if not versioned. Example: <code>"v5"</code>, <code>"1.3.2"</code>. Could be the version of the Seurat/Scanpy package used, the version of the online tool, or the version of the snakemake/nextflow pipeline.</td>
    </tr>
</tbody></table>

### pipeline_description

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>pipeline_description</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A free-text description of the overall pipeline and its purpose. Example: <code>"Standard preprocessing and clustering pipeline for 10x Chromium scRNA-seq data."</code>.</td>
    </tr>
</tbody></table>

### pipeline_url

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>pipeline_url</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A URL pointing to the source code, documentation, or publication describing the pipeline. MUST be a valid URL if provided. Example: <code>"https://github.com/my-lab/my-pipeline"</code>.</td>
    </tr>
</tbody></table>

### creation_date

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>creation_date</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The date and time at which this JSON document was generated, in ISO 8601 format. Example: <code>"2024-01-15T10:30:00Z"</code>.</td>
    </tr>
</tbody></table>

### steps

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>steps</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>array</code> of <code>object</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>An ordered array of step objects describing each analysis step. Steps MUST appear in chronological execution order. Each element is a step object as described in the <a href="#steps-pipeline-steps"><code>steps</code> (Pipeline steps)</a> section. The array MAY be empty (<code>[]</code>) if no steps are documented.</td>
    </tr>
</tbody></table>

## `steps` (Pipeline steps)

`steps` is a JSON Array of step objects. Each step object describes one analysis run or computational step in the pipeline. Steps are **ordered**: the position of a step in the array (0-indexed) reflects its execution order.

### Step identification

#### step_label

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>step_label</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A short, human-readable label for the step. SHOULD be unique within the <code>steps</code> array, as it may be used for cross-referencing. Example: <code>"Parsing"</code>, <code>"Quality Control"</code>, <code>"UMAP"</code>, <code>"Leiden Clustering"</code>.</td>
    </tr>
</tbody></table>

#### step_description

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>step_description</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A free-text description of what this step does. Example: <code>"Filters cells with fewer than 200 genes expressed and removes cells with more than 20% mitochondrial gene expression."</code>.</td>
    </tr>
</tbody></table>

#### step_category

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>step_category</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A controlled term describing the broad category of the analysis step (inspired from <a href="https://www.sc-best-practices.org">sc-best-practices.org</a>). The vocabulary is organized into thematic groups below (groups are informational only; the value is a flat string).<br/>
        MUST be one of the following values, or <code>"Other"</code> if no term applies:<br/><br/>
      <strong>Raw data processing</strong><br/>
      <code>"Alignment"</code> — Read alignment / mapping to a reference genome (e.g. STAR, HISAT2, CellRanger).<br/>
      <code>"Quantification"</code> — Generating a count matrix from aligned reads (e.g. STARsolo, featureCounts, salmon alevin). Should be <code>"Alignment"</code> if bundled with Alignment in all-in-one tools.<br/>
      <code>"Parsing"</code> — Loading and parsing tool output files (e.g. CellRanger output directory, MTX/H5 files) into an in-memory data structure. Could be important if number formats are changing for example.<br/>
      <code>"Report Generation"</code> — Generating automated QC or analysis reports (e.g. MultiQC, FastQC, CellRanger HTML report).<br/><br/>
      <strong>Quality control and filtering</strong><br/>
      <code>"Quality Control"</code> — Computing per-cell and per-gene QC metrics (e.g. number of genes, UMI counts, mitochondrial fraction).<br/>
      <code>"Ambient RNA Removal"</code> — Removing ambient / background RNA contamination from droplet-based data (e.g. SoupX, CellBender, DecontX).<br/>
      <code>"Doublet Detection"</code> — Detecting and/or removing doublet or multiplet cell captures (e.g. Scrublet, DoubletFinder, scDblFinder).<br/>
      <code>"Cell Filtering"</code> — Filtering out low-quality, dying, or otherwise unwanted cells based on QC metrics or other criteria.<br/>
      <code>"Gene Filtering"</code> — Filtering out lowly detected or uninformative genes (e.g. genes expressed in fewer than N cells).<br/><br/>
      <strong>Normalization and imputation</strong><br/>
      <code>"Normalization"</code> — Normalizing raw counts to remove sequencing-depth differences between cells (e.g. CP10K, scran, SCTransform, DESeq2 size factors).<br/>
      <code>"Scaling"</code> — Scaling and/or centering normalized expression values across genes (e.g. z-score standardization prior to PCA). Should be <code>"Normalization"</code> if bundled with Normalization in all-in-one tools such as SCTransform.<br/>
      <code>"Imputation"</code> — Imputing missing or zero (dropout) expression values to denoise the count matrix (e.g. MAGIC, SAVER, scImpute, DeepImpute).<br/><br/>
      <strong>Cell cycle and feature selection</strong><br/>
      <code>"Cell Cycle Scoring"</code> — Assigning cell cycle phase scores and/or regressing out cell cycle effects (e.g. Seurat CellCycleScoring, scanpy score_genes_cell_cycle).<br/>
      <code>"Feature Selection"</code> — Selecting the most informative features (e.g. highly variable genes) for downstream analysis.<br/><br/>
      <strong>Dimensionality reduction and embedding</strong><br/>
      <code>"Dimensionality Reduction"</code> — Reducing the feature space to a lower-dimensional representation (e.g. PCA, LSI, NMF, diffusion maps).<br/>
      <code>"Dimensionality Evaluation"</code> — Determining the number of statistically significant dimensions to retain (e.g. Jackstraw test, elbow plot, scree plot, permutation-based tests).<br/>
      <code>"Embedding"</code> — Computing a low-dimensional (2D/3D) embedding for visualization (e.g. UMAP, t-SNE, ForceAtlas2, diffusion map layout).<br/><br/>
      <strong>Clustering and annotation</strong><br/>
      <code>"Clustering"</code> — Unsupervised partitioning of cells into clusters (e.g. Leiden, Louvain, k-means, hierarchical clustering).<br/>
      <code>"Marker Gene Detection"</code> — Identifying genes that are specifically or differentially expressed within each cluster or cell type.<br/>
      <code>"Cell Type Annotation"</code> — Assigning cell type labels to clusters or individual cells, manually or using automated tools (e.g. SingleR, scANVI, Celltypist, manual curation).<br/>
      <code>"Subsetting"</code> — Subsetting the dataset to a specific cell population for focused downstream analysis (e.g. sub-clustering a lineage).<br/><br/>
      <strong>Batch correction and integration</strong><br/>
      <code>"Batch Correction"</code> — Correcting for technical batch effects between samples, experiments, or platforms (e.g. Harmony, ComBat, MNN, scVI).<br/>
      <code>"Integration"</code> — Integrating data across samples, donors, datasets, or modalities into a joint representation (e.g. Seurat integration, scVI, LIGER, WNN).<br/><br/>
      <strong>Differential and compositional analysis</strong><br/>
      <code>"Differential Expression"</code> — Testing for genes that are differentially expressed between conditions, cell types, or time points (e.g. DESeq2, edgeR, FindMarkers, MAST).<br/>
      <code>"Pseudobulk Analysis"</code> — Aggregating single-cell counts into pseudobulk profiles per sample for robust differential analysis.<br/>
      <code>"Compositional Analysis"</code> — Analysing changes in cell type proportions across conditions (e.g. scCODA, propeller, DA-seq).<br/>
      <code>"Gene Set Enrichment"</code> — Gene set enrichment, pathway, or Gene Ontology (GO) analysis on gene lists (e.g. GSEA, enrichR, clusterProfiler, fgsea).<br/>
      <code>"Perturbation Modeling"</code> — Modelling the transcriptional effect of genetic or chemical perturbations (e.g. scGen, CPA, CINEMA-OT).<br/><br/>
      <strong>Trajectory and dynamics</strong><br/>
      <code>"Trajectory Analysis"</code> — Inferring pseudotemporal orderings and differentiation trajectories (e.g. Monocle, PAGA, Palantir, scFates).<br/>
      <code>"RNA Velocity"</code> — Estimating the future transcriptional state of cells from spliced/unspliced ratios (e.g. scVelo, velocyto, UniTVelo).<br/>
      <code>"Lineage Tracing"</code> — Reconstructing cell lineages from clonal barcodes or somatic mutations (e.g. Cassiopeia, Starcode, LARRY).<br/><br/>
      <strong>Mechanisms and interactions</strong><br/>
      <code>"Cell Communication"</code> — Inferring cell-cell communication via ligand-receptor interactions (e.g. CellChat, NicheNet, LIANA, CellPhoneDB).<br/>
      <code>"Gene Regulatory Network"</code> — Inferring gene regulatory networks and transcription factor activity (e.g. SCENIC, pySCENIC, GRNBoost2, Dictys).<br/>
      <code>"Copy Number Variation"</code> — Inferring copy number variation from single-cell expression data to identify tumour cells (e.g. inferCNV, CopyKAT, SCEVAN).<br/>
      <code>"Deconvolution"</code> — Deconvolving bulk or spatial transcriptomics profiles into cell type fractions (e.g. RCTD, SPOTlight, BayesPrism, MuSiC).<br/><br/>
      <strong>Immune repertoire (TCR/BCR)</strong><br/>
      <code>"Immune Receptor Profiling"</code> — Profiling T-cell or B-cell receptor (TCR/BCR) sequences from V(D)J data (e.g. CellRanger V(D)J, Trust4).<br/>
      <code>"Clonotype Analysis"</code> — Analysing clonotype diversity, expansion, and sharing across samples (e.g. Scirpy, immunarch).<br/><br/>
      <strong>Chromatin accessibility (scATAC-seq)</strong><br/>
      <code>"Peak Calling"</code> — Calling chromatin accessibility peaks from scATAC-seq fragment data (e.g. MACS2, ArchR, Signac).<br/>
      <code>"Motif Analysis"</code> — Analysing transcription factor motif enrichment within accessibility peaks (e.g. chromVAR, HOMER, ArchR).<br/><br/>
      <strong>Spatial transcriptomics</strong><br/>
      <code>"Spatially Variable Genes"</code> — Identifying genes whose expression is spatially structured (e.g. SpatialDE, SPARK, Moran's I).<br/>
      <code>"Spatial Neighborhood Analysis"</code> — Analysing cellular co-localization and spatial interaction patterns (e.g. Squidpy, SPATA2).<br/>
      <code>"Spatial Domain Detection"</code> — Identifying spatially coherent tissue regions or domains (e.g. BayesSpace, STAGATE, GraphST).<br/>
      <code>"Spatial Deconvolution"</code> — Deconvolving cell type composition at each spatial spot (e.g. RCTD, SPOTlight, cell2location).<br/><br/>
      <strong>Other methods and plotting</strong><br/>
      <code>"Visualization"</code> — Generating analysis plots or figures not covered by another category (e.g. custom heatmaps, dot plots, violin plots).<br/>
      <code>"Other"</code> — Any analysis step not covered by the categories above.</td>
    </tr>
</tbody></table>

### Step execution

#### method

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>method</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The name of the method, script, function, or tool used for this step. Example: <code>"Parsing.java"</code>, <code>"scanpy.pp.normalize_total"</code>, <code>"Seurat::FindClusters"</code>, <code>"cellranger count"</code>, <code>"Manual curation"</code>.</td>
    </tr>
</tbody></table>

#### command

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>command</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The exact command line or function call used to run this step, including all arguments. If <code>null</code>, the step is considered <strong>non-reproducible</strong> and all environment fields (<code>docker_image_name</code>, <code>docker_image_url</code>, <code>docker_image_digest</code>, <code>conda_env_url</code>, <code>conda_env_file</code>) and data fields (<code>parameters</code>, <code>inputs</code>, <code>outputs</code>) MAY also be <code>null</code>. If not <code>null</code>, all environment fields SHOULD be provided. Example: <code>"java -jar /opt/asap/Parsing.jar --input /data/raw/ --genome hg38 --threads 8"</code>.</td>
    </tr>
</tbody></table>

#### software_version

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>software_version</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The version of the tool, package, or method used in this step. This refers to the software itself (e.g. Scanpy <code>"1.10.1"</code>), not the container. <code>null</code> if not applicable or unknown. Example: <code>"1.10.1"</code>, <code>"4.3.1"</code>, <code>"v5"</code>, <code>"8.2.0"</code>.</td>
    </tr>
</tbody></table>

#### programming_language

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>programming_language</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The programming language used to implement this step. Example: <code>"Python"</code>, <code>"R"</code>, <code>"Java"</code>, <code>"Bash"</code>, <code>"Julia"</code>.</td>
    </tr>
</tbody></table>

#### programming_language_version

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>programming_language_version</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The version of the programming language runtime used. Example: <code>"3.11.5"</code> (Python), <code>"4.3.2"</code> (R), <code>"11"</code> (Java).</td>
    </tr>
</tbody></table>

### Step environment

The environment section describes the computational environment in which the step was executed. A Docker container image or a conda environment SHOULD be provided for any reproducible step (i.e. when `command` is not `null`). Both MAY be provided simultaneously.

#### docker_repo

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>docker_repo</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if <code>command</code> is not <code>null</code> and a Docker image was used</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The Docker registry hosting the image used for this step. MUST be one of: <code>"dockerhub"</code> (<a href="https://hub.docker.com/">Docker Hub</a>), <code>"ghcr"</code> (<a href="https://ghcr.io/">GitHub Container Registry</a>), <code>"quay"</code> (<a href="https://quay.io/">Quay.io</a>), <code>"custom"</code> (any other registry). <code>null</code> if Docker was not used.</td>
    </tr>
</tbody></table>

#### docker_image_url

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>docker_image_url</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if <code>docker_repo</code> is not <code>null</code></td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A URL pointing to the specific image (or image layer page) on the Docker registry. MUST be a valid URL if provided. Example: <code>"https://hub.docker.com/layers/fabdavid/asap_run/v5"</code>.</td>
    </tr>
</tbody></table>

#### docker_image_name

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>docker_image_name</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if <code>docker_repo</code> is not <code>null</code></td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The full Docker image name with tag, following the format <code>"<repository>/<image>:<tag>"</code>. Example: <code>"fabdavid/asap_run:v5"</code>, <code>"bioconductor/bioconductor_docker:RELEASE_3_18"</code>.</td>
    </tr>
</tbody></table>

#### docker_image_digest

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>docker_image_digest</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if <code>docker_image_name</code> is not <code>null</code></td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The content-addressable SHA-256 digest of the Docker image, in the format <code>"sha256:<hex>"</code>. This provides stronger reproducibility guarantees than a mutable tag, since a tag can be reassigned to a different image. Example: <code>"sha256:a7c4d1f2e3b5c6d7e8f9..."</code>.</td>
    </tr>
</tbody></table>

#### conda_env_url

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>conda_env_url</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A URL pointing to a conda environment specification file (e.g. <code>environment.yml</code>) used for this step. MUST be a valid URL if provided. Either <code>docker_image_name</code> or <code>conda_env_url</code> SHOULD be provided for reproducible steps; both MAY be provided simultaneously. Example: <code>"https://github.com/my-lab/pipeline/blob/v1.0/env/environment.yml"</code>.</td>
    </tr>
</tbody></table>

#### conda_env_file

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>conda_env_file</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The full verbatim content of the conda <code>environment.yml</code> file, embedded as a string. Provides the strongest offline reproducibility guarantees when no stable public URL exists. Either <code>conda_env_url</code> or <code>conda_env_file</code> SHOULD be provided (not necessarily both).</td>
    </tr>
</tbody></table>

### Step parameters

#### parameters

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>parameters</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if <code>command</code> is not <code>null</code></td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>array</code> of <code>object</code>, or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>An ordered array of parameter objects, each describing one named argument or setting passed to the method in this run. Parameters SHOULD be listed in the same order as they appear in the <code>command</code>. <code>null</code> if not applicable or if the step is non-reproducible.</td>
    </tr>
</tbody></table>

##### parameters[*].name

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>parameters[*].name</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED within each parameter object</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The name of the parameter as used by the tool or method. Example: <code>"min_genes"</code>, <code>"n_neighbors"</code>, <code>"resolution"</code>, <code>"--threads"</code>.</td>
    </tr>
</tbody></table>

##### parameters[*].value

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>parameters[*].value</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED within each parameter object</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code>, <code>number</code>, <code>boolean</code>, <code>array</code>, or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The value of the parameter as used in this run. <code>null</code> means the tool default was used without explicit specification. Example: <code>200</code>, <code>0.5</code>, <code>true</code>, <code>"leiden"</code>, <code>["PC_1", "PC_2"]</code>.</td>
    </tr>
</tbody></table>

##### parameters[*].type

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>parameters[*].type</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The expected data type of the parameter value. SHOULD be one of: <code>"string"</code>, <code>"integer"</code>, <code>"float"</code>, <code>"boolean"</code>, <code>"array"</code>. Example: <code>"integer"</code>.</td>
    </tr>
</tbody></table>

##### parameters[*].description

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>parameters[*].description</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A free-text description of what this parameter controls. Example: <code>"Minimum number of genes a cell must express to be retained after filtering."</code>.</td>
    </tr>
</tbody></table>

### Step inputs

#### inputs

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>inputs</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if <code>command</code> is not <code>null</code></td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>array</code> of <code>object</code>, or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>An array of input objects, each describing a file or h5ad metadata slot consumed by this step. <code>null</code> if no inputs are documented or if the step is non-reproducible. Each input object contains the sub-fields described below.</td>
    </tr>
</tbody></table>

##### inputs[*].label

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>inputs[*].label</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED within each input object</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A short, human-readable label for this input. Example: <code>"Raw count matrix"</code>, <code>"Cell barcodes"</code>, <code>"Genome reference FASTA"</code>.</td>
    </tr>
</tbody></table>

##### inputs[*].type

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>inputs[*].type</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED within each input object</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The type of the input. MUST be one of: <code>"file"</code> (a file on disk or remote storage), <code>"metadata_key"</code> (a slot or key in the h5ad object, e.g. <code>obs</code>, <code>var</code>, <code>obsm</code>, <code>obsp</code>, <code>varm</code>, <code>varp</code>, <code>uns</code>), <code>"url"</code> (a remote resource referenced by URL but not downloaded as a file).</td>
    </tr>
</tbody></table>

##### inputs[*].format

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>inputs[*].format</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if <code>inputs[*].type</code> is <code>"file"</code></td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The file format of the input. SHOULD use <a href="http://edamontology.org/">EDAM ontology</a> format terms (e.g. <code>"format_3590"</code> for HDF5) when possible. Free-text values are also accepted. Example: <code>"h5ad"</code>, <code>"fastq.gz"</code>, <code>"mtx"</code>, <code>"tsv"</code>, <code>"csv"</code>, <code>"bam"</code>.</td>
    </tr>
</tbody></table>

##### inputs[*].location

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>inputs[*].location</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The file path, URL, or h5ad slot key of the input. MAY be a relative path, absolute path, or URL. For <code>"metadata_key"</code> type inputs, this SHOULD be the Python-style accessor string. Example: <code>"data/raw/matrix.mtx.gz"</code>, <code>"https://ftp.ncbi.nlm.nih.gov/..."</code>, <code>"obsm['X_pca']"</code>, <code>"uns['neighbors']"</code>.</td>
    </tr>
</tbody></table>

##### inputs[*].description

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>inputs[*].description</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A free-text description of this input and its role in the step. Example: <code>"10x Genomics CellRanger output directory containing barcodes.tsv.gz, features.tsv.gz, and matrix.mtx.gz."</code>.</td>
    </tr>
</tbody></table>

##### inputs[*].checksum

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>inputs[*].checksum</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>A checksum of the input file for integrity verification, in the format <code>"<algorithm>:<hex>"</code>. SHA-256 is preferred. Example: <code>"sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"</code>.</td>
    </tr>
</tbody></table>

### Step outputs

#### outputs

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>outputs</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if <code>command</code> is not <code>null</code></td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>array</code> of <code>object</code>, or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>An array of output objects, each describing a file or h5ad metadata slot produced by this step. <code>null</code> if no outputs are documented or if the step is non-reproducible. Each output object contains the same sub-fields as the input objects.</td>
    </tr>
</tbody></table>
<br/>
The fields `outputs[*].label`, `outputs[*].type`, `outputs[*].format`, `outputs[*].location`, `outputs[*].description`, and `outputs[*].checksum` follow exactly the same specification as the corresponding [`inputs[*].*`](#step-inputs) fields described above.


### Step resources

#### resources

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>resources</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>object</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>An object describing the computational resources used for this step. <code>null</code> if not documented. Contains the sub-fields described below.</td>
    </tr>
</tbody></table>

##### resources.cpu

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>resources.cpu</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>integer</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The number of CPU cores used or requested. Example: <code>8</code>.</td>
    </tr>
</tbody></table>

##### resources.memory_gb

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>resources.memory_gb</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>number</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The amount of RAM used or requested, in gigabytes. Example: <code>64.0</code>.</td>
    </tr>
</tbody></table>

##### resources.gpu

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>resources.gpu</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>integer</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The number of GPU devices used. <code>0</code> if the step ran on CPU only. <code>null</code> if not documented. Example: <code>1</code>.</td>
    </tr>
</tbody></table>

##### resources.gpu_model

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>resources.gpu_model</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The model name of the GPU device(s) used. Example: <code>"NVIDIA A100 80GB"</code>, <code>"NVIDIA V100"</code>.</td>
    </tr>
</tbody></table>

### Execution metadata

#### execution_timestamp

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>execution_timestamp</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>string</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The date and time at which this step started executing, in ISO 8601 format. Example: <code>"2024-01-15T10:35:00Z"</code>.</td>
    </tr>
</tbody></table>

#### execution_duration_seconds

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>execution_duration_seconds</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>number</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The wall-clock execution time of this step, in seconds. Example: <code>120.5</code>.</td>
    </tr>
</tbody></table>

#### random_seed

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>random_seed</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>RECOMMENDED if the method involves any stochastic component</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>integer</code> or <code>null</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>The integer random seed used in this step, if the method involves any stochastic element (e.g. UMAP, t-SNE, Leiden/Louvain clustering, doublet detection, simulation). Setting and recording a fixed seed is critical for exact reproducibility. <code>null</code> if the step is fully deterministic or if the seed was not recorded. Example: <code>42</code>.</td>
    </tr>
</tbody></table>

## Appendix A: Full example JSON

The following is a complete, valid `analysis_json` document illustrating all field types, including reproducible steps, a non-reproducible step, and steps that read from or write to h5ad slots.

```json
{
  "schema_version": "7.1.0+scfair1.0",
  "pipeline_name": "ASAP Analysis Pipeline",
  "pipeline_version": "v5",
  "pipeline_description": "Standard scRNA-seq preprocessing and clustering pipeline for 10x Chromium data, including parsing, QC, normalization, PCA, UMAP, and Leiden clustering.",
  "pipeline_url": "https://github.com/my-lab/asap-pipeline",
  "creation_date": "2024-01-15T10:30:00Z",
  "steps": [
    {
      "step_label": "Parsing",
      "step_description": "Parses 10x CellRanger output into an AnnData object with raw counts.",
      "step_category": "Parsing",
      "method": "Parsing.java",
      "command": "java -jar /opt/asap/Parsing.jar --input /data/raw/ --output /data/parsed.h5ad --genome hg38 --threads 4",
      "software_version": "v5",
      "programming_language": "Java",
      "programming_language_version": "11",
      "docker_repo": "dockerhub",
      "docker_image_url": "https://hub.docker.com/layers/fabdavid/asap_run/v5",
      "docker_image_name": "fabdavid/asap_run:v5",
      "docker_image_digest": "sha256:a7c4d1f2e3b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1",
      "conda_env_url": null,
      "conda_env_file": null,
      "parameters": [
        {
          "name": "--genome",
          "value": "hg38",
          "type": "string",
          "description": "Reference genome assembly used for alignment."
        },
        {
          "name": "--threads",
          "value": 4,
          "type": "integer",
          "description": "Number of CPU threads for parallel processing."
        }
      ],
      "inputs": [
        {
          "label": "CellRanger output directory",
          "type": "file",
          "format": "mtx",
          "location": "/data/raw/",
          "description": "10x Genomics CellRanger output directory containing barcodes.tsv.gz, features.tsv.gz, and matrix.mtx.gz.",
          "checksum": null
        }
      ],
      "outputs": [
        {
          "label": "Parsed AnnData object",
          "type": "file",
          "format": "h5ad",
          "location": "/data/parsed.h5ad",
          "description": "AnnData object with raw counts in X and cell barcodes/gene names as obs/var index.",
          "checksum": null
        }
      ],
      "resources": {
        "cpu": 4,
        "memory_gb": 16.0,
        "gpu": 0,
        "gpu_model": null
      },
      "execution_timestamp": "2024-01-15T10:35:00Z",
      "execution_duration_seconds": 45.2,
      "random_seed": null
    },
    {
      "step_label": "Quality Control",
      "step_description": "Computes QC metrics and filters low-quality cells (too few genes, too many mitochondrial reads) and lowly detected genes.",
      "step_category": "Quality Control",
      "method": "scanpy.pp.filter_cells / scanpy.pp.filter_genes",
      "command": "python qc.py --input /data/parsed.h5ad --output /data/filtered.h5ad --min-genes 200 --max-pct-mito 20 --min-cells 3",
      "software_version": "1.10.1",
      "programming_language": "Python",
      "programming_language_version": "3.11.5",
      "docker_repo": "dockerhub",
      "docker_image_url": "https://hub.docker.com/layers/fabdavid/asap_run/v5",
      "docker_image_name": "fabdavid/asap_run:v5",
      "docker_image_digest": "sha256:a7c4d1f2e3b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1",
      "conda_env_url": null,
      "conda_env_file": null,
      "parameters": [
        {
          "name": "min_genes",
          "value": 200,
          "type": "integer",
          "description": "Minimum number of genes a cell must express to be retained."
        },
        {
          "name": "max_pct_mito",
          "value": 20.0,
          "type": "float",
          "description": "Maximum percentage of mitochondrial gene expression allowed per cell."
        },
        {
          "name": "min_cells",
          "value": 3,
          "type": "integer",
          "description": "Minimum number of cells a gene must be detected in to be retained."
        }
      ],
      "inputs": [
        {
          "label": "Parsed AnnData object",
          "type": "file",
          "format": "h5ad",
          "location": "/data/parsed.h5ad",
          "description": "Raw AnnData object from the Parsing step.",
          "checksum": null
        }
      ],
      "outputs": [
        {
          "label": "QC-filtered AnnData object",
          "type": "file",
          "format": "h5ad",
          "location": "/data/filtered.h5ad",
          "description": "AnnData object after cell and gene filtering, with QC metrics stored in obs.",
          "checksum": null
        }
      ],
      "resources": {
        "cpu": 2,
        "memory_gb": 8.0,
        "gpu": 0,
        "gpu_model": null
      },
      "execution_timestamp": "2024-01-15T10:36:00Z",
      "execution_duration_seconds": 12.1,
      "random_seed": null
    },
    {
      "step_label": "Normalization",
      "step_description": "Normalizes counts to 10,000 reads per cell (CP10K) and log-transforms.",
      "step_category": "Normalization",
      "method": "scanpy.pp.normalize_total / scanpy.pp.log1p",
      "command": "python normalize.py --input /data/filtered.h5ad --output /data/normalized.h5ad --target-sum 10000",
      "software_version": "1.10.1",
      "programming_language": "Python",
      "programming_language_version": "3.11.5",
      "docker_repo": "dockerhub",
      "docker_image_url": "https://hub.docker.com/layers/fabdavid/asap_run/v5",
      "docker_image_name": "fabdavid/asap_run:v5",
      "docker_image_digest": "sha256:a7c4d1f2e3b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1",
      "conda_env_url": null,
      "conda_env_file": null,
      "parameters": [
        {
          "name": "target_sum",
          "value": 10000,
          "type": "integer",
          "description": "Total count to normalize each cell to before log-transformation."
        }
      ],
      "inputs": [
        {
          "label": "Filtered AnnData object",
          "type": "file",
          "format": "h5ad",
          "location": "/data/filtered.h5ad",
          "description": "QC-filtered AnnData from the Quality Control step.",
          "checksum": null
        }
      ],
      "outputs": [
        {
          "label": "Normalized AnnData object",
          "type": "file",
          "format": "h5ad",
          "location": "/data/normalized.h5ad",
          "description": "AnnData with log1p-normalized counts in X; raw counts preserved in raw.X.",
          "checksum": null
        }
      ],
      "resources": {
        "cpu": 2,
        "memory_gb": 8.0,
        "gpu": 0,
        "gpu_model": null
      },
      "execution_timestamp": "2024-01-15T10:36:30Z",
      "execution_duration_seconds": 8.4,
      "random_seed": null
    },
    {
      "step_label": "UMAP",
      "step_description": "Computes UMAP embedding for 2D visualization using the precomputed neighborhood graph.",
      "step_category": "Embedding",
      "method": "scanpy.tl.umap",
      "command": "python embedding.py --input /data/normalized.h5ad --output /data/embedded.h5ad --n-neighbors 15 --min-dist 0.5 --random-state 42",
      "software_version": "1.10.1",
      "programming_language": "Python",
      "programming_language_version": "3.11.5",
      "docker_repo": "dockerhub",
      "docker_image_url": "https://hub.docker.com/layers/fabdavid/asap_run/v5",
      "docker_image_name": "fabdavid/asap_run:v5",
      "docker_image_digest": "sha256:a7c4d1f2e3b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b3c4d5e6f7a8b9c0d1",
      "conda_env_url": null,
      "conda_env_file": null,
      "parameters": [
        {
          "name": "n_neighbors",
          "value": 15,
          "type": "integer",
          "description": "Number of nearest neighbors used to build the neighborhood graph."
        },
        {
          "name": "min_dist",
          "value": 0.5,
          "type": "float",
          "description": "Minimum distance between points in the low-dimensional embedding."
        },
        {
          "name": "random_state",
          "value": 42,
          "type": "integer",
          "description": "Random seed for reproducibility of the stochastic UMAP algorithm."
        }
      ],
      "inputs": [
        {
          "label": "Neighborhood graph connectivities",
          "type": "metadata_key",
          "format": null,
          "location": "obsp['connectivities']",
          "description": "Precomputed k-nearest neighbor graph stored in the AnnData object.",
          "checksum": null
        }
      ],
      "outputs": [
        {
          "label": "UMAP 2D coordinates",
          "type": "metadata_key",
          "format": null,
          "location": "obsm['X_umap']",
          "description": "2D UMAP embedding coordinates (n_cells × 2) stored in the AnnData object.",
          "checksum": null
        }
      ],
      "resources": {
        "cpu": 4,
        "memory_gb": 32.0,
        "gpu": 0,
        "gpu_model": null
      },
      "execution_timestamp": "2024-01-15T11:05:00Z",
      "execution_duration_seconds": 180.4,
      "random_seed": 42
    },
    {
      "step_label": "Manual Cell Type Annotation",
      "step_description": "Manual cell type annotation performed by a domain expert based on canonical marker genes. Not computationally reproducible.",
      "step_category": "Cell Type Annotation",
      "method": "Manual curation",
      "command": null,
      "software_version": null,
      "programming_language": null,
      "programming_language_version": null,
      "docker_repo": null,
      "docker_image_url": null,
      "docker_image_name": null,
      "docker_image_digest": null,
      "conda_env_url": null,
      "conda_env_file": null,
      "parameters": null,
      "inputs": null,
      "outputs": null,
      "resources": null,
      "execution_timestamp": null,
      "execution_duration_seconds": null,
      "random_seed": null
    }
  ]
}
```


## Appendix B: Changelog

### scfair 1.0

- Initial release of the `analysis_json` schema.
- Defined top-level pipeline fields: `schema_version`, `pipeline_name`, `pipeline_version`, `pipeline_description`, `pipeline_url`, `creation_date`, `steps`.
- Defined per-step fields: `step_label`, `step_description`, `step_category`, `method`, `command`, `software_version`, `programming_language`, `programming_language_version`, `docker_repo`, `docker_image_url`, `docker_image_name`, `docker_image_digest`, `conda_env_url`, `conda_env_file`, `parameters`, `inputs`, `outputs`, `resources`, `execution_timestamp`, `execution_duration_seconds`, `random_seed`.
