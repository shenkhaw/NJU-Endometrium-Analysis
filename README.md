# Integrated Single-Cell RNA-seq Analysis of the Human Endometrium

## Overview

This repository contains a Python-based bioinformatics workflow for the integration and analysis of publicly available human endometrial single-cell RNA sequencing (scRNA-seq) datasets.

The project aims to characterise major endometrial cell populations, integrate multiple datasets while accounting for sample-associated variation, and investigate transcription factor (TF) expression patterns in stromal and epithelial cell populations.

The workflow uses Scanpy, scVI and scANVI for single-cell preprocessing, integration, dimensionality reduction, clustering and cell-type annotation, followed by sample-aware pseudobulk analysis for transcription factor profiling.

## Dataset summary

The analysis currently includes 15 samples from five public datasets.

| Repository | Accession | Samples |
|---|---|---|
| GEO | GSE250130 | E03, E04, E08, E09, E11, E14 |
| GEO | GSE214411 | N1, N5, N7 |
| GEO | GSE183837 | Ctrl-1, Ctrl-2, Ctrl-3 |
| ArrayExpress | E-MTAB-14039 | FX9011, FX1106 |
| ArrayExpress | E-MTAB-10287 | MRC_Endo8712024 |

## Analysis workflow

### 1. Data loading and integration

- Import the processed single-cell datasets.
- Preserve sample identifiers for downstream sample-aware analysis.
- Harmonise gene identifiers and retain shared genes through an inner gene join.
- Combine the datasets into a unified AnnData object.

### 2. Quality control

The downloaded datasets were already quality-controlled before being obtained. Additional filtering was performed during this analysis using the following criteria:

- Minimum 200 detected genes per cell.
- Minimum 1,000 total UMI counts per cell.
- Maximum 20% mitochondrial UMI counts.
- Exclusion of cells above the 99th percentile of total UMI counts.

Post-filtering quality-control metrics were visualised across samples.

**Note:** The original unfiltered datasets were not used. Therefore, this repository does not reconstruct the original pre-QC distributions or calculate retention rates relative to the original cell populations.

### 3. Highly variable gene selection

- Select 3,000 highly variable genes.
- Use the Seurat v3 selection method.
- Account for sample identity using `sample_id` as the batch variable.

### 4. Batch integration using scVI

The scVI model was trained using raw count data with the following configuration:

| Parameter | Value |
|---|---|
| Latent dimensions | 30 |
| Likelihood | Negative binomial |
| Maximum training epochs | 400 |
| Input | Raw count layer |

The learned latent representation was used for downstream neighbourhood graph construction and visualisation.

### 5. Clustering and visualisation

- Construct a neighbourhood graph using 15 nearest neighbours.
- Generate UMAP embeddings with a minimum distance of 0.3.
- Perform Leiden clustering at resolution 0.5.
- Use the igraph implementation with two iterations.

### 6. Cell-type annotation using scANVI

Preliminary cell-type annotations were assigned using established marker genes and cluster-level expression patterns.

Four broad cell populations were considered:

- Stromal
- Epithelial
- Immune
- Endothelial

The scANVI model was trained using these preliminary labels, with unlabelled cells assigned the `Unknown` category during training. The trained model was then used to generate cell-type predictions across the integrated dataset.

### 7. Pseudobulk transcription factor analysis

Raw UMI counts were aggregated by sample and predicted cell type.

- Grouping variables: `sample_id` and `scanvi_prediction`.
- Minimum group size: 10 cells.
- Target populations: Stromal and Epithelial.
- Reference population: Mean expression of other available cell types within the same sample.
- Statistical test: One-sided paired Wilcoxon signed-rank test.
- Multiple-testing correction: Benjamini–Hochberg procedure.

Log2 fold change, AUC and an exploratory TF importance score were also calculated to support TF ranking and visualisation.

The volcano plots use exploratory thresholds of:

- Log2 fold change > 1.
- Adjusted P-value < 0.05.

These thresholds are analysis-specific and should not be interpreted as universal biological cut-offs.

## Repository structure

