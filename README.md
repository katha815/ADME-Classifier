# ADME-Classifier

Using machine-learning algorithms and genomics features to build binary classifiers that distinguish human ADME (Absorption, Distribution, Metabolism, and Excretion) gene variants from non-ADME variants.

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/katha815/ADME-Classifier/blob/main/Different_ML_models.ipynb)

---

## Overview

ADME genes govern how drugs are absorbed, distributed, metabolized, and excreted in the human body. This project frames variant classification as a supervised binary classification problem: given a set of genomic and conservation features for a single nucleotide variant (SNV), predict whether it falls in an ADME gene.

The main experimental notebook is [`Different_ML_models.ipynb`](Different_ML_models.ipynb). It covers the full ML workflow from data loading through model evaluation with chromosome-based cross-validation.

---

## Features / Inputs

Each variant is represented by the following feature groups:

| # | Feature group | Description |
|---|---------------|-------------|
| 1 | **phastCons scores** | 6 conservation scores from the phastCons track |
| 2 | **phyloP scores** | 6 scores from the phyloP conservation track |
| 3 | **vepDistance** | Distance to the nearest annotated transcript (VEP) |
| 4 | **Base-change dummies** | One-hot encoded nucleotide substitution type (e.g. A→C, A→G, …) |
| 5 | **VEP / Ensembl consequence** | Functional consequence annotation (e.g. missense, synonymous) |
| 6 | **SIFT / PolyPhen scores** | Predicted functional impact of the amino-acid change |
| 7 | **Population frequencies** | Allele frequency information |

Labels are stored in a `Label` column; chromosome, reference allele, and alternate allele columns are excluded from the feature matrix.

---

## Workflow

```
Load CSV data
    │
    ▼
One-hot encode categorical columns (pd.get_dummies)
    │
    ▼
Chromosome-based split preparation
  ├─ Leave-One-Chromosome-Out (LOCO) folds
  └─ Simple train / validation / test split (25 % test)
    │
    ▼
StandardScaler normalisation
    │
    ▼
Model training & evaluation
  ├─ SGDClassifier  (linear baseline)
  ├─ XGBClassifier  (default + Bayesian hyperparameter search)
  └─ GradientBoostingClassifier
    │
    ▼
Probability thresholding
  └─ Convert predict_proba output to labels using a confidence threshold;
     variants below the threshold are omitted and the omitted % is reported
```

---

## Models Tried

| Model | Notes |
|-------|-------|
| **SGDClassifier** | Linear baseline; trained per LOCO fold and with K-fold CV |
| **XGBClassifier** | Default hyperparameters; Bayesian optimisation (scikit-optimize `BayesSearchCV`); `approx` tree method variant |
| **GradientBoostingClassifier** | Default hyperparameters; Bayesian optimisation |

---

## Evaluation Strategy

Two evaluation schemes are used:

* **Leave-One-Chromosome-Out (LOCO):** In each fold, all variants on one chromosome form the test set and the rest are used for training. This tests generalization to chromosomes unseen during training.
* **Simple split:** A standard random 75 / 25 train-test split with a nested validation set for hyperparameter tuning.

Metrics reported:
* **Accuracy** – average per-fold accuracy and whole-dataset accuracy
* **ROC AUC** – area under the receiver operating characteristic curve
* **Omitted %** – fraction of variants whose predicted probabilities fall below the chosen confidence threshold (probability thresholding experiments only)

Example results recorded in notebook comments (LOCO, XGBoost, no optimisation):

```
Average Accuracy : 61.20 %
Whole Accuracy   : 62.57 %
AUC Score        : 0.6163
```

With probability thresholding (threshold = 0.8):

```
Whole Accuracy   : 68.12 %   Omitted: 31.44 %
AUC Score        : 0.6691
```

> These figures are illustrative outputs from the notebook and may vary with different random seeds or data versions.

---

## Repository Layout

```
ADME-Classifier/
├── Different_ML_models.ipynb          # Main experiment notebook
├── Different_ML_models_Box_plot.ipynb # Box-plot visualisations of results
├── Different_ML_models_Thresholding_results_code.ipynb  # Thresholding analysis
├── Baseline_XGBoost_Model_LeaveOneChrom_OtherTesting.ipynb
├── XGBoost_Pars_Optimised_for_4_training_feature_groups.ipynb
├── *_Features_Extraction*.ipynb       # Feature extraction pipelines
├── Dataset_Expansion*.ipynb           # Dataset expansion experiments
├── Extracted Features/                # Pre-extracted feature files
├── dataS/                             # Supporting data directory
└── thresholding results (graphs)/     # Saved threshold sweep plots
```

---

## Requirements / Setup

The notebooks are designed to run in **Google Colab** with Google Drive mounted. Key dependencies:

```
numpy
pandas
scikit-learn
xgboost
scikit-optimize   # pip install scikit-optimize
numba             # pip install numba
matplotlib
```

Install any missing packages at the top of the notebook or with:

```bash
pip install scikit-optimize numba xgboost
```

---

## Usage / Reproducibility

1. **Open the notebook in Colab** using the badge at the top of this file, or clone the repo and open locally with Jupyter.
2. **Mount Google Drive** and set the `base` variable to the folder that contains your input CSV (e.g. `Odata_fzero.csv`).
3. **Run all cells** in order:
   - Data loading and one-hot encoding
   - Chromosome split preparation
   - Feature scaling
   - Model training and LOCO evaluation
   - (Optional) Probability thresholding sweep
4. Results (accuracy, AUC, omitted %) are printed inline and can be collected for the box-plot notebook.

---

## Contributing

Pull requests and issues are welcome. Please open an issue first to discuss any significant changes.
