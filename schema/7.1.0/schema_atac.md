# Schema

Contact: vincent.gardeux@epfl.ch

Document Status: _Drafting_

Version: 7.1.0+scfair1.0

Current schema: **atac**

## Schema split

The scFAIR schema is split into multiple part, to differentiate the metadata specific to certain modalities:
- The **core** schema ['schema.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md) is the main schema for all types of single-cell data (scRNA-seq, scATAC-seq, perturbation, spatial, ...). It describes the core metadata between all modalities.
- The **analysis_json** schema ['schema_analysis_json.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md) describes the analysis pipeline, including software and versionning (cellranger, seurat, scanpy, ...), methods and parameters (umap, louvain, ...), and eventual Docker images
- The **spatial** schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md) describes the additional metadata that are specific to spatial datasets (Visium)
- The **perturb** schema ['schema_perturb.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md) describes the additional metadata that are specific to perturbation datasets (CRISPR screens, perturb-seq, ...)
- This **atac** schema ['schema_atac.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md) describes the additional metadata that are specific to scATAC datasets (scATAC-seq, multiomics)

## `X` (Matrix Layers)

<b>paired assay</b>. `assay_ontology_term_id` is a descendant of both <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010891"><code>"EFO:0010891"</code></a> for <i>scATAC-seq</i> and <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008913"><code>"EFO:0008913"</code></a> for <i>single-cell RNA sequencing</i>. A gene expression matrix (RNA data) is required.

<b>unpaired assay</b>. `assay_ontology_term_id` is <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0010891"><code>"EFO:0010891"</code></a> for <i>scATAC-seq</i> or a descendant and is not a descendant of <a href="https://www.ebi.ac.uk/ols4/ontologies/efo/classes?obo_id=EFO%3A0008913"><code>"EFO:0008913"</code></a> for <i>single-cell RNA sequencing</i>. A gene activity matrix and not a peak matrix is required.

Also see the requirements for [scATAC-seq assets](#scatac-seq-assets).<br><br>

The following table describes the matrix data and layers requirements that are **assay-specific**. If an entry in the table is empty, the schema does not have any other requirements on data in those layers beyond the ones listed above.

| Assay | "raw" required? | "raw" location | "normalized" required? | "normalized" location |
|-|-|-|-|-|
| paired scRNA-seq (e.g. 10x multiome) | REQUIRED. Values MUST be de-duplicated molecule counts. Each cell MUST contain at least one non-zero value. All non-zero values MUST be positive integers stored as `numpy.float32`. Any two cells MUST NOT contain identical values for all their features. | `AnnData.raw.X` unless no "normalized" is provided, then `AnnData.X` | STRONGLY RECOMMENDED | `AnnData.X` |
| unpaired Accessibility (e.g. ATAC-seq, mCT-seq) | NOT REQUIRED | | REQUIRED | `AnnData.X` | STRONGLY RECOMMENDED |

## scATAC-seq assets

### Requirements

A Dataset MUST meet all of the following requirements to be eligible for scATAC-seq assets:
* <code>assay_ontology_term_id</code> values MUST be either all <i>paired assays</i> or <i>unpaired assays</i>
* <code>is_primary_data</code> values MUST be all <code>True</code>
* <code>organism_ontology_term_id</code> value MUST be either <code>"NCBITaxon:9606"</code> for <i>Homo sapiens</i> or <code>"NCBITaxon:10090"</code> for <i>Mus musculus</i> or one of its descendants. The value determines the required Chromosome Table.

If the <code>assay_ontology_term_id</code> values are all <i>paired assays</i> then the Dataset MAY have a fragments file asset.

If the <code>assay_ontology_term_id</code> values are all <i>unpaired assays</i> then the Dataset MUST have a fragments file asset.

## scATAC-seq Asset: Submitted Fragment File

This MUST be a gzipped tab-separated values (TSV) file.

The curator MUST annotate the following header-less columns. Additional columns and header lines beginning with `#` MUST NOT be included. Each row MUST represent a unique fragment.

### first column

<table><tbody>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>str</code>. This MUST be the reference genome chromosome the fragment is located on.<br/><br/>If the value of organism_ontology_term_id</code> in the associated Dataset is <code>"NCBITaxon:9606"</code> for <i>Homo sapiens</i> then the first column value MUST be a value from the <code>Chromosome</code> column in the <a href="#human-grch38p14">Human Chromosome Table</a>.<br/><br/>
          If the value of <code>organism_ontology_term_id</code> in the associated Dataset is <code>"NCBITaxon:10090"</code> for <i>Mus musculus</i> then the first column value MUST be a value from the <code>Chromosome</code> column in the <a href="#mouse-grcm39">Mouse Chromosome Table</a>.
        </td>
    </tr>
</tbody></table>
<br/>


### second column

<table><tbody>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>int</code>. This MUST be the 0-based start coordinate of the fragment.
        </td>
    </tr>
</tbody></table>
<br/>

### third column

<table><tbody>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>int</code>. This MUST be the 0-based end coordinate of the fragment. The end position is exclusive, representing the position immediately following the fragment interval. The value MUST be greater than the start coordinate specified in the second column and less than or equal to the <code>Length</code> of the <code>Chromosome</code> specified in the first column, as specified in the appropriate Chromosome Table.
        </td>
    </tr>
</tbody></table>
<br/>

### fourth column

<table><tbody>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>str</code>. This MUST be an observation identifier from the <a href="#index-of-pandasdataframe"><code>obs</code> index</a> of the associated Dataset. Every <code>obs</code> index value of the associated Dataset MUST appear at least once in this column.
        </td>
    </tr>
</tbody></table>
<br/>

### fifth column

<table><tbody>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>int</code>. This MUST be the total number of read pairs associated with this fragment. The value MUST be <code>1</code> or greater.
        </td>
    </tr>
</tbody></table>
<br/>

## scATAC-seq Asset: Processed Fragments File

From every submitted fragments file asset, scFAIR Discover MUST generate <code>{artifact_id}-fragments.tsv.gz</code>, a tab-separated values (TSV) file position-sorted and compressed by bgzip.

## scATAC-seq Asset: Fragments File index

From every processed fragments file asset, scFAIR Discover MUST generate <code>{artifact_id}-fragments.tsv.gz.tbi</code>, a <a href="https://www.htslib.org/doc/tabix.html">tabix</a> index of the fragment intervals from the fragments file.

## Chromosome Tables

Chromosome Tables are determined by the reference assembly for the gene annotation versions pinned in this version of the schema. Only chromosomes or scaffolds that have at least one gene feature present are included.

### <a href="https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_48/GRCh38.primary_assembly.genome.fa.gz">Human (GRCh38.p14)</a>

<table>
  <thead>
  <tr>
  <th>Chromosome</th>
  <th>Length</th>
  </tr>
  </thead>
  <tbody>
    <tr>
        <td>chr1</td>
        <td>248956422</td>
    </tr>
    <tr>
        <td>chr2</td>
        <td>242193529</td>
    </tr>
    <tr>
        <td>chr3</td>
        <td>198295559</td>
    </tr>
    <tr>
        <td>chr4</td>
        <td>190214555</td>
    </tr>
    <tr>
        <td>chr5</td>
        <td>181538259</td>
    </tr>
    <tr>
        <td>chr6</td>
        <td>170805979</td>
    </tr>
    <tr>
        <td>chr7</td>
        <td>159345973</td>
    </tr>
    <tr>
        <td>chr8</td>
        <td>145138636</td>
    </tr>
    <tr>
        <td>chr9</td>
        <td>138394717</td>
    </tr>
    <tr>
        <td>chr10</td>
        <td>133797422</td>
    </tr>
    <tr>
        <td>chr11</td>
        <td>135086622</td>
    </tr>
    <tr>
        <td>chr12</td>
        <td>133275309</td>
    </tr>
    <tr>
        <td>chr13</td>
        <td>114364328</td>
    </tr>
    <tr>
        <td>chr14</td>
        <td>107043718</td>
    </tr>
    <tr>
        <td>chr15</td>
        <td>101991189</td>
    </tr>
    <tr>
        <td>chr16</td>
        <td>90338345</td>
    </tr>
    <tr>
        <td>chr17</td>
        <td>83257441</td>
    </tr>
    <tr>
        <td>chr18</td>
        <td>80373285</td>
    </tr>
    <tr>
        <td>chr19</td>
        <td>58617616</td>
    </tr>
    <tr>
        <td>chr20</td>
        <td>64444167</td>
    </tr>
    <tr>
        <td>chr21</td>
        <td>46709983</td>
    </tr>
    <tr>
        <td>chr22</td>
        <td>50818468</td>
    </tr>
    <tr>
        <td>chrX</td>
        <td>156040895</td>
    </tr>
    <tr>
        <td>chrY</td>
        <td>57227415</td>
    </tr>
    <tr>
        <td>chrM</td>
        <td>16569</td>
    </tr>
    <tr>
        <td>GL000009.2</td>
        <td>201709</td>
    </tr>
    <tr>
        <td>GL000194.1</td>
        <td>191469</td>
    </tr>
    <tr>
        <td>GL000195.1</td>
        <td>182896</td>
    </tr>
    <tr>
        <td>GL000205.2</td>
        <td>185591</td>
    </tr>
    <tr>
        <td>GL000213.1</td>
        <td>164239</td>
    </tr>
    <tr>
        <td>GL000216.2</td>
        <td>176608</td>
    </tr>
    <tr>
        <td>GL000218.1</td>
        <td>161147</td>
    </tr>
    <tr>
        <td>GL000219.1</td>
        <td>179198</td>
    </tr>
    <tr>
        <td>GL000220.1</td>
        <td>161802</td>
    </tr>
    <tr>
        <td>GL000225.1</td>
        <td>211173</td>
    </tr>
    <tr>
        <td>KI270442.1</td>
        <td>392061</td>
    </tr>
    <tr>
        <td>KI270711.1</td>
        <td>42210</td>
    </tr>
    <tr>
        <td>KI270713.1</td>
        <td>40745</td>
    </tr>
    <tr>
        <td>KI270721.1</td>
        <td>100316</td>
    </tr>
    <tr>
        <td>KI270726.1</td>
        <td>43739</td>
    </tr>
    <tr>
        <td>KI270727.1</td>
        <td>448248</td>
    </tr>
    <tr>
        <td>KI270728.1</td>
        <td>1872759</td>
    </tr>
    <tr>
        <td>KI270731.1</td>
        <td>150754</td>
    </tr>
    <tr>
        <td>KI270733.1</td>
        <td>179772</td>
    </tr>
    <tr>
        <td>KI270734.1</td>
        <td>165050</td>
    </tr>
    <tr>
        <td>KI270744.1</td>
        <td>168472</td>
    </tr>
    <tr>
        <td>KI270750.1</td>
        <td>148850</td>
    </tr>
</tbody></table>

### <a href="https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M37/GRCm39.primary_assembly.genome.fa.gz">Mouse (GRCm39)</a>

<table>
  <thead>
  <tr>
  <th>Chromosome</th>
  <th>Length</th>
  </tr>
  </thead>
  <tbody>
    <tr>
        <td>chr1</td>
        <td>195154279</td>
    </tr>
    <tr>
        <td>chr2</td>
        <td>181755017</td>
    </tr>
    <tr>
        <td>chr3</td>
        <td>159745316</td>
    </tr>
    <tr>
        <td>chr4</td>
        <td>156860686</td>
    </tr>
    <tr>
        <td>chr5</td>
        <td>151758149</td>
    </tr>
    <tr>
        <td>chr6</td>
        <td>149588044</td>
    </tr>
    <tr>
        <td>chr7</td>
        <td>144995196</td>
    </tr>
    <tr>
        <td>chr8</td>
        <td>130127694</td>
    </tr>
    <tr>
        <td>chr9</td>
        <td>124359700</td>
    </tr>
    <tr>
        <td>chr10</td>
        <td>130530862</td>
    </tr>
    <tr>
        <td>chr11</td>
        <td>121973369</td>
    </tr>
    <tr>
        <td>chr12</td>
        <td>120092757</td>
    </tr>
    <tr>
        <td>chr13</td>
        <td>120883175</td>
    </tr>
    <tr>
        <td>chr14</td>
        <td>125139656</td>
    </tr>
    <tr>
        <td>chr15</td>
        <td>104073951</td>
    </tr>
    <tr>
        <td>chr16</td>
        <td>98008968</td>
    </tr>
    <tr>
        <td>chr17</td>
        <td>95294699</td>
    </tr>
    <tr>
        <td>chr18</td>
        <td>90720763</td>
    </tr>
    <tr>
        <td>chr19</td>
        <td>61420004</td>
    </tr>
    <tr>
        <td>chrX</td>
        <td>169476592</td>
    </tr>
    <tr>
        <td>chrY</td>
        <td>91455967</td>
    </tr>
    <tr>
        <td>chrM</td>
        <td>16299</td>
    </tr>
    <tr>
        <td>GL456210.1</td>
        <td>169725</td>
    </tr>
    <tr>
        <td>GL456211.1</td>
        <td>241735</td>
    </tr>
    <tr>
        <td>GL456212.1</td>
        <td>153618</td>
    </tr>
    <tr>
        <td>GL456219.1</td>
        <td>175968</td>
    </tr>
    <tr>
        <td>GL456221.1</td>
        <td>206961</td>
    </tr>
    <tr>
        <td>GL456239.1</td>
        <td>40056</td>
    </tr>
    <tr>
        <td>GL456354.1</td>
        <td>195993</td>
    </tr>
    <tr>
        <td>GL456372.1</td>
        <td>28664</td>
    </tr>
    <tr>
        <td>GL456381.1</td>
        <td>25871</td>
    </tr>
    <tr>
        <td>GL456385.1</td>
        <td>35240</td>
    </tr>
    <tr>
        <td>JH584295.1</td>
        <td>1976</td>
    </tr>
    <tr>
        <td>JH584296.1</td>
        <td>199368</td>
    </tr>
    <tr>
        <td>JH584297.1</td>
        <td>205776</td>
    </tr>
    <tr>
        <td>JH584298.1</td>
        <td>184189</td>
    </tr>
    <tr>
        <td>JH584299.1</td>
        <td>953012</td>
    </tr>
    <tr>
        <td>JH584303.1</td>
        <td>158099</td>
    </tr>
    <tr>
        <td>JH584304.1</td>
        <td>114452</td>
    </tr>
</tbody></table>
