# Schema

Contact: vincent.gardeux@epfl.ch

Document Status: _Drafting_

Version: 7.1.0+scfair1.0

Current schema: **perturb**

## Schema split

The scFAIR schema is split into multiple part, to differentiate the metadata specific to certain modalities:
- The **core** schema ['schema.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md) is the main schema for all types of single-cell data (scRNA-seq, scATAC-seq, perturbation, spatial, ...). It describes the core metadata between all modalities.
- The **analysis_json** schema ['schema_analysis_json.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_analysis_json.md) describes the analysis pipeline, including software and versionning (cellranger, seurat, scanpy, ...), methods and parameters (umap, louvain, ...), and eventual Docker images
- The **spatial** schema ['schema_spatial.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_spatial.md) describes the additional metadata that are specific to spatial datasets (Visium)
- This **perturb** schema ['schema_perturb.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_perturb.md) describes the additional metadata that are specific to perturbation datasets (CRISPR screens, perturb-seq, ...)
- The **atac** schema ['schema_atac.md'](https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema_atac.md) describes the additional metadata that are specific to scATAC datasets (scATAC-seq, multiomics)

## `obs` (Cell Metadata)

`obs` is a [`pandas.DataFrame`](https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.DataFrame.html).

**In addition** to the metadata described in the *core* schema [https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md]('schema.md'), curators MUST annotate the following columns in the `obs` dataframe:

### genetic_perturbation_id

<table><tbody>
    <tr>
      <th>Key</th>
      <td>genetic_perturbation_id</td>
    </tr>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate if <code>uns['genetic_perturbations']</code> is present; otherwise this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          categorical with <code>str</code> categories. The corresponding <code>organism_ontology_term_id</code> MUST be one of: 
          <ul>
            <li>
              <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A7955"><code>"NCBITaxon:7955"</code></a> for <i>Danio rerio</i>
            </li>
            <li>
              <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A9606"><code>"NCBITaxon:9606"</code></a> for <i>Homo sapiens</i>
           </li>
            <li>
            <a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A10090"><code>"NCBITaxon:10090"</code></a> for <i>Mus musculus</i>
           </li>
         </ul>
         The value MUST be either be <code>"na"</code> or one or more genetic perturbation identifiers in ascending lexical order separated by the delimiter <code>" || "</code> with no duplication of identifiers. Each identifier MUST match a <code>uns['genetic_perturbations'][<i>id</i>]</code>. If the corresponding value of <code>genetic_perturbation_strategy</code> is <code>"control"</code>, then the value of the role in <code>uns['genetic_perturbations'][<i>id</i>]['role']</code> for each matching <code>uns['genetic_perturbations'][<i>id</i>]</code> MUST be <code>"control"</code>.<br><br>All observations MUST NOT contain <code>"na"</code>.
        </td>
    </tr>
</tbody></table>
<br>

### genetic_perturbation_strategy

<table><tbody>
    <tr>
      <th>Key</th>
      <td>genetic_perturbation_strategy</td>
    </tr>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate if <code>obs['genetic_perturbation_id']</code> is present; otherwise, this key MUST NOT be present</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>str</code>. If the corresponding value of <code>obs['genetic_perturbation_id']</code> is <code>"na"</code>, then the value MUST be <code>"no perturbations"</code>; otherwise, the value MUST be one of:
          <ul>
            <li><code>"control"</code></li>
            <li><code>"CRISPR activation screen"</code></li>
            <li><code>"CRISPR interference screen"</code></li>
            <li><code>"CRISPR knockout mutant"</code> describes F0 (founder generation) samples and does not include subsequent generations of homozygous mutants.</li> 
            <li><code>"CRISPR knockout screen"</code></li>
         </ul>
        </td>
    </tr>
</tbody></table>
<br>

## `uns` (Dataset Metadata)

`uns` is a ordered dictionary with a `str` key. The data stored as a value for a key in `uns` MUST be `True`, `False`, `None`, or its size MUST NOT be zero.

**Note:** Terms in italic are auto-filled by the CZI CELLxGENE submission pipeline. scFAIR schema still consider them as required.

**In addition** to the metadata described in the *core* schema [https://github.com/scFAIR/scFAIR/blob/main/schema/7.1.0/schema.md]('schema.md'), curators MUST annotate the following keys and values in `uns`:

### genetic_perturbations

<table><tbody>
    <tr>
      <th>Key</th>
      <td>genetic_perturbations</td>
    </tr>
    <tr>
      <th>Annotator</th>
      <td>Curator MUST annotate if <code>obs['genetic_perturbation_id']</code> is present; otherwise, this key MUST NOT be present.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>dict</code>. The requirements for the key-value pairs and their annotators are documented in the following sections:
          <ul>
          <li>genetic_perturbations[<i>id</i>]</li>
          <li>genetic_perturbations[<i>id</i>]['role']</li>
          <li>genetic_perturbations[<i>id</i>]['protospacer_sequence']</li>
          <li>genetic_perturbations[<i>id</i>]['protospacer_adjacent_motif']</li>
          <li>genetic_perturbations[<i>id</i>]['derived_genomic_regions']</li>
          <li>genetic_perturbations[<i>id</i>]['derived_features']</li>
          <li>genetic_perturbations[<i>id</i>]['derived_features'][<i>feature_id</i>]</li>
         </ul><br/>Additional key-value pairs MUST NOT be present.
        </td>
    </tr>
</tbody></table>
<br/>

#### genetic_perturbations[<i>id</i>]

<table><tbody>
    <tr>
      <th>Key</th>
      <td>
        <i>id</i>
      </td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate. The key is the unique identifier for the genetic perturbation and MUST be an ASCII string, excluding whitespaces, slashes, quotes, or commas. The key MUST NOT be <code>"na"</code>. 
    </tr>
    <tr>
      <th>Value</th>
        <td><code>dict</code>. The requirements for the key-value pairs and their annotators are documented in the following sections:
          <ul>
          <li>genetic_perturbations[<i>id</i>]['role']</li>
          <li>genetic_perturbations[<i>id</i>]['protospacer_sequence']</li>
          <li>genetic_perturbations[<i>id</i>]['protospacer_adjacent_motif']</li>
          <li>genetic_perturbations[<i>id</i>]['derived_genomic_regions']</li>
          <li>genetic_perturbations[<i>id</i>]['derived_features']</li>
          <li>genetic_perturbations[<i>id</i>]['derived_features'][<i>feature_id</i>]</li>
         </ul><br/>Additional key-value pairs MUST NOT be present.
        </td>
    </tr>
</tbody></table>
<br/>

#### genetic_perturbations[<i>id</i>]['role']

<table><tbody>
    <tr>
      <th>Key</th>
      <td>role</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>str</code>. The value MUST either be <code>"control"</code> or <code>"targeting"</code>.
        </td>
    </tr>
</tbody></table>
<br/>

#### genetic_perturbations[<i>id</i>]['protospacer_sequence']

<table><tbody>
    <tr>
      <th>Key</th>
      <td>protospacer_sequence</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>str</code>. This MUST be the protospacer sequence for the <code>genetic_perturbations[<i>id</i>]</code>. The value MUST be an ASCII string composed only of the characters 'A', 'C', 'G', or 'T' which represent the primary DNA nucleotide base codes. Its length MUST be 14-22 characters.
        </td>
    </tr>
</tbody></table>
<br/>

#### genetic_perturbations[<i>id</i>]['protospacer_adjacent_motif']

<table><tbody>
    <tr>
      <th>Key</th>
      <td>protospacer_adjacent_motif</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>str</code>. The protospacer adjacent motif (PAM) MUST be matched at the end of the <code>genetic_perturbations[<i>id</i>]['protospacer_sequence']</code>. Its value MUST be formatted as <code>"3' <b>MOTIF</b>"</code> such as <code>"3' NGG"</code>.<br/><br/><b>MOTIF</b> MUST be the protospacer adjacent motif (PAM) for the <code>genetic_perturbations[<i>id</i>]</code>. Its value MUST be an ASCII string composed only of the characters 'A', 'B', 'C', 'D', 'G', 'H', 'K', 'M', 'N', 'R', 'S', 'T', 'V', 'W', or 'Y' which represent the nucleotide base codes defined in <a href="https://academic.oup.com/nar/article/13/9/3021/2381659">Nomenclature for incompletely specified bases in nucleic acid sequences: recommendations 1984</a>.  <b>MOTIF</b> MUST contain at least one character. 
        </td>
    </tr>
</tbody></table>
<br/>

#### *genetic_perturbations[<i>id</i>]['derived_genomic_regions']*

<table><tbody>
    <tr>
      <th>Key</th>
      <td>derived_genomic_regions</td>
    </tr>
    <tr>
      <th>Annotation</th>
        <td>Curator MUST annotate if <a href="https://genomebiology.biomedcentral.com/articles/10.1186/s13059-025-03488-8">Genome-wide CRISPR guide RNA design and specificity analysis with GuideScan2</a> successfully matched the values of <code>genetic_perturbations[<i>id</i>]['protospacer']</code> and <code>genetic_perturbations[<i>id</i>]['protospacer_adjacent_motif']</code> to genomic regions in the following FASTA references:<br/><br/>
          <table>
            <thead>
              <tr>
                <th>organism_ontology_term_id</th>
                <th>Identifier<br/>Encoding
                <th>FASTA reference</th>
              </tr>
             </thead>
             <tbody>
              <tr>
                <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A7955"><code>"NCBITaxon:7955"</code></a><br/>for <i>Danio rerio</i></td>
                <td>ENSEMBL</td>
                <td><a href="https://ftp.ensembl.org/pub/release-114/fasta/danio_rerio/dna/Danio_rerio.GRCz11.dna.primary_assembly.fa.gz">Danio_rerio.GRCz11.dna.primary_assembly.fa</a></td>
             </tr> 
             <tr>
               <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A9606"><code>"NCBITaxon:9606"</code></a><br/>for <i>Homo sapiens</i></td>
               <td>GENCODE/UCSC</td>
              <td><a href="https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_human/release_48/GRCh38.primary_assembly.genome.fa.gz">GRCh38.primary_assembly.genome.fa</a></td>
            </tr>
            <tr>
              <td><a href="https://www.ebi.ac.uk/ols4/ontologies/ncbitaxon/classes?obo_id=NCBITaxon%3A10090"><code>"NCBITaxon:10090"</code></a><br/>for <i>Mus musculus</i></td>
               <td>GENCODE/UCSC</td>
              <td><a href="https://ftp.ebi.ac.uk/pub/databases/gencode/Gencode_mouse/release_M37/GRCm39.primary_assembly.genome.fa.gz">GRCm39.primary_assembly.genome.fa</a></td>
           </tr>
        </tbody></table>Otherwise, this key MUST NOT be present.<br/><br/>
        </td>
    </tr>
    <tr>
      <th>Value</th>
        <td><code>list[str]</code>. Each element in the unordered <code>list</code> MUST be a <b>unique</b> genomic region matched by <a href="https://genomebiology.biomedcentral.com/articles/10.1186/s13059-025-03488-8">Genome-wide CRISPR guide RNA design and specificity analysis with GuideScan2</a> formatted as <code>"SEQUENCE_ID:START-END(STRAND)"</code> such as <code>"1:12345-12346(+)"</code>.<br/><br/><b>SEQUENCE_ID</b> MUST be a <a href="https://www.ncbi.nlm.nih.gov/genbank/fastaformat/">sequence identifier</a> from the FASTA reference for the <code>organism_ontology_term_id</code>. It MUST be encoded based on ENSEMBL identifiers. If the FASTA reference uses GENCODE/UCSC identifiers, then the sequence identifiers for autosome,  mitochondrial, and sex chromosomes MUST be updated to:
          <ul>
            <li>Remove their <code>"chr"</code> prefix</li>
            <li>Rename the mitochondrial designator from <code>"M"</code> to <code>"MT"</code></li>
          </ul><b>START</b> and <b>START</b> MUST be unsigned integers without leading zeros. <b>START</b> MUST be ≥ 1 and <b>END</b> MUST be > <b>START</b>. The <b>START-STOP</b> coordinates MUST be <i>1-start, fully-closed (1-based)</i>. See <a href="https://genome-blog.gi.ucsc.edu/blog/2016/12/12/the-ucsc-genome-browser-coordinate-counting-systems/">The UCSC Genome Browser Coordinate Counting Systems</a>.<br/><br/><b>STRAND</b> MUST be either <code>"(+)"</code> (forward) or <code>"(-)"</code> (reverse).
        </td>
    </tr>
</tbody></table>
<br/>

#### *genetic_perturbations[<i>id</i>]['derived_features']*

<table><tbody>
    <tr>
      <th>Key</th>
      <td>derived_features</td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate when <code>genetic_perturbations[<i>id</i>]['derived_genomic_regions']</code> is annotated and one or more features in the <a href="#required-gene-annotations">corresponding gene reference</a> of the <code>organism_ontology_term_id</code> overlapped a genomic region by at least one nucleotide; otherwise, this key MUST NOT be present.  
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>dict</code>. The requirements for the key-value pair and its annotator are documented in the following section:
          <ul>
          <li>genetic_perturbations[<i>id</i>]['derived_features'][<i>feature_id</i>]</li>
         </ul><br/>Additional key-value pairs MUST NOT be present.
        </td>
    </tr>
</tbody></table>
<br/>

#### *genetic_perturbations[<i>id</i>]['derived_features'][<i>feature_id</i>]*

<table><tbody>
    <tr>
      <th>Key</th>
      <td>
        <i>feature_id</i>
      </td>
    </tr>
    <tr>
      <th>Annotation</th>
      <td>Curator MUST annotate. The key MUST be the <code>gene_id</code> attribute from the <a href="#required-gene-annotations">corresponding gene reference</a> of the <code>organism_ontology_term_id</code> for a feature that overlapped a genomic region in <code>genetic_perturbations[<i>id</i>]['derived_genomic_regions']</code> by at least one nucleotide.<br/><br/> Version numbers MUST be removed from the <code>gene_id</code> if it is prefixed with <code>"ENS"</code> for <i>Ensembl stable identifier</i>. See <a href="https://ensembl.org/Help/Faq?id=488">I have an Ensembl ID, what can I tell about it from the ID?</a> For example, if the <code>gene_id</code> is <code>“ENSG00000186092.7”</code>, then the <code><i>feature_id</i></code> MUST be <code>“ENSG00000186092”</code>.</td>
    </tr>
    <tr>
      <th>Value</th>
        <td>
          <code>str</code>. If a <code>gene_name</code> attribute is assigned to the <code>gene_id</code> attribute in the <a href="#required-gene-annotations">corresponding gene reference</a> of the <code>organism_ontology_term_id</code>, the value MUST be the <code>gene_name</code>. Otherwise, the value MUST be the <code><i>feature_id</i></code> key.
        </td>
    </tr>
</tbody></table>
<br/>
