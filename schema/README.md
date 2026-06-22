# Metadata schemas compatible with scFAIR.

## Description
This repo contains all scFAIR schemas for storing single-cell metadata.
In particular, we splitted the schema into:
- **core**: Main schema, applicable to any single cell modality
- **spatial**: Extension of the core schema, for spatial data
- **atac**: Extension of the core schema, for multiomics / ATAC+RNA single-cell datasets
- **perturbation**: Extension of the core schema for perturbational datasets

We also added a **analysis_pipeline** JSON schema for storing the analytical pipeline within the **core** schema.

Of note, initial 7.1.0 schema was forked from release 7.1.0 from [CZI CELLxGENE schema](https://github.com/chanzuckerberg/single-cell-curation/tree/main/schema)

## Validator
We implemented a validator in [ASAP](https://asap-test.epfl.ch/compliance/file-check) where you can submit your dataset for compliance check and correction.

Of note, for this we implemented the rules of the latest schema in a YAML-formatted configuration file, which you can access [here](https://github.com/DeplanckeLab/asap_web/blob/main/src/config/scfair/7.1.0/rules.yaml).
