# Schema

Contact: vincent.gardeux@epfl.ch

Document Status: _Drafting_

Version: 7.1.0+scfair1.0

Current schema: **spatial**

## Schema split

The scFAIR schema is split into multiple part, to differentiate the metadata specific to certain modalities:
- The **core** schema ['schema.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md) is the main schema for all types of single-cell data (scRNA-seq, scATAC-seq, perturbation, spatial, ...). It describes the core metadata between all modalities.
- The **analysis_json** schema ['schema_analysis_json.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md) describes the analysis pipeline, including software and versionning (cellranger, seurat, scanpy, ...), methods and parameters (umap, louvain, ...), and eventual Docker images
- This **spatial** schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md) describes the additional metadata that are specific to spatial datasets (Visium)
- The **perturb** schema ['schema_perturb.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md) describes the additional metadata that are specific to perturbation datasets (CRISPR screens, perturb-seq, ...)
- The **atac** schema ['schema_atac.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md) describes the additional metadata that are specific to scATAC datasets (scATAC-seq, multiomics)

## `obs` (Cell Metadata)

`obs` is a [`pandas.DataFrame`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html).

**In addition** to the metadata described in the *core* schema [https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md]('schema.md'), curators MUST annotate the following columns in the `obs` dataframe:

### array_col

<table><tbody>
    <tr>
      <th>Key</th>
      <td>array_col</td>
    </tr>
    <tr>
      <th>Annotator</th>
         <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is  a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>int</code>. This MUST be the value of the column coordinate for the corresponding spot from the <code>array_col</code> field in <code>tissue_positions_list.csv</code> or <code>tissue_positions.csv</code>. See <a href="https://www.10xgenomics.com/support/software/space-ranger/analysis/outputs/spatial-outputs">Space Ranger Spatial Outputs</a>.<br>
        <br>
          <table>
          <thead>
          <tr>
          <th>For Assay</th>
          <th>Value MUST be in<br>the range between</th>
          </tr>
          </thead>
          <tbody>
            <tr>
              <td><i>Visium Spatial Gene Expression V1</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022857"><code>EFO:0022857</code></a>]</td>
              <td><code>0</code> and <code>127</code></td>
           </tr> 
            <tr>
              <td><i>Visium CytAssist Spatial Gene Expression, 6.5mm</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022859"><code>EFO:0022859</code></a>]</td>
              <td><code>0</code> and <code>127</code></td>
           </tr>
            <tr>
              <td><i>Visium CytAssist Spatial Gene Expression, 11mm</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022860"><code>EFO:0022860</code></a>]</td>
              <td><code>0</code> and <code>223</code></td>
           </tr>
          </tbody></table>
        </td>
    </tr>
</tbody></table>
<br>

### array_row

<table><tbody>
    <tr>
      <th>Key</th>
      <td>array_row</td>
    </tr>
    <tr>
      <th>Annotator</th>
         <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>int</code>. This MUST be value of the row coordinate for the corresponding spot from the <code>array_row</code> field in in <code>tissue_positions_list.csv</code> or <code>tissue_positions.csv</code>. See <a href="https://www.10xgenomics.com/support/software/space-ranger/analysis/outputs/spatial-outputs">Space Ranger Spatial Outputs</a>.<br>
        <br>
          <table>
          <thead>
          <tr>
          <th>For Assay</th>
          <th>Value MUST be in<br>the range between</th>
          </tr>
          </thead>
          <tbody>
            <tr>
              <td><i>Visium Spatial Gene Expression V1</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022857"><code>EFO:0022857</code></a>]</td>
              <td><code>0</code> and <code>77</code></td>
           </tr> 
            <tr>
              <td><i>Visium CytAssist Spatial Gene Expression, 6.5mm</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022859"><code>EFO:0022859</code></a>]</td>
              <td><code>0</code> and <code>77</code></td>
           </tr>
            <tr>
              <td><i>Visium CytAssist Spatial Gene Expression, 11mm</i> [<a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022860"><code>EFO:0022860</code></a>]</td>
              <td><code>0</code> and <code>127</code></td>
           </tr>
          </tbody></table>
        </td>
    </tr>
</tbody></table>
<br>

### in_tissue

<table><tbody>
    <tr>
      <th>Key</th>
      <td>in_tissue</td>
    </tr>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>int</code>. This MUST be the value for the corresponding spot from the <code>in_tissue</code> field in <code>tissue_positions_list.csv</code> or <code>tissue_positions.csv</code> which is either <code>0</code> if the spot falls outside tissue or <code>1</code> if the spot falls inside tissue. See <a href="https://www.10xgenomics.com/support/software/space-ranger/analysis/outputs/spatial-outputs">Space Ranger Spatial Outputs</a>.
        </td>
    </tr>
</tbody></table>
<br>

## `obsm` (Embeddings)

The value for each `str` key MUST be a  `numpy.ndarray` of shape `(n_obs, m)`, where `n_obs` is the number of rows in `X` and `m >= 1`. 

**In addition** to the metadata described in the *core* schema [https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md]('schema.md'), curators MUST annotate the following columns in the `obsm` dataframe:

### spatial

<table><tbody>
    <tr>
      <th>Key</th>
      <td>spatial</td>
    </tr>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate if <code>uns['spatial']['is_single']</code> is <code>True</code>.<br/><br/>Curator MAY annotate if <code>uns['spatial']['is_single']</code> is <code>False</code>.
      <br/><br/>Otherwise, this key MUST NOT be present.</td>
    </tr>
        <tr>
      <th>Value</th>
        <td><code>numpy.ndarray</code> with the following requirements<br/><br/>
          <ul>
          <li>MUST have the same number of rows as <code>X</code> and MUST include at least two columns</li>
          <li>MUST be a <a href="https://numpy.org/doc/stable/reference/generated/numpy.dtype.kind.html"><code>numpy.dtype.kind</code></a> of <code>"f"</code>, <code>"i"</code>, or "<code>u"</code></li>
          <li>MUST NOT contain any <a href="https://numpy.org/devdocs/reference/constants.html#numpy.inf">positive infinity (<code>numpy.inf</code>)</a> or <a href="https://numpy.org/devdocs/reference/constants.html#numpy.NINF">negative infinity (<code>numpy.NINF</code>)</a> values </li>
          <li>MUST NOT contain <a href="https://numpy.org/devdocs/reference/constants.html#numpy.nan">Not a Number (<code>numpy.nan</code>)</a> values</li></ul><br/>If <code>assay_ontology_term_id</code>is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>, the array MUST be created from the corresponding <code>pxl_row_in_fullres</code> and <code>pxl_col_in_fullres</code> fields from <code>tissue_positions_list.csv</code> or <code>tissue_positions.csv</code>. See <a href="https://www.10xgenomics.com/support/software/space-ranger/analysis/outputs/spatial-outputs">Space Ranger Spatial Outputs</a>.
        </td>
    </tr>
</tbody></table>
<br/>

## `uns` (Dataset Metadata)

`uns` is a ordered dictionary with a `str` key. The data stored as a value for a key in `uns` MUST be `True`, `False`, `None`, or its size MUST NOT be zero.

**In addition** to the metadata described in the *core* schema [https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md]('schema.md'), curators MUST annotate the following keys and values in `uns`:

### spatial

<table><tbody>
    <tr>
      <th>Key</th>
      <td>spatial</td>
    </tr>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> or <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030062"><code>"EFO:0030062"</code></a> for <i>Slide-seqV2</i>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>dict</code>. The requirements for the key-value pairs are documented in the following sections:
          <ul>
          <li>spatial['is_single']</li>         
          <li>spatial[<i>library_id</i>]</li>
          <li>spatial[<i>library_id</i>]['images']</li>
          <li>spatial[<i>library_id</i>]['images']['fullres']</li>
          <li>spatial[<i>library_id</i>]['images']['hires']</li>
          <li>spatial[<i>library_id</i>]['scalefactors']</li>
          <li>spatial[<i>library_id</i>]['scalefactors']['spot_diameter_fullres']</li>
          <li>spatial[<i>library_id</i>]['scalefactors']['tissue_hires_scalef']</li>
         </ul><br/>Additional key-value pairs MUST NOT be present.
        </td>
    </tr>
</tbody></table>
<br/>

#### spatial['is_single']

<table><tbody>
    <tr>
      <th>Key</th>
      <td>is_single</td>
    </tr>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> or <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030062"><code>"EFO:0030062"</code></a> for <i>Slide-seqV2</i>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>bool</code>. This MUST be <code>True</code>:
        <ul>
        <li>if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and the dataset represents one Space Ranger output for a single tissue section
      </li>
      <li> if <code>assay_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0030062"><code>"EFO:0030062"</code></a> for <i>Slide-seqV2</i> and the dataset represents the output for a single array on a puck </li>
        </ul>
        Otherwise, this MUST be <code>False</code>.
        </td>
    </tr>
</tbody></table>
<br/>

#### spatial[_library_id_]
<table><tbody>
    <tr>
      <th>Key</th>
      <td>Identifier for the Visium library</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>dict</code>. There MUST be only one <code><i>library_id</i></code>.</td>
    </tr>
</tbody></table>
<br/>

#### spatial[_library_id_]['images']
<table><tbody>
    <tr>
      <th>Key</th>
      <td>images</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>dict</code>
        </td>
    </tr>
</tbody></table>
<br/>

#### spatial[_library_id_]['images']['fullres']
<table><tbody>
    <tr>
      <th>Key</th>
      <td>fullres</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MAY annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          The full resolution image MUST be converted to a<code>numpy.ndarray</code> with the following requirements:<br/><br/>
          <ul>
          <li>The length of <code>numpy.ndarray.shape</code> MUST be <code>3</code></li>
          <li>The <code>numpy.ndarray.dtype</code> MUST be <code>numpy.uint8</code></li>
          <li>The <code>numpy.ndarray.shape[2]</code> MUST be either <code>3</code> (RGB color model for example) or <code>4</code> (RGBA color model for example)</li>
          </ul>
        </td>
    </tr>
</tbody></table>
<br/>

#### spatial[_library_id_]['images']['hires']
<table><tbody>
    <tr>
      <th>Key</th>
      <td>hires</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>tissue_hires_image.png</code> MUST be converted to a<code>numpy.ndarray</code> with the following requirements:<br/><br/>
          <ul>
          <li>The length of <code>numpy.ndarray.shape</code> MUST be <code>3</code></li>
          <li>The <code>numpy.ndarray.dtype</code> MUST be <code>numpy.uint8</code></li>
          <li>If <code>assay_ontology_term_id</code> is <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0022860"><code>"EFO:0022860"</code></a> for <i>Visium CytAssist Spatial Gene Expression, 11mm</i>, the largest dimension in <code>numpy.ndarray.shape[:2]</code> MUST be <code>4000</code>pixels; otherwise, the largest dimension in <code>numpy.ndarray.shape[:2]</code> MUST be <code>2000</code>pixels. See <a href="https://www.10xgenomics.com/support/software/space-ranger/analysis/outputs/spatial-outputs">Space Ranger Spatial Outputs</a></li>
          <li>The <code>numpy.ndarray.shape[2]</code> MUST be either <code>3</code> (RGB color model for example) for <code>4</code> (RGBA color model for example)</li>
          </ul>
        </td>
    </tr>
</tbody></table>
<br/>

#### spatial[_library_id_]['scalefactors']
<table><tbody>
    <tr>
      <th>Key</th>
      <td>scalefactors</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>dict</code>
        </td>
    </tr>
</tbody></table>
<br/>

#### spatial[_library_id_]['scalefactors']['spot_diameter_fullres']
<table><tbody>
    <tr>
      <th>Key</th>
      <td>spot_diameter_fullres</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>float</code>. This must be the value of the <code>spot_diameter_fullres</code> field from <code>scalefactors_json.json</code>. See <a href="https://www.10xgenomics.com/support/software/space-ranger/analysis/outputs/spatial-outputs">Space Ranger Spatial Outputs</a>.</td>
    </tr>
</tbody></table>
<br/>

#### spatial[_library_id_]['scalefactors']['tissue_hires_scalef']
<table><tbody>
    <tr>
      <th>Key</th>
      <td>tissue_hires_scalef</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate if <code>assay_ontology_term_id</code> is a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010961"><code>"EFO:0010961"</code></a> for <i>Visium Spatial Gene Expression</i> and <code>uns['spatial']['is_single']</code> is <code>True</code>; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>float</code>. This must be the value of the <code>tissue_hires_scalef</code> field from <code>scalefactors_json.json</code>. See <a href="https://www.10xgenomics.com/support/software/space-ranger/analysis/outputs/spatial-outputs">Space Ranger Spatial Outputs</a>.
        </td>
    </tr>
</tbody></table>
<br/>
