# scFAIR: an initiative to standardize single-cell genomics data and promote findable, accessible, interoperable, and reusable single-cell data.

## Description
This repo contains open metadata schemas for single cell data extending the existing CZI schemas.

You can access [our portal](https://sc-fair.org/) to browse datasets that are compatible with our schemas.

## Validator
We implemented a validator in [ASAP](https://asap-test.epfl.ch/compliance/file-check) where you can submit your dataset for compliance check and correction.

Of note, for this we implemented the rules of the latest schema in a YAML-formatted configuration file, which you can access [here](https://github.com/DeplanckeLab/asap_web/blob/main/src/config/scfair/7.1.0/rules.yaml).
