# Introduction-to-QIIME
This repository will host beginner friendly introduction to QIIME2.
The tutorial will focus on QIIME2 for microbiome data analysis.

## Learning Objectives
- What is QIIME2
- How to import data into QIIME2
- Quality control
- Denoising
- OTU Clustering
- Taxonomic classification
- Diversity analysis
- Phylogenetic tree generation(bonus)

## Resources
To follow through the training you will need QIIME2 installed and have an updated browser e.g Chrome for the visualizations of some outputs.

QIIME2 can be installed from this link [QIIME2](https://docs.qiime2.org/)

Installing QIIME2 using a Conda Environment
```
wget https://data.qiime2.org/distro/core/qiime2-2022.2-py38-linux-conda.yml
conda env create -n qiime2-2022.2 --file qiime2-2022.2-py38-linux-conda.yml
conda activate qiime2-2022.2
```
To test Installation run 
```
QIIME --help
```
## Pre-requisite
A basic understand of the commandline is required as the  workflow will be run from the terminal.

An understanding of NGS technologies would be beneficial.
## Reference materials
Estaki, M., Jiang, L., Bokulich, N. A., McDonald, D., González, A., Kosciolek, T., ... & Knight, R. (2020). QIIME 2 enables comprehensive end‐to‐end analysis of diverse microbiome data and comparative studies with publicly available data. Current protocols in bioinformatics, 70(1), e100.
<https://doi.org/10.1002/cpbi.100>

