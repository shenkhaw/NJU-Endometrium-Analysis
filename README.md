# Endometrium scRNA-seq Integration and TF Analysis

## Overview
This repository contains the bioinformatics pipeline for integrating 5 public human endometrium scRNA-seq datasets (mid-secretory phase), correcting for batch effects using SCVI/SCANVI, and identifying lineage-specific transcription factors.

## Data
The processed `.h5ad` file (raw counts) is available upon request or via [Insert GEO accession number once published].

## Environment Setup
To reproduce the analysis, install the required packages:
`pip install -r requirements.txt`

## Execution
1. Open `pipeline.ipynb`.
2. Ensure the `integrated_scanvi_RAWCOUNTS_final.h5ad` file is in the same directory.
3. Run all cells. This will generate the 29 publication-ready figures and the final TF Excel sheets.# NJU-Endometrium-Analysis