```text
NJU-Endometrium-Analysis/
│
├── README.md
├── requirements.txt
├── endometrium_analysis.ipynb
│
├── Pseudobulk/
│   ├── pseudobulk_counts.csv
│   ├── pseudobulk_metadata.csv
│   └── TF_Filtered_Significant.xlsx

```

## Requirements

The analysis uses Python and the major packages are provided in 'requirements.txt'. Python version 1.14.5 **must** be used. Any difference in python version will affect the script. 

## Installation

1. Create a folder and put 'endometrium_final.h5ad', 'requirements.txt' and 'endometrium.analysis.ipnyb' together
  #### Note: Contact author for original h5ad files

2. Open VS code and load the 'endometrium_analysis.ipnyb'

3. Install the dependencies and Run all:

```bash
pip install -r requirements.txt
```

For reproducibility, use the package versions in 'requirements.txt'

## Input data

The integrated AnnData object is expected to contain:

- `sample_id` in `adata.obs`, identifying the original sample.
- `adata.layers["counts"]`, containing raw, non-negative integer UMI counts.
- `X_scVI` or the corresponding trained latent representation where required by the analysis.
- Cell-type labels or predictions in the relevant observation metadata.

Large `.h5ad` files and raw downloaded datasets are not included in this repository. Obtain the original datasets from their respective public repositories and or contact author. 

**Important:** The pseudobulk analysis requires genuine raw counts. Normalised or log-transformed expression values must not be substituted for the raw count layer.

## Running the analysis

1. Download the required datasets from GEO and ArrayExpress.
2. Prepare the processed AnnData objects and preserve sample identifiers.
3. Place the required integrated `.h5ad` file in the project data directory or update the input path in the notebook.
4. Install the Python dependencies.
5. Open the notebook using Jupyter or Visual Studio Code.
6. Select the correct Python environment.
7. Execute the notebook cells in order.
8. Review the generated UMAPs, QC plots, cell-type annotations and TF analysis results.

## Expected outputs

The workflow generates:

- Integrated UMAP visualisations.
- Sample-level integration and batch-mixing visualisations.
- Cell-type marker expression plots.
- Post-QC quality-control distributions.
- Cell-type annotation summaries.
- Pseudobulk raw count matrices and metadata.
- Stromal and epithelial TF comparison tables.
- TF volcano plots and heatmaps.
- Excel workbooks containing selected analysis results.

## Statistical considerations and limitations

The TF analysis uses paired comparisons across samples, which accounts for matching at the sample level. However, the comparison of a target cell type against the mean of other cell types is exploratory and should not be interpreted as a conventional disease-versus-control differential-expression analysis.

The analysis also depends on the quality and annotation consistency of the publicly available input datasets. Differences in experimental protocols, sample composition, sequencing platforms and upstream processing may contribute to residual technical variation.

Becausefiltered source datasets were used, the original pre-QC distributions and cell-retention rates cannot be independently reconstructed.

## References

1. Wolf, F. A., Angerer, P., & Theis, F. J. (2018). SCANPY: large-scale single-cell gene expression data analysis. *Genome Biology*, 19, 15. https://doi.org/10.1186/s13059-017-1382-0

2. Lopez, R., Regier, J., Cole, M. B., Jordan, M. I., & Yosef, N. (2018). Deep generative modeling for single-cell transcriptomics. *Nature Methods*, 15, 1053–1058. https://doi.org/10.1038/s41592-018-0229-2

3. Xu, C., Lopez, R., Mehlman, E., et al. (2021). Probabilistic harmonization and annotation of single-cell transcriptomics data with deep generative models. *Molecular Systems Biology*, 17, e9620. https://doi.org/10.15252/msb.20209620

4. Gayoso, A., Lopez, R., Xing, G., et al. (2022). A Python library for probabilistic analysis of single-cell omics data. *Nature Biotechnology*, 40, 163–166. https://doi.org/10.1038/s41587-021-01206-w

## Acknowledgements

This project uses publicly available human endometrial single-cell RNA sequencing datasets.

