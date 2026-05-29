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

| Key         | `schema_version`                                                                                                                                                                       |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED                                                                                                                                                                               |
| Type        | `string`                                                                                                                                                                               |
| Value       | The version of the scFAIR `analysis_json` schema used to generate this document. MUST match the version string of the core schema. Example: `"7.1.0+scfair1.0"`. |

### pipeline_name

| Key         | `pipeline_name`                                                                                                |
| ----------- | -------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED                                                                                                       |
| Type        | `string`                                                                                                       |
| Value       | A short, human-readable name for the pipeline. Example: `"ASAP Analysis Pipeline"`, `"Seurat Standard Workflow"`. |

### pipeline_version

| Key         | `pipeline_version`                                                                                                                                                      |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED                                                                                                                                                             |
| Type        | `string` or `null`                                                                                                                                                      |
| Value       | The version of the pipeline that was run. SHOULD follow [Semantic Versioning](https://semver.org/). `null` if not versioned. Example: `"v5"`, `"1.3.2"`. |

### pipeline_description

| Key         | `pipeline_description`                                                                                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Requirement | RECOMMENDED                                                                                                                                                                    |
| Type        | `string` or `null`                                                                                                                                                             |
| Value       | A free-text description of the overall pipeline and its purpose. Example: `"Standard preprocessing and clustering pipeline for 10x Chromium scRNA-seq data."`. |

### pipeline_url

| Key         | `pipeline_url`                                                                                                                                                         |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED                                                                                                                                                            |
| Type        | `string` or `null`                                                                                                                                                     |
| Value       | A URL pointing to the source code, documentation, or publication describing the pipeline. MUST be a valid URL if provided. Example: `"https://github.com/my-lab/asap-pipeline"`. |

### creation_date

| Key         | `creation_date`                                                                                                                                |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED                                                                                                                                    |
| Type        | `string` or `null`                                                                                                                             |
| Value       | The date and time at which this JSON document was generated, in ISO 8601 format. Example: `"2024-01-15T10:30:00Z"`. |

### steps

| Key         | `steps`                                                                                                                                                                                                                        |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Requirement | REQUIRED                                                                                                                                                                                                                       |
| Type        | `array` of `object`                                                                                                                                                                                                            |
| Value       | An ordered array of step objects describing each analysis step. Steps MUST appear in chronological execution order. Each element is a step object as described in the [`steps` (Pipeline steps)](#steps-pipeline-steps) section. The array MAY be empty (`[]`) if no steps are documented. |


## `steps` (Pipeline steps)

`steps` is a JSON Array of step objects. Each step object describes one analysis run or computational step in the pipeline. Steps are **ordered**: the position of a step in the array (0-indexed) reflects its execution order.

### Step identification

#### step_label

| Key         | `step_label`                                                                                                                                                                                   |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED                                                                                                                                                                                       |
| Type        | `string`                                                                                                                                                                                       |
| Value       | A short, human-readable label for the step. SHOULD be unique within the `steps` array, as it may be used for cross-referencing. Example: `"Parsing"`, `"Quality Control"`, `"UMAP"`, `"Leiden Clustering"`. |

#### step_description

| Key         | `step_description`                                                                                                                                                                                          |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                                                                                                                                    |
| Type        | `string` or `null`                                                                                                                                                                                          |
| Value       | A free-text description of what this step does. Example: `"Filters cells with fewer than 200 genes expressed and removes cells with more than 20% mitochondrial gene expression."`. |

#### step_category

| Key         | `step_category`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| Type        | `string` or `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Value       | A controlled term describing the broad category of the analysis step. MUST be one of: `"Parsing"`, `"Quality Control"`, `"Doublet Detection"`, `"Normalization"`, `"Scaling"`, `"Feature Selection"`, `"Dimensionality Reduction"`, `"Embedding"`, `"Clustering"`, `"Differential Expression"`, `"Cell Type Annotation"`, `"Trajectory Analysis"`, `"RNA Velocity"`, `"Batch Correction"`, `"Integration"`, `"Cell Communication"`, `"Gene Regulatory Network"`, `"Peak Calling"`, `"Motif Analysis"`, `"Other"`. |


### Step execution

#### method

| Key         | `method`                                                                                                                                                                                           |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED                                                                                                                                                                                           |
| Type        | `string`                                                                                                                                                                                           |
| Value       | The name of the method, script, function, or tool used for this step. Example: `"Parsing.java"`, `"scanpy.pp.normalize_total"`, `"Seurat::FindClusters"`, `"cellranger count"`, `"Manual curation"`. |

#### command

| Key         | `command`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| Type        | `string` or `null`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                         |
| Value       | The exact command line or function call used to run this step, including all arguments. If `null`, the step is considered **non-reproducible** and all environment fields (`docker_image_name`, `docker_image_url`, `docker_image_digest`, `conda_env_url`, `conda_env_file`) and data fields (`parameters`, `inputs`, `outputs`) MAY also be `null`. If not `null`, all environment fields SHOULD be provided. Example: `"java -jar /opt/asap/Parsing.jar --input /data/raw/ --genome hg38 --threads 8"`. |

#### software_version

| Key         | `software_version`                                                                                                                                                                                                                                       |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED                                                                                                                                                                                                                                              |
| Type        | `string` or `null`                                                                                                                                                                                                                                       |
| Value       | The version of the tool, package, or method used in this step. This refers to the software itself (e.g. Scanpy `"1.10.1"`), not the container. `null` if not applicable or unknown. Example: `"1.10.1"`, `"4.3.1"`, `"v5"`, `"8.2.0"`. |

#### programming_language

| Key         | `programming_language`                                                                                                                      |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                                                                    |
| Type        | `string` or `null`                                                                                                                          |
| Value       | The programming language used to implement this step. Example: `"Python"`, `"R"`, `"Java"`, `"Bash"`, `"Julia"`. |

#### programming_language_version

| Key         | `programming_language_version`                                                                                         |
| ----------- | ---------------------------------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                                               |
| Type        | `string` or `null`                                                                                                     |
| Value       | The version of the programming language runtime used. Example: `"3.11.5"` (Python), `"4.3.2"` (R), `"11"` (Java). |


### Step environment

The environment section describes the computational environment in which the step was executed. A Docker container image or a conda environment SHOULD be provided for any reproducible step (i.e. when `command` is not `null`). Both MAY be provided simultaneously.

#### docker_repo

| Key         | `docker_repo`                                                                                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED if `command` is not `null` and a Docker image was used                                                                                                                                                             |
| Type        | `string` or `null`                                                                                                                                                                                                              |
| Value       | The Docker registry hosting the image used for this step. MUST be one of: `"dockerhub"` ([Docker Hub](https://hub.docker.com/)), `"ghcr"` ([GitHub Container Registry](https://ghcr.io/)), `"quay"` ([Quay.io](https://quay.io/)), `"custom"` (any other registry). `null` if Docker was not used. |

#### docker_image_url

| Key         | `docker_image_url`                                                                                                                                                                   |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Requirement | RECOMMENDED if `docker_repo` is not `null`                                                                                                                                          |
| Type        | `string` or `null`                                                                                                                                                                   |
| Value       | A URL pointing to the specific image (or image layer page) on the Docker registry. MUST be a valid URL if provided. Example: `"https://hub.docker.com/layers/fabdavid/asap_run/v5"`. |

#### docker_image_name

| Key         | `docker_image_name`                                                                                                                                                                         |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED if `docker_repo` is not `null`                                                                                                                                                 |
| Type        | `string` or `null`                                                                                                                                                                          |
| Value       | The full Docker image name with tag, following the format `"<repository>/<image>:<tag>"`. Example: `"fabdavid/asap_run:v5"`, `"bioconductor/bioconductor_docker:RELEASE_3_18"`. |

#### docker_image_digest

| Key         | `docker_image_digest`                                                                                                                                                                                                                                              |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Requirement | RECOMMENDED if `docker_image_name` is not `null`                                                                                                                                                                                                                   |
| Type        | `string` or `null`                                                                                                                                                                                                                                                 |
| Value       | The content-addressable SHA-256 digest of the Docker image, in the format `"sha256:<hex>"`. This provides stronger reproducibility guarantees than a mutable tag, since a tag can be reassigned to a different image. Example: `"sha256:a7c4d1f2e3b5c6d7e8f9..."`. |

#### conda_env_url

| Key         | `conda_env_url`                                                                                                                                                                                                                                                        |
| ----------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                                                                                                                                                                                               |
| Type        | `string` or `null`                                                                                                                                                                                                                                                     |
| Value       | A URL pointing to a conda environment specification file (e.g. `environment.yml`) used for this step. MUST be a valid URL if provided. Either `docker_image_name` or `conda_env_url` SHOULD be provided for reproducible steps; both MAY be provided simultaneously. Example: `"https://github.com/my-lab/pipeline/blob/v1.0/env/environment.yml"`. |

#### conda_env_file

| Key         | `conda_env_file`                                                                                                                                                                                                                                    |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                                                                                                                                                                            |
| Type        | `string` or `null`                                                                                                                                                                                                                                  |
| Value       | The full verbatim content of the conda `environment.yml` file, embedded as a string. Provides the strongest offline reproducibility guarantees when no stable public URL exists. Either `conda_env_url` or `conda_env_file` SHOULD be provided (not necessarily both). |


### Step parameters

#### parameters

| Key         | `parameters`                                                                                                                                                                                                                                              |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED if `command` is not `null`                                                                                                                                                                                                                   |
| Type        | `array` of `object`, or `null`                                                                                                                                                                                                                            |
| Value       | An ordered array of parameter objects, each describing one named argument or setting passed to the method in this run. Parameters SHOULD be listed in the same order as they appear in the `command`. `null` if not applicable or if the step is non-reproducible. |

##### parameters[*].name

| Key         | `parameters[*].name`                                                                                              |
| ----------- | ----------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED within each parameter object                                                                             |
| Type        | `string`                                                                                                          |
| Value       | The name of the parameter as used by the tool or method. Example: `"min_genes"`, `"n_neighbors"`, `"resolution"`, `"--threads"`. |

##### parameters[*].value

| Key         | `parameters[*].value`                                                                                                                                    |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED within each parameter object                                                                                                                    |
| Type        | `string`, `number`, `boolean`, `array`, or `null`                                                                                                        |
| Value       | The value of the parameter as used in this run. `null` means the tool default was used without explicit specification. Example: `200`, `0.5`, `true`, `"leiden"`, `["PC_1", "PC_2"]`. |

##### parameters[*].type

| Key         | `parameters[*].type`                                                                                                                         |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED                                                                                                                                  |
| Type        | `string` or `null`                                                                                                                           |
| Value       | The expected data type of the parameter value. SHOULD be one of: `"string"`, `"integer"`, `"float"`, `"boolean"`, `"array"`. Example: `"integer"`. |

##### parameters[*].description

| Key         | `parameters[*].description`                                                                                                            |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                                                               |
| Type        | `string` or `null`                                                                                                                     |
| Value       | A free-text description of what this parameter controls. Example: `"Minimum number of genes a cell must express to be retained after filtering."`. |


### Step inputs

#### inputs

| Key         | `inputs`                                                                                                                                                                                                                                        |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED if `command` is not `null`                                                                                                                                                                                                         |
| Type        | `array` of `object`, or `null`                                                                                                                                                                                                                  |
| Value       | An array of input objects, each describing a file or h5ad metadata slot consumed by this step. `null` if no inputs are documented or if the step is non-reproducible. Each input object contains the sub-fields described below. |

##### inputs[*].label

| Key         | `inputs[*].label`                                                                                                           |
| ----------- | --------------------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED within each input object                                                                                           |
| Type        | `string`                                                                                                                    |
| Value       | A short, human-readable label for this input. Example: `"Raw count matrix"`, `"Cell barcodes"`, `"Genome reference FASTA"`. |

##### inputs[*].type

| Key         | `inputs[*].type`                                                                                                                                                                                                                                                      |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | REQUIRED within each input object                                                                                                                                                                                                                                     |
| Type        | `string`                                                                                                                                                                                                                                                              |
| Value       | The type of the input. MUST be one of: `"file"` (a file on disk or remote storage), `"metadata_key"` (a slot or key in the h5ad object, e.g. `obs`, `var`, `obsm`, `obsp`, `varm`, `varp`, `uns`), `"url"` (a remote resource referenced by URL but not downloaded as a file). |

##### inputs[*].format

| Key         | `inputs[*].format`                                                                                                                                                                                                      |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED if `inputs[*].type` is `"file"`                                                                                                                                                                            |
| Type        | `string` or `null`                                                                                                                                                                                                      |
| Value       | The file format of the input. SHOULD use [EDAM ontology](http://edamontology.org/) format terms (e.g. `"format_3590"` for HDF5) when possible. Free-text values are also accepted. Example: `"h5ad"`, `"fastq.gz"`, `"mtx"`, `"tsv"`, `"csv"`, `"bam"`. |

##### inputs[*].location

| Key         | `inputs[*].location`                                                                                                                                                                                                                       |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Requirement | RECOMMENDED                                                                                                                                                                                                                                |
| Type        | `string` or `null`                                                                                                                                                                                                                         |
| Value       | The file path, URL, or h5ad slot key of the input. MAY be a relative path, absolute path, or URL. For `"metadata_key"` type inputs, this SHOULD be the Python-style accessor string. Example: `"data/raw/matrix.mtx.gz"`, `"https://ftp.ncbi.nlm.nih.gov/..."`, `"obsm['X_pca']"`, `"uns['neighbors']"`. |

##### inputs[*].description

| Key         | `inputs[*].description`                                                                                                                                                                    |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Requirement | OPTIONAL                                                                                                                                                                                   |
| Type        | `string` or `null`                                                                                                                                                                         |
| Value       | A free-text description of this input and its role in the step. Example: `"10x Genomics CellRanger output directory containing barcodes.tsv.gz, features.tsv.gz, and matrix.mtx.gz."`. |

##### inputs[*].checksum

| Key         | `inputs[*].checksum`                                                                                                                                                        |
| ----------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                                                                                                    |
| Type        | `string` or `null`                                                                                                                                                          |
| Value       | A checksum of the input file for integrity verification, in the format `"<algorithm>:<hex>"`. SHA-256 is preferred. Example: `"sha256:e3b0c44298fc1c149afbf4c8996fb92427ae41e4649b934ca495991b7852b855"`. |


### Step outputs

#### outputs

| Key         | `outputs`                                                                                                                                                                                                                                       |
| ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED if `command` is not `null`                                                                                                                                                                                                         |
| Type        | `array` of `object`, or `null`                                                                                                                                                                                                                  |
| Value       | An array of output objects, each describing a file or h5ad metadata slot produced by this step. `null` if no outputs are documented or if the step is non-reproducible. Each output object contains the same sub-fields as the input objects. |

The fields `outputs[*].label`, `outputs[*].type`, `outputs[*].format`, `outputs[*].location`, `outputs[*].description`, and `outputs[*].checksum` follow exactly the same specification as the corresponding [`inputs[*].*`](#step-inputs) fields described above.


### Step resources

#### resources

| Key         | `resources`                                                                                                                          |
| ----------- | ------------------------------------------------------------------------------------------------------------------------------------ |
| Requirement | OPTIONAL                                                                                                                             |
| Type        | `object` or `null`                                                                                                                   |
| Value       | An object describing the computational resources used for this step. `null` if not documented. Contains the sub-fields described below. |

##### resources.cpu

| Key         | `resources.cpu`                                                    |
| ----------- | ------------------------------------------------------------------ |
| Requirement | OPTIONAL                                                           |
| Type        | `integer` or `null`                                                |
| Value       | The number of CPU cores used or requested. Example: `8`. |

##### resources.memory_gb

| Key         | `resources.memory_gb`                                                           |
| ----------- | ------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                        |
| Type        | `number` or `null`                                                              |
| Value       | The amount of RAM used or requested, in gigabytes. Example: `64.0`. |

##### resources.gpu

| Key         | `resources.gpu`                                                                                        |
| ----------- | ------------------------------------------------------------------------------------------------------ |
| Requirement | OPTIONAL                                                                                               |
| Type        | `integer` or `null`                                                                                    |
| Value       | The number of GPU devices used. `0` if the step ran on CPU only. `null` if not documented. Example: `1`. |

##### resources.gpu_model

| Key         | `resources.gpu_model`                                                                         |
| ----------- | --------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                      |
| Type        | `string` or `null`                                                                            |
| Value       | The model name of the GPU device(s) used. Example: `"NVIDIA A100 80GB"`, `"NVIDIA V100"`. |


### Execution metadata

#### execution_timestamp

| Key         | `execution_timestamp`                                                                                                    |
| ----------- | ------------------------------------------------------------------------------------------------------------------------ |
| Requirement | OPTIONAL                                                                                                                 |
| Type        | `string` or `null`                                                                                                       |
| Value       | The date and time at which this step started executing, in ISO 8601 format. Example: `"2024-01-15T10:35:00Z"`. |

#### execution_duration_seconds

| Key         | `execution_duration_seconds`                                                                              |
| ----------- | --------------------------------------------------------------------------------------------------------- |
| Requirement | OPTIONAL                                                                                                  |
| Type        | `number` or `null`                                                                                        |
| Value       | The wall-clock execution time of this step, in seconds. Example: `120.5`. |

#### random_seed

| Key         | `random_seed`                                                                                                                                                                                                                                                                          |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Requirement | RECOMMENDED if the method involves any stochastic component                                                                                                                                                                                                                           |
| Type        | `integer` or `null`                                                                                                                                                                                                                                                                    |
| Value       | The integer random seed used in this step, if the method involves any stochastic element (e.g. UMAP, t-SNE, Leiden/Louvain clustering, doublet detection, simulation). Setting and recording a fixed seed is critical for exact reproducibility. `null` if the step is fully deterministic or if the seed was not recorded. Example: `42`. |


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
