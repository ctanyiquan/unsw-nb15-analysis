# UNSW-NB15 Analysis

A machine learning analysis of the [UNSW-NB15](https://research.unsw.edu.au/projects/unsw-nb15-dataset) network traffic dataset, structured as a series of Jupyter notebooks. The project builds a Network Intrusion Detection System (NIDS) pipeline covering data merging, profiling, cleaning, exploratory data analysis, feature engineering, and modelling for both binary and multi-class intrusion detection.

## Dataset

The UNSW-NB15 dataset contains labelled network traffic records with:
- **Binary target** — `label`: normal (0) vs. attack (1)
- **Multi-class target** — `attack_cat`: specific attack category across 9 types (Generic, Exploits, Fuzzers, DoS, Reconnaissance, Analysis, Backdoor, Shellcode, Worms)

The raw dataset consists of 4 CSV files (~2.5M records combined) plus a features definition file.

## Project Structure

```
unsw-nb15-analysis/
├── data/
│   ├── raw/                  # Original UNSW-NB15 CSV files (4 data files + features file)
│   ├── processed/            # Merged CSV (all 4 data files combined with headers)
│   ├── cleaned/              # Cleaned CSV (after preprocessing pipeline)
│   └── engineered/           # Train/test splits (after feature engineering and resampling)
├── models/
│   ├── stacking_binary.joblib         # Trained binary classification stacking ensemble
│   ├── stacking_binary_threshold.txt  # Optimal decision threshold for binary model
│   └── stacking_classifier.joblib     # Trained multi-class stacking ensemble
├── notebooks/
│   ├── 01_data_merging.ipynb
│   ├── 02_data_profiling.ipynb
│   ├── 03_data_cleaning.ipynb
│   ├── 04_eda.ipynb
│   ├── 05_feature_engineering.ipynb
│   ├── 06_modelling_binary.ipynb
│   └── 07_modelling_classifier.ipynb
├── CHANGELOG.md
├── README.md
└── requirements.txt
```

> **Note:** The `data/` and `models/` directories are not tracked by Git. The datasets are large (~1.5 GB total) and must be downloaded separately from the [UNSW-NB15 dataset page](https://research.unsw.edu.au/projects/unsw-nb15-dataset). Serialized models can be reproduced by running the modelling notebooks.

## Notebooks

### 01 — Data Merging
Loads the features definition file to extract column headers, concatenates the 4 raw CSV files into a single DataFrame, and exports the merged result to `data/processed/UNSW-NB15.csv` (~2.54M records, 49 features).

### 02 — Data Profiling
Inspects shape, column names, data types, and descriptive statistics. Identifies missing values (treating `'-'` as NaN), checks for infinite values in numeric columns, and analyses class distributions for both targets.

### 03 — Data Cleaning
- Drops 9 redundant or identifier columns: `srcip`, `dstip`, `sport`, `dsport`, `Stime`, `Ltime`, `tcprtt`, `stcpb`, `dtcpb`
- Strips whitespace from headers and string columns
- Normalises attack category labels (e.g. `Backdoors` → `Backdoor`)
- Handles missing values in `service`, `ct_flw_http_mthd`, `is_ftp_login`, and `attack_cat`
- Removes duplicate records and impossible records (zero duration with no bytes transferred)
- Exports cleaned data to `data/cleaned/UNSW-NB15.csv` (~2.02M records, 40 features)

### 04 — Exploratory Data Analysis
- **Class distributions** — pie and bar charts for binary label; bar chart for attack categories
- **Cardinality analysis** — unique value counts per feature, classified as Binary / Low / Medium / High to guide encoding decisions
- **Feature distributions** — histograms for all 35 numeric features (log y-axis to handle skew)
- **Outlier detection** — IQR-based outlier counts; box plots for the top 12 most-outlier-affected features split by label
- **Correlation analysis** — Kendall tau correlation matrix; feature-to-label bar chart; tables of weak predictors (`|τ| < 0.05`) and redundant pairs (`|τ| > 0.9`)

Kendall tau is used over Pearson because it is rank-based and more robust to the significant outliers present in this dataset.

### 05 — Feature Engineering
- Drops 13 weak predictors (`|τ| < 0.05` with `Label`) and 4 redundant features from high-correlation pairs
- Engineers 3 interaction features validated via Kendall tau: `bytes_ratio`, `total_bytes`, `bytes_per_dpkt`
- Encodes categoricals: frequency encoding for `proto` (135 values); one-hot encoding for `state` and `service`
- Log-transforms skewed features (`|skew| > 1`)
- Applies 80/20 stratified train-test split
- Resamples training set only: oversamples rare attack categories and undersamples normal traffic to a 3:1 ratio
- Standardises numeric features using training set statistics
- Exports `data/engineered/train.csv` and `data/engineered/test.csv`

### 06 — Binary Classification
Trains and evaluates four classifiers for attack detection (normal vs. attack):
- Logistic Regression (baseline)
- Random Forest
- XGBoost
- Multi-Layer Perceptron (MLP)

Key steps:
- 5-fold stratified cross-validation with score distribution comparison
- Feature importance analysis (MDI, XGBoost gain, permutation importance)
- Threshold optimisation to guarantee recall = 1.00 on the test set
- XGBoost hyperparameter tuning via 30-iteration `RandomizedSearchCV` (macro F1)
- **Stacking ensemble** (all 4 base models + Logistic Regression meta-learner) with bootstrap 95% CIs
- SHAP and LIME explainability, calibration curves, and learning curves

Saves `models/stacking_binary.joblib` and `models/stacking_binary_threshold.txt`.

### 07 — Multi-Class Attack Classification
Trains and evaluates the same four classifiers for attack category prediction (9 attack types + Normal).

Key steps:
- 5-fold stratified cross-validation, model comparison, feature importance
- Per-class precision-recall analysis (one-vs-rest)
- XGBoost hyperparameter tuning scored by macro F1 to weight rare classes equally
- **Stacking ensemble** with bootstrap 95% CIs
- SHAP and LIME explainability, misclassification analysis, calibration curves, learning curves, and partial dependence / ICE plots

Saves `models/stacking_classifier.joblib`.

## Tech Stack

| Tool | Purpose |
|---|---|
| Python 3.14 | Language |
| Jupyter Notebooks | Development environment |
| pandas | Data loading and manipulation |
| numpy | Numerical operations |
| matplotlib / seaborn | Visualisation |
| scikit-learn | Feature engineering, modelling, and evaluation |
| imbalanced-learn | Class resampling |
| xgboost | Gradient boosting classifier |
| shap | Global and local model explainability |
| lime | Local surrogate model explainability |

## Setup

1. Clone the repository:
   ```bash
   git clone https://github.com/ctanyiquan/unsw-nb15-analysis.git
   cd unsw-nb15-analysis
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Download the UNSW-NB15 dataset from the [official page](https://research.unsw.edu.au/projects/unsw-nb15-dataset) and place the files in `data/raw/`.

4. Run the notebooks in order starting from `01_data_merging.ipynb`.
