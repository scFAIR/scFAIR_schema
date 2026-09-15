# Schema

Maintainer: vincent.gardeux@epfl.ch

Document Status: _Drafting_

Version: 7.1.0+scfair1.0

Current schema: **core**

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED" "MAY", and "OPTIONAL" in this document are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14), [RFC2119](https://www.rfc-editor.org/rfc/rfc2119.txt), and [RFC8174](https://www.rfc-editor.org/rfc/rfc8174.txt) when, and only when, they appear in all capitals, as shown here.

## Schema versioning

The scFAIR schema version is based on [Semantic Versioning](https://semver.org/) and matches major/minor versions of the CZI CELLxGENE schemas (see below for exact fork).

Since this is the first fork of the CZI CELLxGENE schema, all changes are documented in the scFAIR schema [Changelog](#appendix-a-changelog).

## Schema split

The scFAIR schema is split into multiple part, to differentiate the metadata specific to certain modalities:
- This **core** schema ['schema.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md) is the main schema for all types of single-cell data (scRNA-seq, scATAC-seq, perturbation, spatial, ...). It describes the core metadata between all modalities.
- The **analysis_json** schema ['schema_analysis_json.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md) describes the analysis pipeline, including software and versionning (cellranger, seurat, scanpy, ...), methods and parameters (umap, louvain, ...), and eventual Docker images
- The **spatial** schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md) describes the additional metadata that are specific to spatial datasets (Visium)
- The **perturb** schema ['schema_perturb.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md) describes the additional metadata that are specific to perturbation datasets (CRISPR screens, perturb-seq, ...)
- The **atac** schema ['schema_atac.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md) describes the additional and modified metadata that are specific to scATAC datasets (scATAC-seq, multiomics)

## Acknowledgments

This document was forked and extended from [schema.md](https://github.com/chanzuckerberg/single-cell-curation/blob/main/schema/7.1.0/schema.md), a metadata schema made for CZI CELLxGENE.

## Background

This document describes the schema, a type of contract, that scFAIR requires all datasets to adhere to so that it can enable searching, filtering, and integration of datasets across platforms.

Note that the requirements in the schema are just the minimum required information. Datasets may have additional metadata.

## Overview

This schema supports multiple assay types. Each assay takes the form of one or more two-dimensional matrices whose values are quantitative measures of the phenotypes of cells.

The schema additionally describes how the dataset, genes, and cells are annotated to describe the biological and technical characteristics of the data.

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

**AnnData.** The canonical data format adopted by scFAIR is HDF5-backed [AnnData](https://anndata.readthedocs.io/en/latest) as written by AnnData version 0.8.0 or greater. The on-disk format must be [AnnData specification (v0.1.0)](https://anndata.readthedocs.io/en/latest/fileformat-prose.html#anndata-specification-v0-1-0). Part of the rationale for selecting this format is to allow resources to access both the data and metadata within a single file. The schema requirements and definitions for the AnnData `X`, `obs`, `var`, `raw.var`, `obsm`, and `uns` attributes are described below.

**Important note on types** For simplification, and for not entering into the HDF5 typing complexity, all types below are written as python3 types, as expected by the AnnData package.<br/>
It is important to understand, for tool/language cross-compatibilities that all these python types are valid only when loading the .h5ad in an AnnData object in Python. However, the .h5ad is an HDF5 file, so it has its own intrisic typing, chunking, indexing which makes it compatible with any computing language supporting HDF5 file loading, through dedicated computing language-specific packages.<br/>
Finally, please note that a python3 `str` is a sequence of Unicode code points, which is stored null-terminated and UTF-8-encoded by AnnData.

**Reserved Names**. The names of metadata fields MUST NOT start with `"__"`. The names of the metadata fields specified by the schema are **reserved** for the purposes and specifications described in the schema.

**Unique Names**. The names of schema and data submitter metadata fields in `obs` and `var` MUST be unique. For example, duplicate `"feature_biotype"` keys in AnnData `var` are not allowed.

Reserved Names from previous schema versions that have since been deprecated MUST NOT be present in datasets:

<table>
<thead>
  <tr>
    <th>Reserved Name</th>
    <th>AnnData</th>
    <th>Deprecated in</th>
  </tr>
</thead>
<tbody>
 <tr>
    <td>organism</td>
    <td>obs</td>
    <td>CELLxGENE schema 6.0.0</td>
  </tr>
  <tr>
    <td>organism_ontology_term_id</td>
    <td>obs</td>
    <td>CELLxGENE schema 6.0.0</td>
  </tr>
  <tr>
  <tr>
    <td>ethnicity</td>
    <td>obs</td>
    <td>CELLxGENE schema 3.0.0</td>
  </tr>
  <tr>
    <td>ethnicity_ontology_term_id</td>
    <td>obs</td>
    <td>CELLxGENE schema 3.0.0</td>
  </tr>
  <tr>
    <td>X_normalization</td>
    <td>uns</td>
    <td>CELLxGENE schema 3.0.0</td>
  </tr>
  <tr>
    <td>default_field</td>
    <td>uns</td>
    <td>CELLxGENE schema 2.0.0</td>
  </tr>
  <tr>
    <td>layer_descriptions</td>
    <td>uns</td>
    <td>CELLxGENE schema 2.0.0</td>
  </tr>
  <tr>
    <td>tags</td>
    <td>uns</td>
    <td>CELLxGENE schema 2.0.0</td>
  </tr>
   <tr>
    <td>version</td>
    <td>uns</td>
    <td>CELLxGENE schema 2.0.0</td>
  </tr>
  <tr>
    <td>contributors</td>
    <td>uns</td>
    <td>CELLxGENE schema 1.1.0</td>
  </tr>
  <tr>
    <td>preprint_doi</td>
    <td>uns</td>
    <td>CELLxGENE schema 1.1.0</td>
  </tr>
  <tr>
    <td>project_description</td>
    <td>uns</td>
    <td>CELLxGENE schema 1.1.0</td>
  </tr>
  <tr>
    <td>project_links</td>
    <td>uns</td>
    <td>CELLxGENE schema 1.1.0</td>
  </tr>
  <tr>
    <td>project_name</td>
    <td>uns</td>
    <td>CELLxGENE schema 1.1.0</td>
  </tr>
  <tr>
    <td>publication_doi</td>
    <td>uns</td>
    <td>CELLxGENE schema 1.1.0</td>
  </tr>
</tbody>
</table>

**Redundant Metadata**. It is STRONGLY RECOMMENDED to avoid multiple metadata fields containing identical or similar information.

**No Personal Identifiable Information (PII)**. This is not strictly enforced by validation because it is difficult to predict what is and is not PII; however, data submitters and curators MUST agree with this requirement:

> It is my responsibility to ensure that this data is not identifiable. In particular, I commit that I will remove any [direct personal identifiers](https://docs.google.com/document/d/1sboOmbafvMh3VYjK1-3MAUt0I13UUJfkQseq8ANLPl8/edit) in the metadata portions of the data.

This includes names, emails, exact date of birth, or other PII for researchers or curators involved in the data generation and submission.

**Ontology-dependent Metadata**. scFAIR requires ontology terms to enable search, comparison, and integration of data. With the exception of Cellosaurus, ontology terms for cell metadata MUST use [OBO-format identifiers](http://www.obofoundry.org/id-policy.html), meaning a CURIE (prefixed identifier) of the form **Ontology:Identifier**. For example, [EFO:0000001](https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0000001) is a term in the Experimental Factor Ontology (EFO). Cellosaurus requires a prefixed identifier of the form **Ontology_Identifier** such as [CVCL_1P02](https://www.cellosaurus.org/CVCL_1P02).

The most accurate ontology term MUST always be used. If an exact or approximate ontology term is not available, a new term may be requested:

- For the [Cell Ontology], data submitters may [suggest a new term](https://github.com/obophenotype/cell-ontology/issues/new?assignees=bvarner-ebi&labels=new+term+request%2C+scFAIR&template=a_adding_term_scFAIR.md&title=%5BNTR-cxg%5D).

  To meet scFAIR schema requirements, the most accurate available CL term MUST be used until the new term is available. For example if `cell_type_ontology_term_id` describes a relay interneuron, but the most accurate available term in the CL ontology is [CL:0000099](https://www.ebi.ac.uk/ols4/ontologies/cl/classes?obo_id=CL%3A0000099) for *interneuron*, then the interneuron term can be used to fulfill this requirement and ensures that users searching for "neuron" are able to find these data.  If no appropriate term can be found (e.g. the cell type is unknown), then `"unknown"` MUST be used. Users will still be able to access more specific cell type annotations that have been submitted with the dataset (but aren't required by the schema).

Terms documented as obsolete in an ontology MUST NOT be used. For example, [EFO:0009310](https://www.ebi.ac.uk/ols4/ontologies/efo/classes/http%253A%252F%252Fwww.ebi.ac.uk%252Fefo%252FEFO_0009310) for *obsolete_10x v2* was marked as obsolete in EFO version 3.31.0 and replaced by [EFO:0009899](https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0009899) for *10x 3' v2*.

Of note, for tissue, cell type, and stage terms, the collected-metazoan or composite-metazoan version of the [Uberon multi-species anatomy ontology] SHOULD be used. The collected ontology merges Uberon itself with the CL ontology, and several taxon-specific ontologies (e.g., FBbt, ZFA, FBdv, HsapDv), describing anatomy, and developmental and life stages. The composite ontology is derived from the collected ontology, by removing, wherever possible, the taxon-specific terms, and mapping them to the corresponding taxon-neutral terms from Uberon, resulting in less redundancy (see [Combined Multispecies Ontologies document](https://obophenotype.github.io/uberon/combined_multispecies/) for more information). <br />
- Whenever possible, the taxon-neutral Uberon term SHOULD be used.
- A taxon-specific term MUST be used if it is the most precise term available, and corresponds to the correct taxon for the experiment.<br />
- From the composite and collected versions of Uberon, any term descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/uberon/classes?obo_id=UBERON%3A0001062"><code>UBERON:0001062</code></a> for <i>anatomical entity</i>, <a href="https://www.ebi.ac.uk/ols4/ontologies/cl/terms?obo_id=CL:0000000"><code>CL:0000000</code></a> for <i>cell</i>, <a href="https://www.ebi.ac.uk/ols4/ontologies/uberon/classes?obo_id=UBERON%3A0000105"><code>UBERON:0000105</code></a> for <i>life cycle stage</i>, or any term from an imported ontology cross-referenced to them, MUST be used.

**Ontologies by clade.** The ontologies recognised by this schema fall into a **taxon-neutral core** (`UBERON:`, `CL:`, `MONDO:`, `PATO:`, `EFO:`, `CHEBI:`, `uniprot:`) and a set of **clade-specific extensions** (`EHDAA2:`, `EMAPA:`, `MA:`, `ZFA:`, `ZFS:`, `XAO:`, `FBbt:`, `FBdv:`, `WBbt:`, `WBls:`, `HsapDv:`, `MmusDv:`, `CEPH:`, `CTENO:`, `PORO:`, `HAO:` for Metazoa; `PO:`, `TO:`, `PECO:`, `PSO:` for <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i>). Whenever a taxon-neutral term is an adequate description it SHOULD be preferred; a clade-specific term MUST be used where it is more precise, or where the core does not cover the clade at all.

[Appendix B](#appendix-b-relevant-ontologies) is the authoritative index. For each ontology it gives the clade covered, the fields it applies to, the **validation root** its terms MUST descend from, and whether it is distributed inside a Uberon multi-species product. The field definitions below refer to Appendix B rather than restating it.

Note that the `UBERON:0001062` / `CL:0000000` / `UBERON:0000105` descendant requirement stated immediately above is satisfied only by ontologies that are actually merged into a Uberon multi-species product. Ontologies marked **standalone** in Appendix B &mdash; currently `HAO:` and the four Planteome ontologies &mdash; carry no Uberon or CL cross-reference, and their terms MUST instead be validated against the root given in Appendix B.

## `X` (Matrix Layers)

The data stored in the `AnnData.X` data matrix is the main dataset viewable in a portal resource. For `AnnData.X`, `AnnData.raw.X`, and all layers, if a data matrix contains 50% or more values that are zeros, it MUST be encoded as a [`scipy.sparse.csr_matrix`](https://docs.scipy.org/doc/scipy/reference/generated/scipy.sparse.csr_matrix.html) with zero values encoded as <a href="https://docs.scipy.org/doc/scipy/tutorial/sparse.html#sparse-arrays-implicit-zeros-and-duplicates">implicit zeros</a>.

scFAIR's matrix layer requirements are tailored to optimize data reuse. Because each assay has different characteristics, the requirements differ by assay type. In general, scFAIR requires submission of "raw" data suitable for computational reuse when a standard raw matrix format exists for an assay. It is STRONGLY RECOMMENDED to also include a "normalized" matrix with processed values ready for data analysis and suitable for visualization in scFAIR Explorer. So that scFAIR's data can be provided in download formats suitable for both R and Python, the schema imposes the following requirements:

*   All matrix layers MUST have the same shape, and have the same cell labels and gene labels.
*   Because it is impractical to retain all barcodes in raw and normalized matrices, any cell filtering MUST be applied to both.
    By contrast, those wishing to reuse datasets require access to raw gene expression values, so genes SHOULD NOT be filtered from either dataset.
    Summarizing, any cell barcodes that are removed from the data MUST be filtered from both raw and normalized matrices and genes SHOULD NOT be filtered from the raw matrix.
*   Any genes that publishers wish to filter from the normalized matrix MAY have their values replaced by zeros and MUST be flagged in the column [`feature_is_filtered`](#feature_is_filtered) of [`var`](#var-and-rawvar-gene-metadata), which will mask them from exploration.
*   Additional layers provided at author discretion MAY be stored using author-selected keys, but MUST have the same cells and genes as other layers. It is STRONGLY RECOMMENDED that these layers have names that accurately summarize what the numbers in the layer represent (e.g. `"counts_per_million"`, `"SCTransform_normalized"`, or `"RNA_velocity_unspliced"`).

## `obs` (Cell Metadata)

`obs` is a [`pandas.DataFrame`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html).

**Note:** Terms in italic are auto-filled by the CZI CELLxGENE submission pipeline. scFAIR schema now consider them as required.

### index

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>index</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code></td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        The index of the pandas.DataFrame MUST contain unique identifiers for observations (e.g. unique cell names).<br/>
        Of note, in the h5ad file, this is stored as an attribute of <code>obs</code> named <code>_index</code>, pointing to an existing <code>obs</code> metadata.
      </td>
    </tr>
</tbody></table>

### assay_ontology_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>assay_ontology_term_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be an EFO term and either:
          <ul>
            <li>the most accurate descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0002772"><code>"EFO:0002772"</code></a> for <i>assay by molecule</i> excluding <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> while allowing its descendants</li>
            <li>the most accurate descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010183"><code>"EFO:0010183"</code></a>  for <i>single cell library construction</i> excluding <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> while allowing its descendants</li>
          </ul>
          If <code>assay_ontology_term_id</code> is either a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> or <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030062"><code>"EFO:0030062"</code></a> for <i>Slide-seqV2</i> then all observations MUST contain the same value.<br/><br/>
          If <code>assay_ontology_term_id</code> is either <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010891"><code>"EFO:0010891"</code></a> for <i>scATAC-seq</i> or its descendants, there are additional requirements for separate fragments file assets documented in <a href="#scatac-seq-assets">scATAC-seq assets</a>.<br/><br/>
          An assay based on 10X Genomics products SHOULD be the most accurate descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008995"><code>"EFO:0008995"</code></a> for <i>10x technology</i>.<br/><br/>
          An assay based on <i>SMART (Switching Mechanism at the 5' end of the RNA Template) or SMARTer technology</i> SHOULD either be <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010184"><code>"EFO:0010184"</code></a> for <i>Smart-like</i> or preferably its most accurate descendant.<br/><br/>
          Recommended values for specific assays:
          <table>
            <thead>
              <tr>
                <th>For</th>
                <th>Use</th>
              </tr>
            </thead>
            <tbody>
              <tr>
                <td><i>10x 3' v2</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0009899"><code>"EFO:0009899"</code></a></td>
              </tr>
              <tr>
                <td><i>10x 3' v3</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0009922"><code>"EFO:0009922"</code></a></td>
              </tr>
              <tr>
                <td><i>10x 3' v4</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022604"><code>"EFO:0022604"</code></a></td>
              </tr>
              <tr>
                <td><i>10x 5' v1</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0011025"><code>"EFO:0011025"</code></a></td>
              </tr>
              <tr>
                <td><i>10x 5' v2</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0009900"><code>"EFO:0009900"</code></a></td>
              </tr>            <tr>
                <td><i>10x 5' v3</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022605"><code>"EFO:0022605"</code></a></td>
              </tr>            </tr>            <tr>
                <td><i>10x multiome</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030059"><code>"EFO:0030059"</code></a></td>
              </tr>
              <tr>
                <td><i>Smart-seq2</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008931"><code>"EFO:0008931"</code></a></td>
              </tr>
              <tr>
                <td><i>Visium Spatial Gene Expression V1</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022857"><code>"EFO:0022857"</code></a></td>
              </tr>
              <tr>
                <td><i>Visium CytAssist Spatial Gene Expression, 6.5mm</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022859"><code>"EFO:0022859"</code></a></td>
              </tr>
              <tr>
                <td><i>Visium CytAssist Spatial Gene Expression, 11mm</i></td>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022860"><code>"EFO:0022860"</code></a></td>
              </tr>
            </tbody>
          </table>
        </td>
    </tr>
</tbody></table>

### *assay*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>assay</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
          This MUST be the human-readable name assigned to the value of <code>assay_ontology_term_id</code>.
      </td>
    </tr>
</tbody></table>

### tissue_type

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>tissue_type</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        This MUST be one of:
        <ul>
          <li><code>"cell line"</code></li>
          <li><code>"organoid"</code></li>
          <li><code>"primary cell culture"</code></li>
          <li><code>"tissue"</code></li>
        </ul>
        For <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i>: callus is <code>"tissue"</code> (annotated with <a href="http://browser.planteome.org/amigo/term/PO:0005052"><code>PO:0005052</code></a> for <i>plant callus</i> or its most accurate descendant); a cell suspension from a catalogued immortalised line such as BY-2 is <code>"cell line"</code>, one established for the experiment from primary explant is <code>"primary cell culture"</code>; protoplast isolation is a dissociation method rather than a <code>tissue_type</code>, so the value MUST describe the source material.
      </td>
    </tr>
</tbody></table>

### tissue_ontology_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>tissue_ontology_term_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        Please refer to <a href="#appendix-b-relevant-ontologies">Appendix B</a> for relevant anatomical ontologies according to your species.<br/><br/>      
        If <code>tissue_type</code> is <code>"cell line"</code>, this MUST be a Cellosaurus term.<br/><br/>
        If <code>tissue_type</code> is <code>"primary cell culture"</code>, this MUST follow the requirements for <code>cell_type_ontology_term_id</code>.<br/><br/>
        If <code>tissue_type</code> is <code>"organoid"</code>, this MUST NOT be <a href="https://www.ebi.ac.uk/ols4/ontologies/uberon/classes?obo_id=UBERON%3A0000922"><code>UBERON:0000922</code></a> for <i>embryo</i>.<br/>
        <ul>
          <li>If the organoid is an embryoid, it is STRONGLY RECOMMENDED that the value is <a href="https://www.ebi.ac.uk/ols4/ontologies/uberon/classes?obo_id=UBERON%3A0014374"><code>UBERON:0014374</code></a> for <i>embryoid body</i>.</li>
          <li>If the organoid is a gastruloid, it is STRONGLY RECOMMENDED that the value is <a href="https://www.ebi.ac.uk/ols4/ontologies/uberon/classes?obo_id=UBERON%3A0004734"><code>UBERON:0004734</code></a> for <i>gastrula</i>.</li>
        </ul>
        Otherwise, if <code>tissue_type</code> is <code>"organoid"</code> or <code>"tissue"</code> then MUST be the most accurate descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/uberon/classes?obo_id=UBERON%3A0001062"><code>UBERON:0001062</code></a> for <i>anatomical entity</i> (or any term from an imported ontology cross-referenced to it, e.g., <a href="https://www.ebi.ac.uk/ols4/ontologies/fbbt/classes?obo_id=FBBT%3A10000000"><code>FBbt:10000000</code></a> for <i>anatomical entity</i> in <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A7227"><code>NCBITaxon:7227</code></a> for <i>Drosophila melanogaster</i>).<br/><br/>
       If <code>organism_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i> or one of its descendants, the <code>"cell line"</code> and <code>"primary cell culture"</code> branches above are unchanged, but Uberon does not cover Viridiplantae: the final <code>"organoid"</code>/<code>"tissue"</code> branch is instead the most accurate descendant of <a href="http://browser.planteome.org/amigo/term/PO:0025131"><code>PO:0025131</code></a> for <i>plant anatomical entity</i>, excluding <a href="http://browser.planteome.org/amigo/term/PO:0009002"><code>PO:0009002</code></a> for <i>plant cell</i> and its descendants, which belong in <code>cell_type_ontology_term_id</code>. There is no Uberon fallback for Viridiplantae, so Notes 1 and 2 below apply to Metazoa only.<br/><br/>
       <b>Note 1:</b> A taxon-specific term MUST be used if it is the most precise term available, and corresponds to the correct taxon for the experiment. Otherwise, a taxon-neutral Uberon term SHOULD be used.<br/><br/>
       <b>Note 2:</b> The value MAY be a combination of terms in ascending lexical order separated by the delimiter <code>" || "</code> with no duplication of terms. For example, for <i>Drosophila Melanogaster</i>, this entry could be <code>"FBbt:00000002 || FBbt:00000015"</code> for <code>"abdomen || thorax"</code> as a substitute for "body without head" since this term does not exist in the ontology. In addition, even if <code>"FBbt:00000002 || FBbt:00000004 || FBbt:00000015"</code> for <code>"abdomen || head || thorax"</code> could be used to describe the whole fly, we rather recommend using the unique term <code>"UBERON:0000468"</code> for <code>"multicellular organism"</code>.
      </td>
  </tr>
</tbody></table>

### *tissue*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>tissue</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        This MUST be one or more human-readable names for the terms in <code>tissue_ontology_term_id</code> in the same order separated by the delimiter <code>" || "</code>.<br/><br/>
        For example, if the value of <code>tissue_ontology_term_id</code> is <code>"FBbt:00000002 || FBbt:00000015"</code> then the value of <code>tissue</code> MUST be <code>"abdomen || thorax"</code>.
      </td>
    </tr>
</tbody></table>
<br/>

### cell_type_ontology_term_id

<table><tbody>
  <tr>
    <th>Key</th>
    <td><code>cell_type_ontology_term_id</code></td>
  </tr>
  <tr>
    <th>Requirement</th>
    <td>REQUIRED</td>
  </tr>
  <tr>
    <th>Type</th>
    <td>categorical with <code>str</code> categories.</td>
  </tr>
  <tr>
    <th>Value</th>
    <td>
      Please refer to <a href="#appendix-b-relevant-ontologies">Appendix B</a> for relevant cell ontologies according to your species.<br/><br/>
      This MUST be <code>"unknown"</code> when:
      <ul>
        <li>
          no appropriate term can be found (e.g. the cell type is unknown)
        </li>
        <li>
          <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i>,<br/><code>uns['spatial']['is_single']</code> is <code>True</code>,<br/>and the corresponding value of <code>in_tissue</code> is <code>0</code>
        </li>
      </ul>
      If <code>tissue_type</code> is <code>"cell line"</code>, this MAY be <code>"na"</code>, but then all observations where <code>tissue_type</code> is <code>"cell line"</code> MUST be <code>"na"</code>.<br/><br/>The following CL terms MUST NOT be used:
      <ul>
        <li><a href="https://www.ebi.ac.uk/ols4/ontologies/cl/terms?obo_id=CL:0000255"><code>"CL:0000255"</code></a> for <i>eukaryotic cell</i></li>
        <li><a href="https://www.ebi.ac.uk/ols4/ontologies/cl/terms?obo_id=CL:0000257"><code>"CL:0000257"</code></a> for <i>Eumycetozoan cell</i></li>
        <li><a href="https://www.ebi.ac.uk/ols4/ontologies/cl/terms?obo_id=CL:0000548"><code>"CL:0000548"</code></a> for <i>animal cell</i></li>
      </ul>
      Otherwise, this MUST be the most accurate descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/cl/terms?obo_id=CL:0000000"><code>CL:0000000</code></a> for <i>cell</i> (or any term from an imported ontology cross-referenced to it, e.g., <a href="https://www.ebi.ac.uk/ols4/ontologies/fbbt/terms?obo_id=FBbt:00007002"><code>FBbt:00007002</code></a> for <i>cell</i> in <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/terms?obo_id=NCBITaxon:7227"><code>NCBITaxon:7227</code></a> for <i>Drosophila melanogaster</i>).<br/><br/>
       If <code>organism_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i> or one of its descendants, the <code>"unknown"</code>, <code>"na"</code> and excluded-term rules above are unchanged, but CL contains no non-obsolete plant cell terms: the final branch is instead the most accurate descendant of <a href="http://browser.planteome.org/amigo/term/PO:0009002"><code>PO:0009002</code></a> for <i>plant cell</i>. There is no CL fallback for Viridiplantae, so Notes 1 and 2 below apply to Metazoa only.<br/><br/>
       <b>Note 1:</b> A taxon-specific term MUST be used if it is the most precise term available, and corresponds to the correct taxon for the experiment. Otherwise, a taxon-neutral CL term SHOULD be used.<br/><br/>
       <b>Note 2:</b> The value MAY be a combination of terms in ascending lexical order separated by the delimiter <code>" || "</code> with no duplication of terms. For example, for <i>Drosophila Melanogaster</i>, this entry could be <code>"FBbt:00003731 || FBbt:00003736"</code> for <code>"T4 neuron || T5 neuron"</code>. Even though, in this case, <code>"FBbt:00003726"</code> for <code>"T neuron"</code> could be used (as direct ascendant of both terms), this feature is meant to allow for more precise annotation of cell-types, preventing loss of information.
    </td>
  </tr>
</tbody></table>

### *cell_type*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>cell_type</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED if <code>cell_type_ontology_term_id</code> is present; otherwise this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be <code>"na"</code> if the value of  <code>cell_type_ontology_term_id</code> is <code>"na"</code>.<br/><br/>
          This MUST be <code>"unknown"</code> if the value of  <code>cell_type_ontology_term_id</code> is <code>"unknown"</code>.<br/><br/>
          Otherwise, this MUST be one or more human-readable names for the terms in <code>cell_type_ontology_term_id</code> in the same order separated by the delimiter <code>" || "</code>.<br/><br/>
          For example, if the value of <code>cell_type_ontology_term_id</code> is <code>"FBbt:00003731 || FBbt:00003736"</code> then the value of <code>cell_type</code> MUST be <code>"T4 neuron || T5 neuron"</code>.
        </td>
    </tr>
</tbody></table>

### development_stage_ontology_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>development_stage_ontology_term_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        Please refer to <a href="#appendix-b-relevant-ontologies">Appendix B</a> for relevant developmental ontologies according to your species.<br/><br/>
        If <code>tissue_type</code> is <code>"cell line"</code>, this MUST be <code>"na"</code>.<br/><br/>If unavailable, this MUST be <code>"unknown"</code>.<br/><br/>
        Otherwise, this MUST be the most accurate descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/uberon/terms?obo_id=UBERON%3A0000105"><code>UBERON:0000105</code></a> for <i>life cycle stage</i> (or any term from an imported ontology cross-referenced to it, e.g., <a href="https://www.ebi.ac.uk/ols4/ontologies/hsapdv/terms?obo_id=HsapDv%3A0000001"><code>HsapDv:0000001</code></a> for <i>life cycle</i> in <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A9606"><code>NCBITaxon:9606</code></a> for <i>Homo sapiens</i>).<br/><br/>
          <b>Note 1:</b> When a taxon-specific developmental stage ontology is available for the organism under study, terms from that ontology MUST be preferred over taxon-neutral UBERON terms where a more precise match exists.<br/><br/>
          <b>Note 2:</b> The value MAY be a combination of terms in ascending lexical order separated by the delimiter <code>" || "</code> with no duplication of terms. This feature is meant to prevent loss of information when multiple samples are pooled but without a way to later demultiplex & assign each cell back to the original pre-pooled sample. For example, for <i>Homo Sapiens</i>, this entry could be <code>"HsapDv:0000124 || HsapDv:0000130"</code> for <code>"30-year-old stage || 36-year-old stage"</code> instead of <code>"HsapDv:0000238"</code> for <code>"fourth decade stage"</code> which would be a unique term, but with loss of information.<br/><br/>
          If <code>organism_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i> or one of its descendants, the <code>"na"</code> and <code>"unknown"</code> rules above are unchanged, but Uberon does not cover Viridiplantae: the value MUST instead be the most accurate descendant of <a href="http://browser.planteome.org/amigo/term/PO:0009012"><code>PO:0009012</code></a> for <i>plant structure development stage</i>.<br/><br/>
          The following organism-specific ontologies are recognized and their terms are valid in addition to UBERON:
          <table><tbody>
            <tr>
              <th>Organism</th>
              <th><code>organism_ontology_term_id</code></th>
              <th>Ontology prefix</th>
              <th>Notes</th>
            </tr>
            <tr>
              <td><i>Homo sapiens</i></td>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A9606"><code>NCBITaxon:9606</code></a></td>
              <td><code>HsapDv:</code></td>
              <td></td>
            </tr>
            <tr>
              <td><i>Mus musculus</i></td>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A10090"><code>NCBITaxon:10090</code></a></td>
              <td><code>MmusDv:</code></td>
              <td></td>
            </tr>
            <tr>
              <td><i>Caenorhabditis elegans</i></td>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A6239"><code>NCBITaxon:6239</code></a></td>
              <td><code>WBls:</code></td>
              <td></td>
            </tr>
            <tr>
              <td><i>Danio rerio</i></td>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A7955"><code>NCBITaxon:7955</code></a></td>
              <td><code>ZFS:</code></td>
              <td></td>
            </tr>
            <tr>
              <td><i>Drosophila melanogaster</i></td>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A7227"><code>NCBITaxon:7227</code></a></td>
              <td><code>FBdv:</code></td>
              <td></td>
            </tr>
            <tr>
              <td><i>Aedes aegypti</i></td>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A7159"><code>NCBITaxon:7159</code></a></td>
              <td><code>FBdv:</code></td>
              <td>
                No dedicated ontology exists;<br/>
                <code>FBdv:</code> is used as a proxy<br/>
                given the conservation of dipteran<br/>
                developmental stages.
              </td>
            </tr>
            <tr>
              <td><i>Ctenophora</i></td>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A10197"><code>NCBITaxon:10197</code></a></td>
              <td><code>CTENO:</code></td>
              <td>CTENO carries its own stage terms,<br/>e.g. <code>CTENO:0000024</code> for <i>cydippid stage</i>.</td>
            </tr>
            <tr>
              <td><i>Viridiplantae</i></td>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a></td>
              <td><code>PO:</code></td>
              <td>Descendant of <code>PO:0009012</code> for<br/><i>plant structure development stage</i>.<br/>No taxon-neutral fallback exists.</td>
            </tr>
          </tbody></table>    
          Terms from organism-specific ontologies MUST NOT be used for organisms not listed in the table above or in <a href="#appendix-b-relevant-ontologies">Appendix B</a> (please contact maintainer if you need to add one). For unlisted organisms, a taxon-neutral <code>UBERON:</code> term MUST be used. The clade-specific ontologies that cover anatomy only &mdash; <code>CEPH:</code>, <code>PORO:</code> and <code>HAO:</code> &mdash; provide no developmental stage terms.
      </td>
  </tr>
</tbody></table>

### *development_stage*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>development_stage</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        This MUST be <code>"na"</code> if the value of <code>development_stage_ontology_term_id</code> is <code>"na"</code>.<br/><br/>This MUST be <code>"unknown"</code> if the value of <code>development_stage_ontology_term_id</code> is <code>"unknown"</code>.<br/><br/>
        Otherwise, this MUST be one or more human-readable names for the terms in <code>development_stage_ontology_term_id</code> in the same order separated by the delimiter <code>" || "</code>.<br/><br/>
          For example, if the value of <code>development_stage_ontology_term_id</code> is <code>"HsapDv:0000124 || HsapDv:0000130"</code> then the value of <code>development_stage</code> MUST be <code>"30-year-old stage || 36-year-old stage"</code>.
      </td>
    </tr>
</tbody></table>

### sex_ontology_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>sex_ontology_term_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          If <code>tissue_type</code> is <code>"cell line"</code>, this MUST be <code>"na"</code>.<br/><br/>
          If unavailable, this MUST be <code>"unknown"</code>.<br/><br/>
          If <code>organism_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A6239"><code>"NCBITaxon:6239"</code></a> for <i>Caenorhabditis elegans</i>, this MUST be <a href="https://www.ebi.ac.uk/ols4/ontologies/pato/classes?obo_id=PATO%3A0000384"><code>"PATO:0000384"</code></a> for <i>male</i> or <a href="https://www.ebi.ac.uk/ols4/ontologies/pato/classes?obo_id=PATO%3A0001340"><code>"PATO:0001340"</code></a> for <i>hermaphrodite</i><br/><br/>
          If <code>organism_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i>, or a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A6040"><code>NCBITaxon:6040</code></a> for <i>Porifera</i> or <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A10197"><code>NCBITaxon:10197</code></a> for <i>Ctenophora</i>, a female/male distinction is frequently not applicable to the sampled organism. This MUST then be <a href="https://www.ebi.ac.uk/ols4/ontologies/pato/classes?obo_id=PATO%3A0001340"><code>"PATO:0001340"</code></a> for <i>hermaphrodite</i> for a monoecious plant or simultaneous hermaphrodite, <code>"na"</code> where no sex phenotype applies, or <i>female</i>/<i>male</i> for a dioecious plant or gonochoric individual whose sex is known. <code>"unknown"</code> MUST NOT be used to mean "not applicable"; it is reserved for a sex that exists but was not recorded.<br/><br/>
          Otherwise, this MUST be one of:
          <ul>
            <li><a href="https://www.ebi.ac.uk/ols4/ontologies/pato/classes?obo_id=PATO%3A0000383"><code>"PATO:0000383"</code></a> for  <i>female</i></li>
            <li><a href="https://www.ebi.ac.uk/ols4/ontologies/pato/classes?obo_id=PATO%3A0000384"><code>"PATO:0000384"</code></a> for  <i>male</i></li>
            <li><a href="https://www.ebi.ac.uk/ols4/ontologies/pato/classes?obo_id=PATO%3A0001340"><code>"PATO:0001340"</code></a> for  <i>hermaphrodite</i></li>
          </ul>
        </td>
    </tr>
</tbody></table>

### *sex*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>sex</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be <code>"na"</code> if the value of  <code>sex_ontology_term_id</code> is <code>"na"</code>.<br/><br/>This MUST be <code>"unknown"</code> if the value of  <code>sex_ontology_term_id</code> is <code>"unknown"</code>.<br/><br/>
          Otherwise, this MUST be the human-readable name assigned to the value of <code>sex_ontology_term_id</code>.
        </td>
    </tr>
</tbody></table>

### self_reported_ethnicity_ontology_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>self_reported_ethnicity_ontology_term_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        If <code>tissue_type</code> is <code>"cell line"</code>, this MUST be <code>"na"</code><br/><br/>
        If <code>organism_ontolology_term_id</code> is NOT <code>"NCBITaxon:9606"</code> for <i>Homo sapiens</i>, this MUST be <code>"na"</code>.<br/><br/>
        Otherwise, if <code>organism_ontolology_term_id</code> is <code>"NCBITaxon:9606"</code> for <i>Homo sapiens</i>, this MUST be <code>"unknown"</code> if unavailable; otherwise, this MUST meet the following requirements:
        <ul>
          <li>
            The value MUST be formatted as one or more AfPO or HANCESTRO terms in ascending lexical order separated by the delimiter <code>" || "</code> with no duplication of terms.
          </li>
          <li>
            Each AfPO or HANCESTRO term MUST be a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/hancestro/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FHANCESTRO_0601"><code>"HANCESTRO:0601"</code></a> for <i>ethnicity category</i> or <a href="https://www.ebi.ac.uk/ols4/ontologies/hancestro/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FHANCESTRO_0602"><code>"HANCESTRO:0602"</code></a> for <i>geography-based population category</i>.
          </li>
            For example, if the terms are <code>"HANCESTRO:0590</code> and <code>HANCESTRO:0580"</code> then the value of <code>self_reported_ethnicity_ontology_term_id</code> MUST be <code>"HANCESTRO:0580 || HANCESTRO:0590"</code>.
        </ul>
      </td>
    </tr>
</tbody></table>

### *self_reported_ethnicity*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>self_reported_ethnicity</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be <code>"na"</code> if the value of <code>self_reported_ethnicity_ontology_term_id</code> is <code>"na"</code>.<br/><br/>
          This MUST be <code>"unknown"</code> if the value of <code>self_reported_ethnicity_ontology_term_id</code> is <code>"unknown"</code>.<br/><br/>
          Otherwise, this MUST be one or more human-readable names for the terms in <code>self_reported_ethnicity_ontology_term_id</code> in the same order separated by the delimiter <code>" || "</code>.<br/><br/>
          For example, if the value of <code>self_reported_ethnicity_ontology_term_id</code> is <code>"HANCESTRO:0005 || HANCESTRO:0014"</code> then the value of <code>self_reported_ethnicity</code> MUST be <code>"European || Hispanic or Latin American"</code>.
        </td>
    </tr>
</tbody></table>

### strain_or_genetic_background_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>strain_or_genetic_background</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
          When existing and relevant, this MUST be a controlled value describing the experimentally relevant genetic background of a non human organism, such as strain, stock, cultivar, genotype, ecotype, variety, serovar, or breeding and crossing design. For example, strain can be described using EFO terms such as <code>"EFO:0002742"</code> for <code>"C57BL/6 congenically expressing H2g7 mouse strain"</code>.
      </td>
    </tr>
</tbody></table>

### strain_or_genetic_background

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>strain_or_genetic_background</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
          This MUST be the human-readable name assigned to the value of <code>strain_or_genetic_background_term_id</code> if set. Else, it is a free text describing the experimentally relevant genetic background of a non human organism, such as strain, stock, line, cultivar, genotype, ecotype, variety, serovar, or breeding and crossing design.
      </td>
    </tr>
</tbody></table>

### disease_ontology_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>disease_ontology_term_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be one of:
          <ul>
            <li><a href="https://www.ebi.ac.uk/ols4/ontologies/pato/classes?obo_id=PATO%3A0000461"><code>"PATO:0000461"</code></a> for <i>normal</i> or <i>healthy</i>.</li>
            <li>one or more MONDO terms in ascending lexical order separated by the delimiter <code>" || "</code> with no duplication of terms. For example, if the terms are <code>"MONDO:1030008"</code>, <code>"MONDO:0800349"</code>, <code>"MONDO:0004604"</code>, and <code>"MONDO:0043004"</code> then the value MUST be <code>"MONDO:0004604 || MONDO:0043004 || MONDO:0800349 || MONDO:1030008"</code>.</li>
          </ul>
          MONDO terms MUST be either:
          <ul>
            <li>a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/mondo/classes?obo_id=MONDO%3A0000001"><code>"MONDO:0000001"</code></a> for <i>disease</i></li>
            <li><a href="https://www.ebi.ac.uk/ols4/ontologies/mondo/classes?obo_id=MONDO%3A0021178"><code>"MONDO:0021178"</code></a> for <i>injury</i> or <b>preferably</b> its most accurate descendant</li>
          </ul>
          MONDO does not cover plant disease. If <code>organism_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i> or one of its descendants, this MUST instead be <a href="https://www.ebi.ac.uk/ols4/ontologies/pato/classes?obo_id=PATO%3A0000461"><code>"PATO:0000461"</code></a> for <i>normal</i> or <i>healthy</i>, or one or more descendants of <code>"PSO:0000001"</code> for <i>plant stress</i> in ascending lexical order separated by <code>" || "</code> with no duplication of terms. This is the biotic stress branch covering pathogen- and pest-induced disease, while the abiotic branch is covering environmental damage. <code>MONDO:</code> MUST NOT be used for Viridiplantae, and <code>PSO:</code> MUST NOT be used outside it. Where the study concerns a resistance or susceptibility trait rather than an observed disease state, it is STRONGLY RECOMMENDED to also record the matching descendant of <code>"TO:0000387"</code> for <i>plant trait</i> as an author-defined field. See <a href="#appendix-b-relevant-ontologies">Appendix B</a>.
        </td>
    </tr>
</tbody></table>

### *disease*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>disease</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be one or more human-readable names for the terms in <code>disease_ontology_term_id</code> in the same order separated by the delimiter <code>" || "</code>.<br/><br/>
          For example, if the value of <code>disease_ontology_term_id</code> is <code>"MONDO:0004604 || MONDO:0043004 || MONDO:0800349 || MONDO:1030008"</code> then the value MUST be <code>"Hodgkin's lymphoma, lymphocytic-histiocytic predominance || Weil's disease || atrial fibrillation, familial, 16 || mitral valve insufficiency"</code>.
        </td>
    </tr>
</tbody></table>

### experimental_condition_ontology_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>experimental_condition_ontology_term_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        Please refer to <a href="#appendix-b-relevant-ontologies">Appendix B</a> for relevant developmental ontologies according to your species.<br/><br/>
        The value MUST be either be <code>"na"</code> or one or more experimental condition term identifiers in ascending lexical order separated by the delimiter <code>" || "</code> with no duplication of identifiers.<br/><br/>
        For example, if the terms are <code>"uniprot:P05112"</code>, <code>"anti-uniprot:Q99467"</code>, <code>"EFO:0002757"</code>, <code>"CHEBI:16412"</code>, <code>"EFO:0001702"</code>, and <code>"CHEBI:41774"</code> then the value MUST be <code>"CHEBI:16412 || CHEBI:41774 || EFO:0001702 || EFO:0002757 || anti-uniprot:Q99467 || uniprot:P05112"</code>.<br/><br/>
        The value MUST be <code>"na"</code> when there is no experimental condition for this observation. If all observations are <code>"na"</code>, then this field MUST NOT be present.<br/><br/>
        If the experimental condition is a protein perturbation, then the value MUST include one or more UniProt terms such as <code>"uniprot:P08575"</code> for proteins or a UniProt term prefixed with <code>"anti-"</code> such as <code>"anti-uniprot:P08575"</code> for antibodies.<br/><br/>
        If the experimental condition is a chemical perturbation, then the value MUST include one or more descendants of <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_24431?lang=en"><code>"CHEBI:24431"</code></a> for <i>chemical entity</i>.<br/><br/>
        The following CHEBI terms MUST NOT be used:
        <ul>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_23367?lang=en"><code>"CHEBI:23367"</code></a> for <i>molecular entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_24431?lang=en"><code>"CHEBI:24431"</code></a> for <i>chemical entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_24835?lang=en"><code>"CHEBI:24835"</code></a> for <i>inorganic molecular entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_24867?lang=en"><code>"CHEBI:24867"</code></a> for <i>monoatomic ion</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_24870?lang=en"><code>"CHEBI:24870"</code></a> for <i>ion</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_25212?lang=en"><code>"CHEBI:25212"</code></a> for <i>metabolite</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_25367?lang=en"><code>"CHEBI:25367"</code></a> for <i>molecule</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_25699?lang=en"><code>"CHEBI:25699"</code></a> for <i>organic ion</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_33238?lang=en"><code>"CHEBI:33238"</code></a> for <i>monoatomic entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_33259?lang=en"><code>"CHEBI:33259"</code></a> for <i>elemental molecular entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_33497?lang=en"><code>"CHEBI:33497"</code></a> for <i>transition element molecular entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_33595?lang=en"><code>"CHEBI:33595"</code></a> for <i>cyclic compound</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_33674?lang=en"><code>"CHEBI:33674"</code></a> for <i>s-block molecular entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_33675?lang=en"><code>"CHEBI:33675"</code></a> for <i>p-block molecular entity</i>
          </li> 
           <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_36342?lang=en"><code>"CHEBI:36342"</code></a> for <i>subatomic particle</i> and its descendants
          </li>         
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_36357?lang=en"><code>"CHEBI:36357"</code></a> for <i>polyatomic entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_36358?lang=en"><code>"CHEBI:36358"</code></a> for <i>polyatomic ion</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_36914?lang=en"><code>"CHEBI:36914"</code></a> for <i>inorganic ion</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_37577?lang=en"><code>"CHEBI:37577"</code></a> for <i>heteroatomic molecular entity</i>
          </li>
          <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/chebi/classes/http%253A%252F%252Fpurl.obolibrary.org%252Fobo%252FCHEBI_50906?lang=en"><code>"CHEBI:50906"</code></a and its descendants> for <i>role</i> and its descendants
          </li>
        </ul>
        If the experimental condition is a diet perturbation, then the value MUST include either <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0002755"><code>"EFO:0002755"</code></a> for <i>diet</i> or its most accurate descendant.<br/><br/>
        If the experimental condition is a temperature perturbation, then the value MUST include <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0001702"><code>"EFO:0001702"</code></a> for <i>temperature</i>.<br/><br/>
        If <code>organism_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i> or one of its descendants, the value MAY also include one or more descendants of <a href="http://browser.planteome.org/amigo/term/PECO:0007359"><code>"PECO:0007359"</code></a> for <i>plant experimental condition</i>, covering the biotic and abiotic treatments, growing conditions and study types used in plant experiments (water deficit, red light, photoperiod, soil type, fertiliser, nutrients, growth hormone application). <code>PECO:</code> terms MUST NOT be used for any organism outside Viridiplantae.<br/><br/>
        No other values MUST be present for experimental conditions. 
      </td>
    </tr>
</tbody></table>

### *experimental_condition*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>experimental_condition</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED if <code>obs['experimental_condition_ontology_term_id']</code> is present; otherwise this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
          This MUST be <code>"na"</code> if the value of <code>experimental_condition_ontology_term_id</code> is <code>"na"</code>.<br/><br/>
          Otherwise, this MUST be one or more human-readable names for the terms in <code>experimental_condition_ontology_term_id</code> in the same order separated by the delimiter <code>" || "</code>.<br/><br/>
          If an antibody value is present, then the human-readable name is prefixed with <code>"anti-"</code> such as <code>"anti-PTPRC_HUMAN"</code>.<br/><br/>For example, if the value of <code>experimental_condition_ontology_term_id</code> is <code>"CHEBI:16412 || CHEBI:41774 || EFO:0001702 || EFO:0002757 || anti-uniprot:Q99467 || uniprot:P05112"</code> then the value of <code>experimental_condition</code> MUST be <code>"lipopolysaccharide || tamoxifen || temperature || high fat diet || anti-CD180_HUMAN || IL4_HUMAN"</code>.
      </td>
    </tr>
</tbody></table>

### *perturbation_types*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>perturbation_types</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED if <code>obs['experimental_condition_ontology_term_id']</code> or <code>obs['genetic_perturbation_id']</code> is present; otherwise this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          The value MUST be <code>"no perturbations"</code> when:
          <ul>
            <li>only one of <code>obs['experimental_condition_ontology_term_id']</code> or <code>obs['genetic_perturbation_id']</code> is present and its value is <code>"na"</code></li>
            <li>both <code>obs['experimental_condition_ontology_term_id']</code> and <code>obs['genetic_perturbation_id']</code> are present and their values are <code>"na"</code></li>
          </ul>
          Otherwise, the value MUST be the <b>set</b> of perturbation types present in the observation, limited to the following types:
            <ul>
              <li> <code>"chemical"</code></li>
              <li> <code>"diet"</code></li>
              <li> <code>"genetic"</code></li>
              <li> <code>"protein"</code></li>
              <li> <code>"temperature"</code></li>
              <li> <code>"environmental"</code></li>
            </ul>
          and formatted in ascending lexical order separated by the delimiter <code>" || "</code> with no duplication of elements.<br/><br/>
          If <code>experimental_condition_ontology_term_id</code> contains a <code>"CHEBI:"</code> term identifier, then <code>"chemical"</code> MUST be added to the set of values.<br/><br/>
          If <code>experimental_condition_ontology_term_id</code> contains the <code>"EFO:0002755"</code> term identifier or its descendants, then <code>"diet"</code> MUST be added to the set of values.<br/><br/>
          If <code>genetic_perturbation_term_id</code> is present and its value is not <code>"na"</code>, then <code>"genetic"</code> MUST be added to the set of values.<br/><br/>
          If <code>experimental_condition_ontology_term_id</code> contains a <code>"uniprot:"</code> term identifier, then <code>"protein"</code> MUST be added to the set of values.<br/><br/>
          If <code>experimental_condition_ontology_term_id</code> contains the <code>"EFO:0001702"</code> term identifier, then <code>"temperature"</code> MUST be added to the set of values.<br/><br/>
          If <code>experimental_condition_ontology_term_id</code> contains a <code>"PECO:"</code> term identifier, then the set MUST include the matching type: <code>"chemical"</code> for a chemical or nutrient application, <code>"diet"</code> for a nutrient regime, <code>"temperature"</code> for a thermal condition, and <code>"environmental"</code> for any other growing condition or treatment such as photoperiod, light quality, water deficit, soil type or biotic challenge. A <code>"PECO:"</code> term therefore never leaves the set empty, and <code>"no perturbations"</code> MUST NOT be used for an observation carrying one.
      </td>
    </tr>
</tbody></table>

### donor_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>donor_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          If <code>tissue_type</code> is <code>"cell line"</code>, this MUST be <code>"na"</code>; otherwise, this MUST NOT be <code>"na"</code>, but MUST be free-text that identifies a unique individual that data were derived from.<br/><br/>
          It is STRONGLY RECOMMENDED that this identifier be designed so that it is unique to all datasets, specifically datasets from a same collection (sharing the same DOI).<br/><br/>
          It is STRONGLY RECOMMENDED that <code>"pooled"</code> be used  for observations from a sample of multiple individuals that were not confidently assigned to a single individual through demultiplexing.<br/><br/>
          It is STRONGLY RECOMMENDED that <code>"unknown"</code> ONLY be used for observations in a dataset when it is not known which observations are from the same individual.
        </td>
    </tr>
</tbody></table>

### is_primary_data

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>is_primary_data</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>bool</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be <code>False</code> if <code>uns['spatial']['is_single']</code> is <code>False</code>.<br/><br/>
          This MUST be <code>True</code> if this is the canonical instance of this cellular observation and <code>False</code> if not.<br/><br/>
          This is commonly <code>False</code> for meta-analyses reusing data or for secondary views of data.
        </td>
    </tr>
</tbody></table>

### suspension_type

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>suspension_type</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be <code>"cell"</code>, <code>"nucleus"</code>, or <code>"na"</code>.<br/><br/>
          This MUST be the correct type for the corresponding assay:
          <table>
          <thead>
          <tr>
          <th>For Assay</th>
          <th>MUST Use</th>
          </tr>
          </thead>
          <tbody>
            <tr>
              <td><i>10x transcription profiling</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030080"><code>EFO:0030080</code></a>] and its descendants</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr> 
            <tr>
              <td><i>ATAC-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0007045"><code>EFO:0007045</code></a>] and its descendants</td>
              <td><code>"nucleus"</code></td>
           </tr>
            <tr>
              <td><i>BD Rhapsody Targeted mRNA</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0700004"><code>EFO:0700004</code></a>]</td>
              <td><code>"cell"</code></td>
           </tr>
            <tr>
              <td><i>BD Rhapsody Whole Transcriptome Analysis</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0700003"><code>EFO:0700003</code></a>]</td>
              <td><code>"cell"</code></td>
           </tr>
            <tr>
              <td><i>CEL-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008679"><code>EFO:0008679</code></a>]</td>
              <td><code>"cell"</code></td>
           </tr>
            <tr>
              <td><i>CEL-seq2</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010010"><code>EFO:0010010</code></a>] and its descendants</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr>
            <tr>
              <td><i>DroNc-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008720"><code>EFO:0008720</code></a>]</td>
              <td><code>"nucleus"</code></td>
           </tr>
            <tr>
              <td><i>Drop-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008722"><code>EFO:0008722</code></a>]</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr>
            <tr>
              <td><i>GEXSCOPE technology</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0700011"><code>EFO:0700011</code></a>]</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr> 
            <tr>
              <td><i>inDrop</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008780"><code>EFO:0008780</code></a>]</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr>
            <tr>
              <td><i>MARS-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008796"><code>EFO:0008796</code></a>]</td>
              <td><code>"cell"</code></td>
           </tr>
           <tr>
             <td><i>mCT-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030060"><code>EFO:0030060</code></a>]</td>
             <td><code>"cell"</code> or <code>"nucleus"</code></td>
          </tr>
          <tr>
            <td><i>MERFISH</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008992"><code>EFO:0008992</code></a>]</td>
            <td><code>"na"</code></code></td>
          </tr>
          <tr>
           <td><i>methylation profiling by high throughput sequencing</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0002761"><code>EFO:0002761</code></a>] and its descendants</td>
           <td><code>"nucleus"</code></td>
          </tr>
          <tr>
              <td><i>microwell-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030002"><code>EFO:0030002</code></a>]</td>
              <td><code>"cell"</code></td>
           </tr>    
            <tr>
              <td><i>Patch-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008853"><code>EFO:0008853</code></a>]</td>
              <td><code>"cell"</code></td>
           </tr>
            <tr>
              <td><i>Quartz-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008877"><code>EFO:0008877</code></a>]</td>
              <td><code>"cell"</code></td>
           </tr>
          <tr>
            <td><i>ScaleBio single cell RNA sequencing</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022490"><code>EFO:0022490</code></a>]</td>
           <td><code>"cell"</code> or <code>"nucleus"</code></td>
          </tr>
            <tr>
              <td><i>sci-Plex</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030026"><code>EFO:0030026</code></a>]</td>
              <td><code>"nucleus"</code></td>
           </tr>
            <tr>
              <td><i>sci-RNA-seq3</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030028"><code>EFO:0030028</code></a>]</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr>
            <tr>
              <td><i>Seq-Well</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008919"><code>EFO:0008919</code></a>] and its descendants</td>
              <td><code>"cell"</code></td>
           </tr>
            <tr>
              <td><i>Smart-like</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010184"><code>EFO:0010184</code></a>] and its descendants</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr>
            <tr>
              <td><i>spatial transcriptomics</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008994"><code>EFO:0008994</code></a>] and its descendants</td>
              <td><code>"na"</code></td>
           </tr> 
            <tr>
              <td><i>SPLiT-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0009919"><code>EFO:0009919</code></a>] and its descendants</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr> 
            <tr>
              <td><i>STRT-seq</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008953"><code>EFO:0008953</code></a>] and its descendants</td>
              <td><code>"cell"</code></td>
           </tr>
            <tr>
              <td><i>TruDrop</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0700010"><code>EFO:0700010</code></a>]</td>
              <td><code>"cell"</code> or <code>"nucleus"</code></td>
           </tr> 
          </tbody></table>
          If the assay does not appear in this table, the most appropriate value MUST be selected.
        </td>
    </tr>
</tbody></table>

## `obsm` (Embeddings)

The value for each `str` key MUST be a  `numpy.ndarray` of shape `(n_obs, m)`, where `n_obs` is the number of rows in `X` and `m >= 1`.

### X_{suffix}

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>X_{suffix}</code> with the following requirements:<br/><br/>
      <ul>
        <li>{suffix} MUST be at least one character in length.</li>
        <li>The first character of {suffix} MUST be a letter of the alphabet and the remaining characters MUST be alphanumeric characters, <code>'_'</code>, <code>'-'</code>, or <code>'.'</code> (This is equivalent to the regular expression pattern <code>"^[a-zA-Z][a-zA-Z0-9_.-]*$"</code>.)</li>
        <li>{suffix} MUST NOT be <code>"spatial"</code>.
      </ul>
      <b>Note:</b> See also <code><a href="https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#default_embedding">default_embedding</a></code> in <code><a href="https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md#uns">uns</a></code>.</td>
    </tr>
    <tr>
      <th>Requirement</th>
         <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>numpy.ndarray</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td> 
          If present, this must endorse the following requirements:
          <ul>
            <li>MUST have the same number of rows as <code>X</code> and MUST include at least two columns</li>
            <li>MUST be a <a href="https://numpy.org/doc/stable/reference/generated/numpy.dtype.kind.html"><code>numpy.dtype.kind</code></a> of <code>"f"</code>, <code>"i"</code>, or "<code>u"</code></li>
            <li>MUST NOT contain any <a href="https://numpy.org/devdocs/reference/constants.html#numpy.inf">positive infinity (<code>numpy.inf</code>)</a> or <a href="https://numpy.org/devdocs/reference/constants.html#numpy.NINF">negative infinity (<code>numpy.NINF</code>)</a> values</li>
            <li>MUST NOT contain all <a href="https://numpy.org/devdocs/reference/constants.html#numpy.nan">Not a Number (<code>numpy.nan</code>)</a> values</li>
          </ul>
        </td>
    </tr>
</tbody></table>

## `obsp`

If present, the size of the ndarray stored for a key in `obsp` MUST NOT be zero.

## `var` and `raw.var` (Gene Metadata)

`var` and `raw.var` are both of type [`pandas.DataFrame`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html).

### index

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>index</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          The index of the <code>pandas.DataFrame</code> MUST contain unique identifiers for features (e.g. gene names). If present, the index of <code>raw.var</code> MUST be identical to the index of <code>var</code>.<br/><br/>
          Here, we accept both genes and ERCC spike-ins. In short, ENSEMBL identifiers are required for genes and <a href="https://www.ncbi.nlm.nih.gov/pmc/articles/PMC4978944/">External RNA Controls Consortium (ERCC)</a> identifiers for <a href="https://www.thermofisher.com/document-connect/document-connect.html?url=https%3A%2F%2Fassets.thermofisher.com%2FTFS-Assets%2FLSG%2Fmanuals%2Fcms_086340.pdf&title=VXNlciBHdWlkZTogRVJDQyBSTkEgU3Bpa2UtSW4gQ29udHJvbCBNaXhlcyAoRW5nbGlzaCAp">RNA Spike-In Control Mixes</a> to ensure that all datasets measure the same features and can therefore be integrated.<br/><br/>
          If the feature is a gene then the value MUST be the <code>gene_id</code> attribute from the corresponding <code>organism_ontology_term_id</code>. scFAIR allows gene annotations from any species, and any release present in the Ensembl database.<br/><br/>
          The Ensembl release and assembly used for gene annotation should also be specified in <a href="#uns-dataset-metadata"><code>uns</code></a> entries <a href="#ensembl_release"><code>ensembl_release</code></a>, and <a href="#ensembl_assembly"><code>ensembl_assembly</code></a>.<br/><br/>
          <b>Note:</b> Version numbers MUST be removed from the <code>gene_id</code> if it is prefixed with <code>"ENS"</code> for <i>Ensembl stable identifier</i>. See <a href="https://ensembl.org/Help/Faq?id=488">I have an Ensembl ID, what can I tell about it from the ID?</a> For example, if the <code>gene_id</code> is <code>“ENSG00000186092.7”</code>, then the value MUST be <code>“ENSG00000186092”</code>.<br/><br/>
          If the feature is a <a href="https://www.thermofisher.com/document-connect/document-connect.html?url=https%3A%2F%2Fassets.thermofisher.com%2FTFS-Assets%2FLSG%2Fmanuals%2Fcms_086340.pdf&title=VXNlciBHdWlkZTogRVJDQyBSTkEgU3Bpa2UtSW4gQ29udHJvbCBNaXhlcyAoRW5nbGlzaCAp">RNA Spike-In Control Mix</a> then the value MUST be an ERCC Spike-In identifier (e.g. <code>"ERCC-0003"</code>) from <a href="https://assets.thermofisher.com/TFS-Assets/LSG/manuals/cms_095047.txt">cms_095047.txt</a>.<br/>
        Of note, in the h5ad file, this is stored as an attribute of <code>var</code> named <code>_index</code>, pointing to an existing <code>var</code> metadata.
        </td>
    </tr>
</tbody></table>

### feature_is_filtered

This column is REQUIRED only in the `var` dataframe. This column MUST NOT be present in `raw.var`:

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>feature_is_filtered</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>bool</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          When a raw matrix is not present, the value for all features MUST be <code>False</code>.<br/><br/>
          When both a raw and normalized matrix are present, this MUST be <code>True</code> if the feature was filtered out in the normalized matrix (<code>X</code>) but is present in the raw matrix (<code>raw.X</code>). The value for all cells of the given feature in the normalized matrix MUST be <code>0</code>. If a feature contains all zeroes in the normalized matrix, then either the corresponding feature in the raw matrix MUST be all zeroes or the value MUST be <code>True</code>.
        </td>
    </tr>
</tbody></table>

### *feature_biotype*

This column is REQUIRED in the `var` dataframe and if present, also in the `raw.var` dataframe.

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>feature_biotype</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>This MUST be <code>"gene"</code> or <code>"spike-in"</code>.</td>
    </tr>
</tbody></table>

### *feature_length*

This column is REQUIRED in the `var` dataframe and if present, in the `raw.var` dataframe.

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>feature_length</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>uint</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
        Number of base-pairs (bps) usually used for some normalizations such as RPKM. The value should be calculated as the median of the lengths of isoforms, reusing the median calculation from <a href="https://doi.org/10.1093/bioinformatics/btac561">GTFtools: a software package for analyzing various features of gene models.</a>
      </td>
    </tr>
</tbody></table>

### *feature_name*

This column is REQUIRED in the `var` dataframe and if present, in the `raw.var` dataframe.

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>feature_name</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          If the <code>feature_biotype</code> is <code>"spike-in"</code> then this MUST be the ERCC Spike-In identifier appended with <code>" (spike-in control)"</code>.<br/><br/>
          If the <code>feature_biotype</code> is <code>"gene"</code> and a <code>gene_name</code> attribute is assigned to the <code>var.index</code> feature identifier in its corresponding gene reference, this MUST be the value of the <code>gene_name</code>.<br/><br/>
          If a <code>gene_name</code> attribute is not assigned, then this MUST default to the <code>var.index</code> feature identifier. 
        </td>
    </tr>
</tbody></table>

### *feature_reference*

This column is REQUIRED in the `var` dataframe and if present, in the `raw.var` dataframe.

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>feature_reference</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be the reference organism for the corresponding feature. For example, if the feature comes from the <i>Caenorhabditis elegans</i> assembly then the value MUST be <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A6239"><code>"NCBITaxon:6293"</code></a>.<br /><br />
          Special case when the feature comes from <i>SARS-CoV-2</i>, then the value MUST be <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A2697049"><code>"NCBITaxon:2697049"</code></a><br /><br />
          Similarly, if the feature is an <i>ERCC Spike-Ins</i>, then the value MUST be <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A32630"><code>"NCBITaxon:32630"</code></a>
        </td>
    </tr>
</tbody></table>

### *feature_type*

This column is REQUIRED in the `var` dataframe and if present, in the `raw.var` dataframe.

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>feature_type</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          If the <code>feature_biotype</code> is <code>"gene"</code> then this MUST be the gene type assigned to the feature identifier in <code>var.index</code>.<br/><br/>
          If the <code>feature_biotype</code> is <code>"spike-in"</code> then this MUST be <code>"synthetic"</code>.<br/><br/>
          See <a href="https://useast.ensembl.org/info/genome/genebuild/biotypes.html ">Ensembl</a> references.
        </td>
    </tr>
</tbody></table>

### feature_chromosome

This column is REQUIRED in the `var` dataframe and if present, in the `raw.var` dataframe.

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>feature_chromosome</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          If the <code>feature_biotype</code> is <code>"gene"</code> then this MUST be the chromosome location corresponding to the feature identifier in <code>var.index</code>.<br/><br/>
          If the <code>feature_biotype</code> is <code>"spike-in"</code> then this MUST be <code>"na"</code>.<br/><br/>
          Each chromosome name MUST be a <a href="https://www.ncbi.nlm.nih.gov/genbank/fastaformat/">sequence identifier</a> from the FASTA reference for the <code>organism_ontology_term_id</code>.<br/><br/>
          It MUST be encoded based on ENSEMBL identifiers, and eventually needs to be updated to:
          <ul>
            <li>Remove their <code>"chr"</code> prefix, if present</li>
            <li>Rename the mitochondrial designator from <code>"M"</code> (**or any other identifier**) to <code>"MT"</code></li>
        </td>
    </tr>
</tbody></table>

## `varm`

If present, the size of the ndarray stored for a key in `varm` MUST NOT be zero.

## `varp`

If present, the size of the ndarray stored for a key in `varp` MUST NOT be zero.

## `uns` (Dataset Metadata)

`uns` is a ordered dictionary with a `str` key. The size of the data stored as a value for a key in `uns` MUST NOT be zero.

### ensembl_release

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>ensembl_release</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>int</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          The key MUST be the <b>Ensembl</b> release number of the assembly used for gene annotation, e.g. <code>115</code> for <a href="https://ftp.ensembl.org/pub/release-115/">Ensembl r.115</a>.
        </td>
    </tr>
</tbody></table>

### ensembl_assembly

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>ensembl_assembly</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          The key MUST be the <b>Ensembl</b> assembly name of the assembly used for gene annotation, e.g. <code>"GRCh38.p14"</code> for Homo Sapiens release 115. You can relate for e.g. to this <a href="https://www.ensembl.org/info/website/archives/assembly.html">correspondance table</a>.
        </td>
    </tr>
</tbody></table>

### organism_ontology_term_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>organism_ontology_term_id</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          Should be the `"NCBITaxon"` ontology term corresponding to the main organism of the study, e.g. `"NCBITaxon:7227"` for *Drosophila Melanogaster*, `"NCBITaxon:9606"` for *Homo Sapiens*, or `"NCBITaxon:10090"` for <i>Mus musculus<br/>and its descendants</i>.
        </td>
    </tr>
</tbody></table>

### *organism*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>organism</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td>categorical with <code>str</code> categories.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This MUST be the human-readable name assigned to the value of <code>organism_ontology_term_id</code>.
        </td>
    </tr>
</tbody></table>

### title

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>title</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        This text describes and differentiates the dataset from other datasets.<br/><br/>
        It is STRONGLY RECOMMENDED to make sure that each dataset <code>title</code> is unique and does not depend on other metadata such as a different <code>assay</code> to disambiguate it from other datasets.
      </td>
    </tr>
</tbody></table>

### *schema_reference*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>schema_reference</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>REQUIRED</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        This MUST be <code>"https://github.com/scFAIR/scFAIR/edit/main/schema/7.1.0/schema.md"</code>.
      </td>
    </tr>
</tbody></table>

### *schema_version*

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
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
      <td>
        This MUST be <code>"7.1.0+scfair1.0"</code>.
      </td>
    </tr>
</tbody></table>

---

​The following keys and values in `uns` are OPTIONAL. If the key is present, then its value MUST NOT be empty.
​
### analysis_pipeline

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>analysis_pipeline</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          A JSON entry describing the analysis pipeline used to preprocess (sequencing, alignment, counting) and perform the downstream analysis (filtering, normalization, dimension reduction, integration, clustering, annotation of the clusters) of the dataset.<br/><br/>
          The JSON file should follow the standard defined in <a href="https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md"><code>schema_analysis_json.md</code></a>.<br/><br/>
          The goal of this field is to better understand how the results were obtained and also to enhance reproducibility.
        </td>
    </tr>
</tbody></table>

### batch_condition

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>batch_condition</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>list[str]</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>str</code> values MUST refer to cell metadata keys in <code>obs</code>. Together, these keys define the <i>batches</i> that a normalization or integration algorithm should be aware of.<br/><br/>
          For example if <code>"patient"</code> and <code>"seqBatch"</code> are keys of vectors of cell metadata, either <code>["patient"]</code>, <code>["seqBatch"]</code>, or <code>["patient", "seqBatch"]</code> are valid values.
        </td>
    </tr>
</tbody></table>

### *citation*

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>citation</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          Publication DOI url associated with the dataset.
        </td>
    </tr>
</tbody></table>

### {column}_colors

<table><tbody>
  <tr>
    <th>Key</th>
      <td>
        <code>{column}_colors</code> where {column} MUST be the name of a <code>category</code> data type column in <code>obs</code> that<br/> is annotated by the data submitter or curator.<br/><br/>
        {column} can be for example <code>assay</code>, <code>assay_ontology_term_id</code>, <code>cell_type</code>, or any other categorical <code>obs</code>.
      </td>
  </tr>
  <tr>
    <th>Requirement</th>
    <td>OPTIONAL</td>
  </tr>
  <tr>
    <th>Type</th>
    <td><code>numpy.ndarray</code>.</td>
  </tr>
  <tr>
    <th>Value</th>
      <td>
        This MUST be a 1-D array of shape <code>(, c)</code>, where <code>c</code> is greater than or equal to the<br/> number of categories in the {column} as calculated by:<br/><br/>
           <samp>anndata.obs.{column}.cat.categories.size</samp><br/><br/>
        The color code at the Nth position in the <code>ndarray</code> corresponds to the Nth category of <samp>anndata.obs.{column}.cat.categories</samp>.<br/><br/>
        For example, if <code>cell_type_ontology_term_id</code> includes two categories:<br/><br/>
        <samp>anndata.obs.cell_type_ontology_term_id.cat.categories.values</samp><br/><br/>
        <samp>array(['CL:0000057', 'CL:0000115'], dtype='object')</samp><br/><br/>
        then <code>cell-type_ontology_term_id_colors</code> MUST contain two or more colors such as:<br/><br/>
        <samp>['aqua' 'blueviolet']</samp><br/><br/>where <code>'aqua'</code> is the color assigned to <code>'CL:0000057'</code> and <code>'blueviolet'</code> is the color assigned to<br/> <code>'CL:0000115'</code>.<br/><br/>
        All elements in the <code>ndarray</code> MUST use the same color model, limited to:
          <table>
          <thead>
            <tr>
              <th>Color Model</th>
              <th>Element Format</th>
            </tr>
          </thead><tbody>
            <tr>
              <td>
                <a href="https://www.w3.org/TR/css-color-4/#named-colors"><i>Named Colors </i></a>
              </td>
              <td>
                <code>str</code>. MUST be a case-insensitive CSS4 color name with no spaces such as<br/> <code>"aliceblue"</code>
              </td>
            </tr>
            <tr>
             <td>
              <a href="https://www.w3.org/TR/css-color-4/#hex-notation"><i>Hex Triplet</i></a>
             </td>
             <td>
               <code>str</code>. MUST start with <code>"#"</code> immediately followed by six case-insensitive hexadecimal<br/> characters as in <code>"#08c0ff"</code>
             </td>
            </tr>
          </tbody></table>
        </td>
    </tr>
</tbody></table>

### default_embedding

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>default_embedding</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          The value MUST match a key to an embedding in <code>obsm</code> for the embedding to be displayed by default in any portal.
        </td>
    </tr>
</tbody></table>

### X_approximate_distribution

<table><tbody>
    <tr>
      <th>Key</th>
      <td><code>X_approximate_distribution</code></td>
    </tr>
    <tr>
      <th>Requirement</th>
      <td>OPTIONAL</td>
    </tr>
    <tr>
      <th>Type</th>
      <td><code>str</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          This field enables the data submitter or curator to specify the data distribution explicitly and not relying on portal automatic detection of the type of data (raw counts, or normalized).<br/><br/>
          The value MUST be <code>"count"</code> (for data whose distributions are best approximated by counting distributions like Poisson, Binomial, or Negative Binomial) or <code>"normal"</code> (for data whose distributions are best approximated by the Gaussian distribution.)
        </td>
    </tr>
</tbody></table>

---

## Appendix A. Changelog

### Schema v7.1.0
This is the first fork of CELLxGENE schema. So, here are recorded the differences with CZI CELLxGENE schema v7.1.0

* **Required ontologies**
  * Moved the ontology table from [General Requirements](#general-requirements) as [Appendix B. Relevant ontologies](#appendix-b-relevant-ontologies). Since we don't enforce a schema-specific version anymore
  * Recommended using the [Uberon collected metazoan ontology] or [Uberon composite metazoan ontology] version of [Uberon multi-species anatomy ontology], instead of taxon-specific ontologies, for anatomy, cell types, developemental and life stages
  * Added plants and other invertebrate ontologies
* Moved the **Important note on types** section to the [General Requirements](#general-requirements) section. Expanding on the difference between reported Python types and HDF5 inner typing
* **Required Gene Annotations**
  * This section was removed, but its content was moved to the [`index`](#index) subsection of [`var` and `raw.var`](#var-and-rawvar-gene-metadata) section where it immediately applies
  * CZI CELLxGENE schema only handles certain Taxons, and specify a fixed Ensembl release for each species that they "attach" to the schema version as fixed. scFAIR allows gene annotations from any Ensembl species, and any release present in the [Ensembl database](https://www.ensembl.org/index.html). You can check available species from [species.json](https://ftp.ebi.ac.uk/pub/ensemblorganisms/species.json)
  * Removed GENCODE from authorized gene names. Only Ensembl is allowed
* [`X` (Matrix layers)](#x-matrix-layers)
  * Moved the scATAC-seq part (and requirement table) to scATAC-specific schema ['schema_atac.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md)
* [`obs`](#obs-cell-metadata) (Cell metadata)
  * Reordered the fields to organize them better semantically-speaking
  * Removed `observation_joinid` as it is specific for CELLxGENE
  * Moved `array_row`, `array_col`, and `in_tissue` to spatial-specific schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#array_row)
  * Moved `genetic_perturbation_id`, `genetic_perturbation_strategy` to perturbation-specific schema ['schema_perturb.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md#genetic_perturbation_id)
  * Added `strain_or_genetic_background_term_id` and `strain_or_genetic_background` to describe strain, genotype,etc..
  * Modified `tissue_ontology_term_id` and `tissue` to allow for multiple terms with a separator ` || `
  * Modified `cell_type_ontology_term_id` and `cell_type` to allow for multiple terms with a separator ` || `
  * Modified `development_stage_ontology_term_id` and `development_stage` to allow for multiple terms with a separator ` || `
  * Made `experimental_condition_ontology_term_id` OPTIONAL, since if all values are `na` then the field MUST NOT be present.
  * Removed some restriction in `tissue_ontology_term_id`, `cell_type_ontology_term_id`, `development_stage_ontology_term_id`, `experimental_condition_ontology_term_id`, and `disease_ontology_term_id` regarding certain basic terms that were previously not allowed (like 'cell' or 'organism'). It is clear enough that the more precise term should be used, and became impractical to update with new ontologies/species.
  * Updated `sex_ontology_term_id`, `tissue_ontology_term_id`, `cell_type_ontology_term_id`, `development_stage_ontology_term_id`, `experimental_condition_ontology_term_id`, `perturbation_types`, and `disease_ontology_term_id` to handle plants and new species such as invertebrates which were not handled before
* [`obsm`](#obsm-embeddings) (Embeddings)
  * Modified [`X_{suffix}`](#x_suffix) section and related comments throughout the document to make the embedding optional for visualization (it was CELLxGENE-specific for its visualization portal)
  * Moved `spatial` to the spatial-specific schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md#spatial).
* [`var` and `raw.var`](#var-and-rawvar-gene-metadata) (Gene metadata)
  * Modified [`index`](#index-1) subsection, as detailed above
  * We think [`feature_type`](#feature_type) and [`feature_biotype`](#feature_biotype) are probably intertwined in CxG definition but for now we keep them as is for compatibility purpose
  * Added [`feature_chromosome`](#feature_chromosome) to provide chromosome information for each feature. Useful for MT QC plot
  * Modified [`feature_reference`](#feature_reference) to allow other species. Only kept some examples.
* [`uns`](#uns-dataset-metadata) (Dataset Metadata)
  * Moved this entire section after [`var` and `raw.var`](#var-and-rawvar-gene-metadata), I think it was misplaced before.
  * Added [`ensembl_release`](#ensembl_release) to inform on the Ensembl release used for gene annotation, since scFAIR allows all available species in Ensembl
  * Added, then removed [`ensembl_database`](#ensembl_database) to inform on the Ensembl database used for gene annotation, since scFAIR allows all available species in Ensembl. Since the new Ensembl database does not separate the species as before.
  * Added [`ensembl_assembly`](#ensembl_assembly) to inform on the Ensembl assembly used for gene annotation, since scFAIR allows all available species in Ensembl
  * Added [`analysis_pipeline`](#analysis_pipeline) entry to store the analysis and annotation pipeline, as a JSON. The JSON schema itself is described in ['schema_analysis_json.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md)
  * Removed `is_pre_analysis` as it is specific for CELLxGENE collection handling
  * Moved <code>genetic_perturbations</code>, <code>genetic_perturbations[<i>id</i>]</code>, <code>genetic_perturbations[<i>id</i>]['role']</code>, <code>genetic_perturbations[<i>id</i>]['protospacer_sequence']</code>, <code>genetic_perturbations[<i>id</i>]['protospacer_adjacent_motif']</code>, <code>genetic_perturbations[<i>id</i>]['derived_genomic_regions']</code>, <code>genetic_perturbations[<i>id</i>]['derived_features']</code>, and <code>genetic_perturbations[<i>id</i>]['derived_features'][<i>feature_id</i>]</code> to perturb-specific schema ['schema_perturb.md'](https://github.com/scFAIR/scFAIR/edit/main/schema/7.1.0/schema_perturb.md#genetic_perturbations)
  * Moved <code>spatial</code>, <code>spatial[<i>library_id</i>]</code>, <code>spatial[<i>library_id</i>]['is_single']</code>, <code>spatial[<i>library_id</i>]['images']</code>, <code>spatial[<i>library_id</i>]['images']['fullres']</code>, <code>spatial[<i>library_id</i>]['images']['hires']</code>, <code>spatial[<i>library_id</i>]['scalefactors']</code>, <code>spatial[<i>library_id</i>]['scalefactors']['spot_diameter_fullres']</code>, and <code>spatial[<i>library_id</i>]['scalefactors']['tissue_hires_scalef']</code> to spatial-specific schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/edit/main/schema/7.1.0/schema_spatial.md#spatial)
  * Modified `{column}_colors` so that anything can be edited by the data submitter or curator, and everything is thus optional.
  * Modified `organism_ontology_term_id` to allow any species
  * Modified `citation` so that it contains only the DOI. Made this field Optional
* Move scTAC-seq assets to atac-specific schema ['schema_atac.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md#scatac-seq-assets)
* Created the analysis-specific schema ['schema_analysis_json.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md) in JSON, for storing the analysis and annotation pipeline. The JSON entry itself is stored in uns/[`analysis_pipeline`](#analysis_pipeline)
* Changed the format of the tables, specifically Annotator -> Requirement, and created a specific field for the type, instead of merging with Value

## Appendix B. Relevant ontologies

| Ontology | Prefix | Release | Download |
|:--|:--|:--|:--|
| [C. elegans Development Ontology] | WBls: |  [2025-08-12 WS298](https://github.com/obophenotype/c-elegans-development-ontology/releases/tag/v2025-08-12) | [wbls.owl](https://github.com/obophenotype/c-elegans-development-ontology/blob/v2025-08-12/wbls.owl) |
| [C. elegans Gross Anatomy Ontology] | WBbt: | [2025-08-18 WS298](https://github.com/obophenotype/c-elegans-gross-anatomy-ontology/releases/tag/v2025-08-18) | [wbbt.owl](https://github.com/obophenotype/c-elegans-gross-anatomy-ontology/blob/v2025-08-18/wbbt.owl) |
| [Cell Ontology] | CL: |  [2025-07-30](https://github.com/obophenotype/cell-ontology/releases/tag/v2025-07-30) | [cl.owl](https://github.com/obophenotype/cell-ontology/releases/download/v2025-07-30/cl.owl)|
| [Cellosaurus] | CVCL_ | 53.0 | [cellosaurus.obo ](https://ftp.expasy.org/databases/cellosaurus/cellosaurus.obo)_(Cellosaurus may replace this download with a newer release. Previous releases are <b>unavailable</b>. )_  |
| [Cephalopod Ontology] | CEPH: | [2016-01-12](https://github.com/obophenotype/cephalopod-ontology/releases/tag/v2016-01-12) _(OBO Foundry status: **inactive**)_ | [ceph.owl](http://purl.obolibrary.org/obo/ceph.owl) |
| [Chemical Entities of Biological Interest] | CHEBI: | [2026-01-06](https://ftp.ebi.ac.uk/pub/databases/chebi/ontology/)<br/>248 | [chebi-lite.owl](https://ftp.ebi.ac.uk/pub/databases/chebi/ontology/chebi_lite.owl.gz) _(CHEBI may replace this download with a newer release. Previous releases are [available](https://ftp.ebi.ac.uk/pub/databases/chebi/archive/). )_ |
| [Ctenophore Ontology] | CTENO: | [2016-10-19](https://github.com/obophenotype/ctenophore-ontology/releases/tag/v2016-10-19) | [cteno.owl](http://purl.obolibrary.org/obo/cteno.owl) |
| [Drosophila Anatomy Ontology] | FBbt: | [2025-08-07](https://github.com/FlyBase/drosophila-anatomy-developmental-ontology/releases/tag/v2025-08-07)| [fbbt.owl](https://github.com/FlyBase/drosophila-anatomy-developmental-ontology/releases/download/v2025-08-07/fbbt.owl) |
| [Drosophila Development Ontology] | FBdv: | [2025-05-29](https://github.com/FlyBase/drosophila-developmental-ontology/releases/tag/v2025-05-29) | [fbdv.owl](https://github.com/FlyBase/drosophila-developmental-ontology/releases/download/v2025-05-29/fbdv.owl) |
| [Experimental Factor Ontology] | EFO: | [2025-09-15 EFO 3.82.0](https://github.com/EBISPOT/efo/releases/tag/v3.82.0) | [efo.owl](https://github.com/EBISPOT/efo/releases/download/v3.82.0/efo.owl) |
| [Human Ancestry Ontology] | AfPO:<br/>HANCESTRO: | [2025-04-01](https://github.com/EBISPOT/hancestro/releases/tag/v2025-04-01) | [hancestro.owl](https://github.com/EBISPOT/hancestro/blob/v2025-04-01/hancestro.owl) |
| [Human Developmental Stages] |  HsapDv: | [2025-01-23](https://github.com/obophenotype/developmental-stage-ontologies/releases/tag/v2025-01-23) | [hsapdv.owl](https://github.com/obophenotype/developmental-stage-ontologies/releases/download/v2025-01-23/hsapdv.owl) |
| [Hymenoptera Anatomy Ontology] | HAO: | [2023-06-01](https://github.com/hymao/hao/releases/tag/v2023-06-01) | [hao.owl](http://purl.obolibrary.org/obo/hao.owl) |
| [Mondo Disease Ontology] | MONDO: | [2025-09-02](https://github.com/monarch-initiative/mondo/releases/tag/v2025-09-02) | [mondo.owl](https://github.com/monarch-initiative/mondo/releases/download/v2025-09-02/mondo.owl) |
| [Mouse Developmental Stages]| MmusDv: | [2025-01-23](https://github.com/obophenotype/developmental-stage-ontologies/releases/tag/v2025-01-23) | [mmusdv.owl](https://github.com/obophenotype/developmental-stage-ontologies/releases/download/v2025-01-23/mmusdv.owl) |
| [NCBI organismal classification] |  NCBITaxon: | [2025-09-11](https://github.com/obophenotype/ncbitaxon/releases/tag/v2025-09-11) | [ncbitaxon.owl](https://github.com/obophenotype/ncbitaxon/releases/download/v2025-09-11/ncbitaxon.owl.gz) |
| [Phenotype And Trait Ontology] | PATO: | [2025-05-14](https://github.com/pato-ontology/pato/releases/tag/v2025-05-14) | [pato.owl](https://github.com/pato-ontology/pato/blob/v2025-05-14/pato.owl)  |
| [Plant Experimental Conditions Ontology] | PECO: | [2025-12-02](https://github.com/Planteome/plant-experimental-conditions-ontology/releases/tag/v2025-12-02) | [peco.owl](https://github.com/Planteome/plant-experimental-conditions-ontology/blob/v2025-12-02/peco.owl) |
| [Plant Ontology] | PO: | [2026-01-09](https://github.com/Planteome/plant-ontology/releases/tag/v2026-01-09) | [po.owl](https://github.com/Planteome/plant-ontology/blob/v2026-01-09/po.owl) |
| [Plant Stress Ontology] | PSO: | [2026-01-08](https://github.com/Planteome/plant-stress-ontology/releases/tag/v2026-01-08) | [pso.owl](https://github.com/Planteome/plant-stress-ontology/blob/v2026-01-08/pso.owl) |
| [Plant Trait Ontology] | TO: | [2026-01-14](https://github.com/Planteome/plant-trait-ontology/releases/tag/v2026-01-14) | [to.owl](https://github.com/Planteome/plant-trait-ontology/blob/v2026-01-14/to.owl) |
| [Porifera Ontology] | PORO: | [2016-10-06](https://github.com/obophenotype/porifera-ontology/releases/tag/v2016-10-06) | [poro.owl](http://purl.obolibrary.org/obo/poro.owl) |
| [Uberon multi-species anatomy ontology] |  UBERON: | [2025-08-15](https://github.com/obophenotype/uberon/releases/tag/v2025-08-15) | [uberon.owl](https://github.com/obophenotype/uberon/releases/download/v2025-08-15/uberon.owl) |
| [Uberon composite metazoan ontology] | UBERON:, CL:, and taxon-specific prefixes from imported ontologies | [2025-08-15](https://github.com/obophenotype/uberon/releases/tag/v2025-08-15) | [composite-metazoan.owl](https://github.com/obophenotype/uberon/releases/download/v2025-08-15/composite-metazoan.owl) |
| [Uberon collected metazoan ontology] | UBERON:, CL:, and taxon-specific prefixes from imported ontologies | [2025-08-15](https://github.com/obophenotype/uberon/releases/tag/v2025-08-15) | [collected-metazoan.owl](https://github.com/obophenotype/uberon/releases/download/v2025-08-15/collected-metazoan.owl) |
| [UniProt Knowledgebase] | uniprot: | [08-Oct-2025](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/)<br/>2025_04 | [uniprot_sprot.xml](https://ftp.uniprot.org/pub/databases/uniprot/current_release/knowledgebase/complete/uniprot_sprot.xml.gz) _(UniProt may replace this download with a newer release. Previous releases are [available](https://ftp.uniprot.org/pub/databases/uniprot/previous_major_releases/).)_ | 
| [Zebrafish Anatomy Ontology] | ZFA:<br/>ZFS: | [2025-09-05](https://github.com/ZFIN/zebrafish-anatomical-ontology/releases/tag/v2025-09-05) | [zfa.owl](https://github.com/ZFIN/zebrafish-anatomical-ontology/blob/v2025-09-05/zfa.owl) |

[C. elegans Development Ontology]: https://obofoundry.org/ontology/wbls.html

[C. elegans Gross Anatomy Ontology]: https://obofoundry.org/ontology/wbbt.html

[Cell Ontology]: https://obofoundry.org/ontology/cl.html

[Cellosaurus]: https://www.cellosaurus.org/description.html

[Chemical Entities of Biological Interest]: https://www.ebi.ac.uk/chebi/

[Drosophila Anatomy Ontology]: https://obofoundry.org/ontology/fbbt.html

[Drosophila Development Ontology]: https://obofoundry.org/ontology/fbdv.html

[Experimental Factor Ontology]: https://www.ebi.ac.uk/efo

[Human Ancestry Ontology]: https://www.obofoundry.org/ontology/hancestro.html

[Human Developmental Stages]: https://obofoundry.org/ontology/hsapdv.html

[Mondo Disease Ontology]: https://obofoundry.org/ontology/mondo.html

[Mouse Developmental Stages]: https://obofoundry.org/ontology/mmusdv.html

[NCBI organismal classification]: https://obofoundry.org/ontology/ncbitaxon.html

[Phenotype And Trait Ontology]: https://www.obofoundry.org/ontology/pato.html

[Uberon multi-species anatomy ontology]: https://www.obofoundry.org/ontology/uberon.html

[Uberon collected metazoan ontology]: https://obophenotype.github.io/uberon/combined_multispecies/#collected-ontologies

[Uberon composite metazoan ontology]: https://obophenotype.github.io/uberon/combined_multispecies/#composite-ontologies

[UniProt Knowledgebase]: https://uniprot.org

[Zebrafish Anatomy Ontology]: https://obofoundry.org/ontology/zfa.html

[Cephalopod Ontology]: https://obofoundry.org/ontology/ceph.html

[Ctenophore Ontology]: https://obofoundry.org/ontology/cteno.html

[Hymenoptera Anatomy Ontology]: https://obofoundry.org/ontology/hao.html

[Plant Experimental Conditions Ontology]: https://obofoundry.org/ontology/peco.html

[Plant Ontology]: https://obofoundry.org/ontology/po.html

[Plant Stress Ontology]: https://obofoundry.org/ontology/pso.html

[Plant Trait Ontology]: https://obofoundry.org/ontology/to.html

[Porifera Ontology]: https://obofoundry.org/ontology/poro.html

### B.1 Taxon-neutral core and clade-specific ontologies

The ontologies above fall into two layers.

A **taxon-neutral core** applies to every organism for which it provides an adequate description: [Uberon](https://obophenotype.github.io/uberon/) for anatomical entities and life cycle stages, the [Cell Ontology](https://obophenotype.github.io/cell-ontology/) for cell types, MONDO and PATO for disease and normal/healthy, and EFO, CHEBI and UniProt for experimental conditions. A taxon-neutral term SHOULD be preferred wherever it is adequate, because it maximises cross-species comparability and searchability.

A set of **clade-specific ontologies** MUST be used where one provides a more precise term than the core for the taxon under study, or where the core does not cover the clade at all. The latter is the case for <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A33090"><code>NCBITaxon:33090</code></a> for <i>Viridiplantae</i>, which is outside the scope of both Uberon and CL.

Most metazoan clade-specific ontologies are merged into the Uberon multi-species products, where `collected-metazoan` = `collected-vertebrate` + `collected-drosophila` + `collected-worm` + `CEPH` + `CTENO` + `PORO`. Two caveats follow from the Uberon build: `collected-human` = `EHDAA2` + `AEO` and `collected-zebrafish` = `ZFA` only, so `HsapDv:` and `ZFS:` are **not** in the metazoan products and ship in `collected-lifestages` instead. Ontologies marked **standalone** below are in no Uberon product at all: they carry no Uberon or CL cross-reference, and their terms MUST be validated against the root in the *Validation root* column.

Terms from a clade-specific ontology MUST NOT be used for an organism outside the clade that ontology covers. For an organism in a clade not listed below, a taxon-neutral term MUST be used, and an addition to this table MAY be requested from the schema maintainers.

| Scope | Clade / organism | `organism_ontology_term_id` | Prefix | Applies to | Validation root | In Uberon metazoan product |
|:--|:--|:--|:--|:--|:--|:--|
| Taxon-neutral core | *any* | — | `UBERON:` | `tissue_ontology_term_id`, `development_stage_ontology_term_id` | `UBERON:0001062`, `UBERON:0000105` | core |
| Taxon-neutral core | *any* | — | `CL:` | `cell_type_ontology_term_id` | `CL:0000000` | core |
| Taxon-neutral core | *any* | — | `MONDO:`, `PATO:` | `disease_ontology_term_id`, `sex_ontology_term_id` | `MONDO:0000001` | n/a |
| Taxon-neutral core | *any* | — | `EFO:`, `CHEBI:`, `uniprot:` | `experimental_condition_ontology_term_id` | see field | n/a |
| Metazoa | *Homo sapiens* | `NCBITaxon:9606` | `EHDAA2:` | tissue | `UBERON:0001062` | yes |
| Metazoa | *Homo sapiens* | `NCBITaxon:9606` | `HsapDv:` | development stage | `UBERON:0000105` | via `collected-lifestages` |
| Metazoa | *Mus musculus* | `NCBITaxon:10090` | `EMAPA:`, `MA:`, `MmusDv:` | tissue, development stage | `UBERON:0001062`, `UBERON:0000105` | yes |
| Metazoa | *Danio rerio* | `NCBITaxon:7955` | `ZFA:` | tissue, cell type | `UBERON:0001062`, `CL:0000000` | yes |
| Metazoa | *Danio rerio* | `NCBITaxon:7955` | `ZFS:` | development stage | `UBERON:0000105` | via `collected-lifestages` |
| Metazoa | *Xenopus* | `NCBITaxon:8353` | `XAO:` | tissue, cell type, development stage | `UBERON:0001062`, `CL:0000000` | yes |
| Metazoa | *Drosophila melanogaster* | `NCBITaxon:7227` | `FBbt:`, `FBdv:` | tissue, cell type, development stage | `UBERON:0001062`, `CL:0000000`, `UBERON:0000105` | yes |
| Metazoa | *Caenorhabditis elegans* | `NCBITaxon:6239` | `WBbt:`, `WBls:` | tissue, cell type, life stage | `UBERON:0001062`, `CL:0000000`, `UBERON:0000105` | yes |
| Metazoa | *Cephalopoda* | `NCBITaxon:6605` | `CEPH:` | tissue, cell type | `UBERON:0001062`, `CL:0000000` | yes |
| Metazoa | *Ctenophora* (comb jellies) | `NCBITaxon:10197` | `CTENO:` | tissue, cell type, development stage | `UBERON:0001062`, `CL:0000000`, `UBERON:0000105` | yes |
| Metazoa | *Porifera* (sponges) | `NCBITaxon:6040` | `PORO:` | tissue, cell type | `UBERON:0001062`, `CL:0000000` | yes |
| Metazoa | *Hymenoptera* | `NCBITaxon:7399` | `HAO:` | tissue | `HAO:0000000` | **standalone** (rooted in CARO) |
| Viridiplantae | *Viridiplantae* | `NCBITaxon:33090` | `PO:` | tissue, cell type, development stage | `PO:0025131`, `PO:0009002`, `PO:0009012` | **standalone** |
| Viridiplantae | *Viridiplantae* | `NCBITaxon:33090` | `TO:` | plant trait (author-defined field) | `TO:0000387` | **standalone** |
| Viridiplantae | *Viridiplantae* | `NCBITaxon:33090` | `PECO:` | experimental condition | `PECO:0007359` | **standalone** |
| Viridiplantae | *Viridiplantae* | `NCBITaxon:33090` | `PSO:` | disease | `PSO:0000001` | **standalone** |

