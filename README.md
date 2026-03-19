# Cell2location: Spatial Mapping of Mouse and Human Glioblastoma

**DS 596 Special Topics: Learning from Large-scale Biological Data**  
**Researchers:** — Vaidehi Gupta, Chloe Ploss, Addison Yam  
**Based on:** Kleshchevnikov et al. (2022), *Nature Biotechnology*

---

## Overview

This project reproduces and extends the work of Kleshchevnikov et al. (2022), which introduced Cell2location, a Bayesian hierarchical model for spatially resolving fine-grained cell types in spatial transcriptomics data. The original paper demonstrated Cell2location on healthy tissues including mouse brain, human lymph node, and human gut using 10x Genomics Visium.

Our project has three components:

1. **Reproduction** — Reproduce Figure 2b-f from the original paper using the included mouse brain Visium dataset
2. **Extension on a new dataset** — Apply Cell2location to a human glioblastoma Visium dataset to compare tumor vs. healthy brain cellular architecture across species
3. **Extension to a different resolution** — Apply Cell2location to Xenium spatial transcriptomics data, which offers single-cell resolution, and experiment with finding appropriate model parameters for this data type

---

## Scientific Motivation

Glioblastoma is one of the most aggressive and treatment-resistant brain tumors, in large part because of its spatial cellular heterogeneity. Different regions of the same tumor contain dramatically different cell populations. Most existing single-cell studies of glioblastoma lose spatial context entirely because they require tissue dissociation. By applying Cell2location to spatially resolved data, this project aims to map where specific cell types reside in the tumor microenvironment, how glioblastoma remodels the normal cellular architecture of the brain relative to healthy tissue, and whether spatially resolved analysis at higher resolution via Xenium reveals cell type organization not detectable at Visium resolution.

---

## Project Structure

```
cell2location-glioblastoma/
│
├── data/
│  
├── notebooks/
├── results/
├── envs/
│   └── cell2location.sif          # Singularity container image
│
└── README.md
```

---

## Datasets

### Reproduction
- **Mouse brain Visium + snRNA-seq** — Original data from Kleshchevnikov et al., available via ArrayExpress accession numbers E-MTAB-11114 (Visium) and E-MTAB-11115 (snRNA-seq)

### Glioblastoma Extension
- **Human glioblastoma Visium** — Publicly available processed Visium dataset. Recommended sources include the 10x Genomics dataset portal, GEO, or cellxgene Census filtered for brain tumor tissue
- **scRNA-seq reference** — Processed and annotated human glioblastoma scRNA-seq reference from cellxgene Census or a published glioblastoma single-cell atlas deposited on GEO or Zenodo

### Xenium Extension
- **Xenium brain data** — Higher-resolution spatial transcriptomics data. Xenium datasets are available from the 10x Genomics datasets portal

---

## Environment Setup

This project uses a Singularity container to ensure reproducibility across HPC environments. The cell2location developers provide a Docker image that can be converted to Singularity.

### Key dependencies (handled by container)
- Python 3.7+
- cell2location
- scanpy
- anndata
- numpy, pandas

---

## Reproduction: Figure 2b-f

The goal of this component is to reproduce the following panels from Figure 2 of Kleshchevnikov et al.:

| Panel | Content |
|-------|---------|
| 2b | UMAP of 59 annotated mouse brain cell subtypes from snRNA-seq |
| 2c | H&E histology image with annotated anatomical brain regions |
| 2d | Estimated cell abundances of major regional cell types across the tissue |
| 2e | Estimated cell abundances of sparse inhibitory neuron subtypes |
| 2f | Estimated cell abundances of cortical excitatory neuron subtypes across layers |

### Key parameters used in the original paper (mouse brain Visium)

| Parameter | Value |
|-----------|-------|
| n_cells_per_location (N̂) | 8 |
| detection_alpha | 200 |
| n_iterations | 20,000 |
| Shared genes | 12,809 |
| n_locations | 14,968 |

---

## Glioblastoma Visium Extension

This component applies Cell2location to human glioblastoma Visium data, using a processed and annotated glioblastoma scRNA-seq dataset as the cell type reference. The analysis follows the same workflow as the reproduction but adapted to tumor tissue.

---

## Xenium Extension

Xenium is a higher-resolution spatial transcriptomics platform from 10x Genomics that offers near single-cell resolution (~10 µm) compared to Visium's multi-cell spots (~55 µm). This extension tests whether Cell2location can be applied to Xenium data and what parameter adjustments are necessary.


### Parameter experimentation plan
The following parameters will be systematically varied to find appropriate settings for Xenium data:

| Parameter | Values to test | Rationale |
|-----------|---------------|-----------|
| N̂ (cell abundance prior) | 1, 2, 3 | Near single-cell resolution |
| detection_alpha | 20, 200 | Sensitivity to RNA detection gradients |
| cell_count_cutoff | 5, 15, 30 | Gene filtering stringency |
| cell_percentage_cutoff2 | 0.03, 0.05 | Sparse gene inclusion threshold |
| nonz_mean_cutoff | 1.0, 1.12, 1.5 | Minimum expression level threshold |

The goal is to identify parameter settings that produce spatially coherent and biologically interpretable cell type maps, and to characterize how sensitive the results are to these choices.

---

## Parameter Descriptions

These three gene filtering parameters control which genes are included when estimating reference cell type signatures from the scRNA-seq data:

**cell_count_cutoff** — Minimum number of cells a gene must be detected in to be retained. Increasing this makes filtering more strict and keeps only broadly expressed genes. Decreasing it retains genes expressed in fewer cells, which may help capture rare cell type markers.

**cell_percentage_cutoff2** — Minimum percentage of cells that must express a gene for it to be included. Works alongside cell_count_cutoff as a second filtering criterion for sparsely detected genes. Lowering this allows more sparsely detected genes through.

**nonz_mean_cutoff** — Minimum mean expression level calculated only across cells that actually express the gene (non-zero values). This removes genes detected in enough cells but at extremely low levels that likely represent noise or ambient RNA. The original paper used 1.122018 across all analyses.

---

## Expected Outputs

- Reproduced Figure 2b-f panels from the original paper

---

## References

Kleshchevnikov, V., Shmatko, A., Dann, E. et al. Cell2location maps fine-grained cell types in spatial transcriptomics. *Nat Biotechnol* 40, 661–671 (2022). https://doi.org/10.1038/s41587-021-01139-4

Cell2location documentation: https://cell2location.readthedocs.io/

Cell2location GitHub: https://github.com/BayraktarLab/cell2location


