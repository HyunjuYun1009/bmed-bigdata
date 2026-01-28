# Analysis of UPR Balance Index and Productivity in CHO Cells

## Project Overview

This research project analyzes the correlation between the **UPR (Unfolded Protein Response) balance index** and **protein productivity (Qp)** in CHO (Chinese Hamster Ovary) cells, and develops machine learning models to classify high-productivity and low-productivity cells.

## Background

CHO cells are the most widely used cell line for producing protein therapeutics such as antibodies. However, even under identical culture conditions, cell-specific productivity (Qp) varies significantly and is difficult to predict.

During protein production, the accumulation of unfolded proteins in the endoplasmic reticulum (ER) triggers ER stress, which activates the Unfolded Protein Response (UPR). The UPR consists of:
- **Pro-survival pathway**: Mitigates stress to protect cells
- **Pro-apoptotic pathway**: Induces apoptosis under severe stress

The balance between these two pathways determines cell fate and productivity.

## Research Objectives

1. Define UPR balance index and secretory score, and quantitatively analyze their relationship with CHO cell Qp
2. Build classification models to distinguish high-productivity and low-productivity cells using gene expression data
3. Evaluate the predictive utility of low-dimensional representations learned by Autoencoder

## Dataset

- **Source**: GEO database GSE30321 (Gene expression profiling of CHO production cell lines)
- **Platform**: GPL13791 (Affymetrix CHO-specific microarray)
- **Samples**: 169 out of 295 total samples have Qp values
- **Classification**: Top 30% and bottom 30% classified as high/low producers (103 samples used)

## Methodology

### 1. Data Preprocessing
- Log1p transformation and Z-score standardization
- Selection of top 5,000 variable genes

### 2. UPR & Secretory Gene Set-based Features

Defined the following gene sets to generate pathway features:

**Pro-survival UPR genes**
- HSPA5, HSP90B1, PDIA4, PDIA3, PDIA6, XBP1, ATF6

**Pro-apoptotic UPR genes**
- DDIT3, BBC3, BAX, PMAIP1

**Secretory pathway genes**
- SEC61A1, SEC24D, SAR1A, EDEM1, HERPUD1, HYOU1

**Calculated metrics**
- `uprsurvival`: Pro-survival UPR score
- `uprapoptotic`: Pro-apoptotic UPR score
- `uprbalance`: uprsurvival - uprapoptotic (balance index)
- `secretory`: Secretory pathway score

### 3. Dimensionality Reduction using Autoencoder
- Architecture: 5,000-dim → 256-dim → 2-dim → 256-dim → 5,000-dim
- Training: 50 epochs, Adam optimizer
- Final Reconstruction Loss: 0.48

### 4. Classification Models
- Logistic Regression (4 pathway features)
- Logistic Regression (2-dim Autoencoder latent)
- MLP (2-dim Autoencoder latent)
- Perceptron (4 pathway features)

## Key Results

### Relationship between UPR Balance Index and Productivity
- `uprbalance` and `secretory` scores show positive correlation with Qp
- `uprapoptotic` shows negative correlation
- Higher pro-survival UPR activation and lower pro-apoptotic UPR correlate with increased productivity

### Classification Performance

| Model | Features | Accuracy | ROC AUC |
|-------|----------|----------|---------|
| Logistic Regression | Pathway (4) | **1.00** | **1.00** |
| Logistic Regression | Latent (2) | 0.81 | 0.92 |
| MLP | Latent (2) | 0.81 | 0.95 |
| Perceptron | Pathway (4) | 0.71 | - |

**Cross Validation Results (Pathway Logistic Regression)**
- 5-fold CV: Accuracy 0.805 ± 0.057, AUC 0.854 ± 0.062
- 10 Random Splits: Accuracy 0.810 ± 0.048, AUC 0.881 ± 0.071

## Repository Structure

```
├── final-code.ipynb          # Complete analysis code
├── BMBD_project_report_yunhyeonju.pdf  # Project report
└── README.md                 # This document
```

## Technologies Used

**Python Libraries**
- `pandas`, `numpy`: Data processing
- `scikit-learn`: Machine learning models (Logistic Regression, PCA, StandardScaler)
- `torch`: Autoencoder implementation
- `matplotlib`: Visualization

## Installation & Usage

### 1. Environment Setup
```bash
pip install numpy pandas matplotlib scikit-learn torch
```

### 2. Data Download
- Download [GSE30321](https://www.ncbi.nlm.nih.gov/geo/query/acc.cgi?acc=GSE30321) data from GEO
- Required files: `GSE30321_series_matrix.txt`, `GPL13791_datatable.txt`

### 3. Run Code
```bash
jupyter notebook final-code.ipynb
```

## Key Insights

1. **Simple pathway features achieve perfect classification** (Accuracy 1.0, AUC 1.0)
2. **UPR balance index (uprbalance) and secretory score are critical for productivity prediction**
3. Autoencoder latent representations also show strong performance (AUC 0.92~0.95)
4. A small number of biologically meaningful features outperform thousands of genes

## Limitations & Future Work

- Limited sample size (103 samples)
- Need to explore more diverse gene sets and pathways
- External dataset validation required
- Generalization testing on other cell lines and culture conditions needed

## References

1. Clarke, C. et al. (2011). *Predicting cell-specific productivity from CHO gene expression*. Journal of Biotechnology, 151(2), 159-165.
2. Li, Z.-M. et al. (2022). *Factors Affecting the Expression of Recombinant Protein in CHO Cells*. Frontiers in Bioengineering and Biotechnology, 10.
3. Chakrabarti, A., Chen, A. W., Varner, J. D. (2011). *A review of the mammalian unfolded protein response*. Biotechnology and Bioengineering, 108(12), 2777-2793.
4. Prashad, K., Mehra, S. (2015). *Dynamics of unfolded protein response in recombinant CHO cells*. Cytotechnology, 67(2), 237-254.

## Author

**Hyunju Yun** (Department of Biomedical Engineering, 2023250054)

## License

This project was created for academic research purposes.
