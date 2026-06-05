# Forked from

- [schema.md](https://github.com/chanzuckerberg/single-cell-curation/blob/main/schema/7.1.0/schema.md)
- Last updated on Jan, 14 2026
- [Latest commit](https://github.com/chanzuckerberg/single-cell-curation/commit/65b419f55c403ab6809874a56e48fc1406c14148)

# Summary

## Core schema (scRNA-seq + shared across all modalities)

**Note:** Terms in italic are auto-filled by the CZI CELLxGENE submission pipeline. scFAIR schema still consider them as required. They should match their ontology id paired field entries and/or requirements.

**AnnData.h5ad**
* [`AnnData.raw.X`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#x-matrix-layers) [`scipy.sparse.csr_matrix`] - Main dataset. Raw counts (not normalized). Fix the dimension for all other metadata. Contains all genes and filtered cells.
* [`AnnData.X`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#x-matrix-layers) - Main dataset. Normalized counts. Should be the same dimension than `AnnData.raw.X`. **Exception** (not recommended):  if `AnnData.raw.X` is NOT provided, it can be the raw count matrix.
* [`AnnData.obs`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obs-cell-metadata) - Cell metadata. Describe each cell in the dataset.
  * [`index`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#index-of-pandasdataframe) [`str`] - The index of the pandas.DataFrame MUST contain unique identifiers for observations. It's usually set as the cell barcodes.
  * [`assay_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#assay_ontology_term_id) [`str`] - EFO term describing the assay. e.g. `"EFO:0022605"` for *10x 5' v3*.
  * *[`assay`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#assay)* [`str`] (Paired with `assay_ontology_term_id`) - Human-readable name assigned to the value of `assay_ontology_term_id`.
  * [`tissue_type`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#tissue_type) [`str`] - One of `"tissue"`, `"organoid"`, `"cell line"`, or `"primary cell culture"`.
  * [`tissue_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#tissue_ontology_term_id) [`str`] - UBERON term if `tissue_type` is `"tissue"` or `"organoid"`. Cellosaurus term if `tissue_type` is `"cell line"`, or CL term if `tissue_type` is `"primary cell culture"`.
  * *[`tissue`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#tissue)* [`str`] (Paired with `tissue_ontology_term_id`) - Human-readable name(s) assigned to the value(s) of `tissue_ontology_term_id`.
  * [`cell_type_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#cell_type_ontology_term_id) [`str`] - CL term (or `"unknown"`) describing the curated cell-type. e.g. `"CL:0000115"` for *endothelial cell*. Only present if `uns['is_pre_analysis'] = False`. Can be a cross-referenced species-specific ontology term such as `"FBbt:00046052"` for *fat cell* in flies.
  * *[`cell_type`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#cell_type)* [`str`] (Paired with `cell_type_ontology_term_id`) - Human-readable name assigned to the value of `cell_type_ontology_term_id`.
  * [`development_stage_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#development_stage_ontology_term_id) [`str`] - UBERON term (or `"unknown"`, or `"na"` if `tissue_type` is `"cell line"`) describing the curated developmental stage. e.g. `"UBERON:0000068"` for *embryo stage*. Can be a UBERON cross-referenced species-specific ontology term such as `"FBdv:00007077"` for *day 2 of adulthood* in flies.
  * *[`development_stage`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#development_stage)* [`str`] (Paired with `development_stage_ontology_term_id`) - Human-readable name assigned to the value of `development_stage_ontology_term_id`.
  * [`sex_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#sex_ontology_term_id) [`str`] - One of `"PATO:0000383"` for female, `"PATO:0000384"` for male, or `"PATO:0001340"` for hermaphrodite (or "unknown", or "na" if `tissue_type` is `"cell line"`.
  * *[`sex`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#sex)* [`str`] (Paired with `sex_ontology_term_id`) - Human-readable name(s) assigned to the value(s) of `sex_ontology_term_id`.
  * [`self_reported_ethnicity_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#self_reported_ethnicity_ontology_term_id) [`str`] - HANCESTRO term (or "unknown") only if organism is human ("na" for other organisms or if `tissue_type` is `"cell line"`).
  * *[`self_reported_ethnicity`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#self_reported_ethnicity)* [`str`] (Paired with `self_reported_ethnicity_ontology_term_id`) - Human-readable name(s) assigned to the value(s) of `self_reported_ethnicity_ontology_term_id`.
  * *&lt;Optional&gt;*[`strain_or_genetic_background_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#strain_or_genetic_background_term_id) [`str`] - Controlled value (e.g. EFO) describing the experimentally relevant genetic background of a non human organism (strain, stock, line, cultivar, genotype, ecotype, variety, serovar, or breeding and crossing design).
  * *&lt;Optional&gt;*[`strain_or_genetic_background`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#strain_or_genetic_background) [`str`] - Human-readable name(s) assigned to the value(s) of `strain_or_genetic_background_term_id` id set, or free text describing the experimentally relevant genetic background of a non human organism (strain, stock, line, cultivar, genotype, ecotype, variety, serovar, or breeding and crossing design).
  * [`disease_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#disease_ontology_term_id) [`str`] - `"PATO:0000461"` for *normal* or *healthy*, or MONDO term. Can be multiple terms in ascending lexical order separated by the delimiter `" || "` with no duplication of identifiers e.g. `"MONDO:0004604 || MONDO:0043004"`.
  * *[`disease`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#disease)* [`str`] (Paired with `disease_ontology_term_id`) - Human-readable name(s) assigned to the value(s) of `disease_ontology_term_id`.
  * [`experimental_condition_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#experimental_condition_ontology_term_id) [`str`] - Terms(s) from CHEBI, EFO, uniprot, or anti-uniprot (or `"na"`) to describe experimental conditions. Can be multiple terms in ascending lexical order separated by the delimiter `" || "` with no duplication of identifiers.
  * *[`experimental_condition`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#experimental_condition)* [`str`] (Paired with `experimental_condition_ontology_term_id`) - Human-readable name(s) assigned to the value(s) of `experimental_condition_ontology_term_id`.
  * *[`perturbation_types`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#perturbation_types)* [`str`] - One of `"no perturbations"`, `"chemical"` (if `experimental_condition_ontology_term_id` contains a `CHEBI:` term), "diet" (if `experimental_condition_ontology_term_id` contains the `"EFO:0002755"` term), "genetic" (if `genetic_perturbation_term_id` is not `"na"`, see [schema_perturb.md](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbation_term_id)), "protein" (if `experimental_condition_ontology_term_id` contains a `uniprot:` term), "temperature" (if `experimental_condition_ontology_term_id` contains the `"EFO:0001702"` term)
  * [`donor_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#donor_id) [`str`] - Identifies unique individuals (or `"pooled"`, or `"unknown"`, "or `"na"` if `tissue_type` is `"cell line"`)
  * [`is_primary_data`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#is_primary_data) [`bool`] - True or False depending on origin. For meta-analyses or integration study it should be False.
  * [`suspension_type`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#suspension_type) [`str`] - One of `"cell"`, `"nucleus"`, or `"na"`. Should match with selected `assay`.
* [`AnnData.obsm`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obsm-embeddings) - Embeddings. Describe each embedding in the dataset
  * [`X_{suffix}`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#x_suffix) [`numpy.ndarray`] - One or multiple embeddings e.g. X_tSNE, X_PCA, X_UMAP, ...
* [`Anndata.obsp`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obsp) - Describe pairwise annotation of observations. Nothing is mandatory here.
* [`Anndata.var` and `Anndata.raw.var`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#var-and-rawvar-gene-metadata) - Gene metadata. Describe each gene in the dataset.
  * [`feature_is_filtered`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#feature_is_filtered) - Describe which genes in the normalized matrix are filtered as compared to the raw matrix.
  * *[`feature_biotype`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#feature_biotype)* [`str`] - One of `"gene"` or `"spike-in"`.
  * *[`feature_length`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#feature_length)* [`uint`] - Number of base-pairs (bps). The value is the median of the lengths of isoforms, reusing the median calculation from [GTFtools](https://academic.oup.com/bioinformatics/article/38/20/4806/6674500). This is used for RPKM-like normalization.
  * *[`feature_name`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#feature_name)* [`str`] - Human-readable name for the spike-in or the gene name (not the Ensembl Id)
  * *[`feature_reference`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#feature_reference)* [`str`] - NCBITaxon organism id the feature comes from, e.g. `"NCBITaxon:10090"` for *mus musculus* or `"NCBITaxon:32630"` for ERCC Spike-Ins.
  * *[`feature_type`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#feature_type)* [`str`] - Biotype of the corresponding genes, e.g. `"protein coding"` or `"ncRNA"`. If `feature_biotype` is `"spike-in"` then this MUST be `"synthetic"`.
  * *[`feature_chromosome`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#feature_chromosome)* [`str`] - Chromosome location of the corresponding genes, e.g. `"MT"` or `"1"`.
* [`Anndata.varm`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#varm) - Describe multi-dimensional annotation of variables/features. Nothing is mandatory here.
* [`Anndata.varp`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#varp) - Describe pairwise annotation of variables/features. Nothing is mandatory here.
* [`Anndata.uns`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#uns-dataset-metadata) - Dataset metadata. Describe the dataset as a whole.
  * [`ensembl_release`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#ensembl_release) [`int`] - <b>Ensembl</b> release number of the assembly used for gene annotation, e.g. <code>115</code> for <a href="https://ftp.ensembl.org/pub/release-115/">Ensembl r.115</a>.
  * [`ensembl_database`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#ensembl_database)  [`str`] - <b>Ensembl</b> database name of the assembly used for gene annotation. One of <code>"Ensembl"</code>, <code>"EnsemblBacteria"</code>, <code>"EnsemblFungi"</code>, <code>"EnsemblPlants"</code>, <code>"EnsemblProtists"</code>, <code>"EnsemblMetazoa"</code>, or <code>"EnsemblCOVID-19"</code>.
  * [`ensembl_assembly`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#ensembl_assembly)  [`str`] - <b>Ensembl</b> assembly name of the assembly used for gene annotation, e.g. <code>"GRCh38.p14"</code> for Homo Sapiens release 115.
  * [`organism_ontology_term_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#organism_ontology_term_id) [`str`] - NCBITaxon ontology term corresponding to the main organism of the study, e.g. `"NCBITaxon:7227"` for *Drosophila Melanogaster*/.
  * *[`organism`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#organism)* [`str`] (Paired with `organism_ontology_term_id`) - Human-readable name assigned to the value of `organism_ontology_term_id`.
  * [`title`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#title) [`str`] - Main title of the study.
  * *[`shema_reference`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#shema_reference)* [`str`] - This schema URL: <code>"https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md"</code>.
  * *[`shema_version`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#shema_version)* [`str`] - This schema version: <code>"7.1.0+scfair1.0"</code>.
  * *&lt;Optional&gt;*[`analysis_pipeline`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#analysis_pipeline) [`str`] - A JSON entry based on the <code>['analysis schema'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md)</code> describing the analyis and annotation pipeline.
  * *&lt;Optional&gt;*[`batch_condition`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#batch_condition) [`str`] - One of the cell metadata keys in <code>obs</code> defining the <i>batches</i> (for integration).
  * *&lt;Optional&gt;*<i>[`citation`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#citation)</i> [`str`] - DOI of the study associated with the dataset.
  * *&lt;Optional&gt;*[`{column}_colors`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#column_colors) [`numpy.ndarray`] - Array of hex or color names for categorical metadata in `obs`.
  * *&lt;Optional&gt;*[`default_embedding`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#default_embedding) [`str`] - An embedding in <code>obsm</code> for the embedding to be displayed by default in any portal.
  * *&lt;Optional&gt;*[`X_approximate_distribution`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#X_approximate_distribution) [`str`] - One of `"count"` or `"normal"` describing the distribution of the main dataset.

## Spatial dataset (Visium)

If no description -> same as core

**AnnData.h5ad**
* [`AnnData.raw.X`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#x-matrix-layers)
* [`AnnData.X`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#x-matrix-layers)
* [`AnnData.obs`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obs-cell-metadata)
  * *(Same as core)*
  * **[`array_col`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#array_col)** [`int`] - Value of the column coordinate for the corresponding spot from the `array_col` field in `tissue_positions_list.csv` or `tissue_positions.csv`.
  * **[`array_row`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#array_row)** [`int`] - Value of the row coordinate for the corresponding spot from the `array_row` field in in `tissue_positions_list.csv` or `tissue_positions.csv`.
  * **[`in_tissue`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#in_tissue)** [`int`] - Value for the corresponding spot from the in_tissue field in `tissue_positions_list.csv` or `tissue_positions.csv` which is either 0 if the spot falls outside tissue or 1 if the spot falls inside tissue.
* [`AnnData.obsm`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obsm-embeddings) - Embeddings. Describe each embedding in the dataset.
  * **[`spatial`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatial)** [`numpy.ndarray`] - Spatial coordinates.
* [`Anndata.obsp`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obsp)
* [`Anndata.var` and `Anndata.raw.var`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#var-and-rawvar-gene-metadata)
* [`Anndata.varm`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#varm)
* [`Anndata.varp`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#varp)
* [`Anndata.uns`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#uns-dataset-metadata)
  * **[`spatial`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatial)** [`dict`] - This `dict` contains Space Ranger spatial outputs such as numerized full-res images and scalefactors. Structure of the dict is detailed below.
    * **[`spatial['is_single']`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatialis_single)** [`bool`] - True if the dataset represents one Space Ranger output for a single tissue section (`Visium Spatial Gene Expression`) or the dataset represents the output for a single array on a puck (`Slide-seqV2`).
    * <b>[<code>spatial[<i>library_id</i>]</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatiallibrary_id)</b> [`dict`] - This `dict` maps a unique identifier to the following values.

      ▸ <b>[<code>spatial[<i>library_id</i>]['images']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatiallibrary_idimages)</b> [`dict`] - This `dict` contains multiple numerized images.
        - <b>[<code>spatial[<i>library_id</i>]['images']['fullres']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatiallibrary_idimagesfullres)</b> [`numpy.ndarray`] - The full resolution image, converted to a `numpy.ndarray`.
        - <b>[<code>spatial[<i>library_id</i>]['images']['hires']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatiallibrary_idimageshires)</b> [`numpy.ndarray`] - The `tissue_hires_image.png` image, converted to a `numpy.ndarray`.

      ▸ <b>[<code>spatial[<i>library_id</i>]['scalefactors']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatiallibrary_idiscalefactors)</b> [`dict`] - This `dict` contains fields extracted from the `scalefactors_json.json`.
        - <b>[<code>spatial[<i>library_id</i>]['scalefactors']['spot_diameter_fullres']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatiallibrary_idscalefactorsspot_diameter_fullres)</b> [`float`] - This must be the value of the `spot_diameter_fullres` field from `scalefactors_json.json`.
        - <b>[<code>spatial[<i>library_id</i>]['scalefactors']['tissue_hires_scalef']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatiallibrary_idscalefactorstissue_hires_scalef)</b> [`float`] - This must be the value of the `tissue_hires_scalef` field from `scalefactors_json.json`.

## Genetic perturbation dataset (CRISPR screen, perturb-seq, ...)

If no description -> same as core

**AnnData.h5ad**
* [`AnnData.raw.X`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#x-matrix-layers)
* [`AnnData.X`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#x-matrix-layers)
* [`AnnData.obs`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obs-cell-metadata)
  * *(Same as core)*
  * **[`genetic_perturbation_id`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#genetic_perturbation_id)** [`str`] - `"na"` or one or more genetic perturbation identifiers in ascending lexical order separated by the delimiter `" || "` with no duplication of identifiers.
  * **[`genetic_perturbation_strategy`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#genetic_perturbation_strategy)** [`str`] - One of `"control"`, `"CRISPR activation screen"`, `"CRISPR interference screen"`, `"CRISPR knockout mutant"`, or `"CRISPR knockout screen"` (or `"no perturbations"` if `genetic_perturbation_id` is `"na"`).
* [`AnnData.obsm`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obsm-embeddings)
* [`Anndata.obsp`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obsp)
* [`Anndata.var` and `Anndata.raw.var`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#var-and-rawvar-gene-metadata)
* [`Anndata.varm`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#varm)
* [`Anndata.varp`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#varp)
* [`Anndata.uns`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#uns-dataset-metadata)
  * **[`genetic_perturbations`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbations)** [`dict`] - This `dict` contains multiple informations on the controls, targets, sequences, and genomic regions. Structure of the dict is detailed below.
    * <b>[<code>genetic_perturbations[<i>id</i>]</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbationsid)</b> [`dict`] - This `dict` maps a unique identifier for the genetic perturbation to the following values.
      ▸ <b>[<code>genetic_perturbations[<i>id</i>]['role']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbationsidrole)</b> [`str`] - One of `"control"` or `"targeting"`.<br/>
      ▸ <b>[<code>genetic_perturbations[<i>id</i>]['protospacer_sequence']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbationsidprotospacer_sequence)</b> [`str`] - Protospacer DNA sequence which represent the primary DNA nucleotide base codes. Its length MUST be 14-22 nucleotides in ('A', 'C', 'G', 'T').<br/>
      ▸ <b>[<code>genetic_perturbations[<i>id</i>]['protospacer_adjacent_motif']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbationsidprotospacer_adjacent_motif)</b> [`str`] - Protospacer adjacent motif (PAM) formatted as `"3' <b>MOTIF</b>"` such as `"3' <b>NGG</b>"`. **MOTIF** sequence is a string composed of IUPAC amino acid codes ('A', 'B', 'C', ..., 'W', 'Y').<br/>
      ▸ <i><b>[<code>genetic_perturbations[<i>id</i>]['derived_genomic_regions']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbationsidderived_genomic_regions)</b></i> [`list[str]`] - List of unique genomic region matched by [GuideScan2](https://link.springer.com/article/10.1186/s13059-025-03488-8) and formatted as `"SEQUENCE_ID:START-END(STRAND)"` such as `"1:12345-12346(+)"`.<br/>
      ▸ <i><b>[<code>genetic_perturbations[<i>id</i>]['derived_features']</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbationsidderived_features)</b></i> [`dict`] - This `dict` contains the list of features (such as genes) that are overlapping these genomic regions. It is mapped as a dict of gene_id / gene_name.
        - <i><b>[<code>genetic_perturbations[<i>id</i>]['derived_features'][feature_id]</code>](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbationsidderived_featuresfeature_id)</b></i> [`str`] - The key is the <code>gene_id</code> attribute from the corresponding gene reference of the <code>organism_ontology_term_id</code> for a feature that overlapped a genomic region in <code>genetic_perturbations[<i>id</i>]['derived_genomic_regions']</code> by at least one nucleotide. Value is the <code>gene_name</code>.

## scATAC dataset (scATAC, multimodal)

If no description -> same as core

**AnnData.h5ad**
* **[`AnnData.raw.X`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md#x-matrix-layers)** - For paired assays (multiome), this should be the gene expression matrix (RNA data).
* **[`AnnData.X`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md#x-matrix-layers)** - For unpaired assays (scATAC), this should be the gene activity matrix.
* [`AnnData.obs`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obs-cell-metadata)
* [`AnnData.obsm`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obsm-embeddings)
* [`Anndata.obsp`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#obsp)
* [`Anndata.var` and `Anndata.raw.var`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#var-and-rawvar-gene-metadata)
* [`Anndata.varm`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#varm)
* [`Anndata.varp`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#varp)
* [`Anndata.uns`](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#uns-dataset-metadata)

**Additional files:**
* **[scATAC assets](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md#scatac-seq-assets)** - Fragment/Peak files
