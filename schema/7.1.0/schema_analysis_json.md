# Schema

Contact: vincent.gardeux@epfl.ch

Document Status: _Drafting_

Version: 7.1.0+scfair1.0

Current schema: **analysis_json**

## Schema split

The scFAIR schema is split into multiple part, to differentiate the metadata specific to certain modalities:
- The **core** schema ['schema.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md) is the main schema for all types of single-cell data (scRNA-seq, scATAC-seq, perturbation, spatial, ...). It describes the core metadata between all modalities.
- This **analysis_json** schema ['schema_analysis_json.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md) describes the analysis pipeline, including software and versionning (cellranger, seurat, scanpy, ...), methods and parameters (umap, louvain, ...), and eventual Docker images
- The **spatial** schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md) describes the additional metadata that are specific to spatial datasets (Visium)
- The **perturb** schema ['schema_perturb.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md) describes the additional metadata that are specific to perturbation datasets (CRISPR screens, perturb-seq, ...)
- The **atac** schema ['schema_atac.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md) describes the additional metadata that are specific to scATAC datasets (scATAC-seq, multiomics)

## Overview

This schema describes the pipeline used (preprocessing, downstream analysis, annotation). It describes (in order), all the tools, parameters and versions that were used to generate the results. Eventually, for each step, a Docker image or a conda environment can be referenced to maximize reproducibility.

This document is organized by:

* [General requirements](#general-requirements)
* [`X` (Matrix layers)](#x-matrix-layers), which describe the data required for different assays
* [`obs` (Cell metadata)](#obs-cell-metadata), which describe each cell in the dataset
* [`obsm` (Embeddings)](#obsm-embeddings), which describe each embedding in the dataset
* [`obsp`](#obsp), which describe pairwise annotation of observations
* [`var` and `raw.var` (Gene metadata)](#var-and-rawvar-gene-metadata), which describe each gene in the dataset
* [`varm`](#varm), which describe multi-dimensional annotation of variables/features
* [`varp`](#varp), which describe pairwise annotation of variables/features
* [`uns` (Dataset metadata)](#uns-dataset-metadata), which describe the dataset as a whole

## General Requirements

**JSON.** The canonical format adopted by scFAIR for storing all analyses is JSON. The schema requirements and definitions of each entry are defined below.

**Important note on types** For simplification, and for not entering into the HDF5 typing complexity, all types below are written as python3 types, as expected by the AnnData package.<br/>
It is important to understand, for tool/language cross-compatibilities that all these python types are valid only when loading the .h5ad in an AnnData object in Python. However, the .h5ad is an HDF5 file, so it has its own intrisic typing, chunking, indexing which makes it compatible with any computing language supporting HDF5 file loading, through dedicated computing language-specific packages.<br/>
Finally, please note that a python3 `str` is a sequence of Unicode code points, which is stored null-terminated and UTF-8-encoded by AnnData.

**Reserved Names**. The names of metadata fields MUST NOT start with `"__"`. The names of the metadata fields specified by the schema are **reserved** for the purposes and specifications described in the schema.

**Unique Names**. The names of schema and data submitter metadata fields in `obs` and `var` MUST be unique. For example, duplicate `"feature_biotype"` keys in AnnData `var` are not allowed.

Reserved Names from previous schema versions that have since been deprecated MUST NOT be present in datasets:

## BLABLA
