# UNSW-NB15 Analysis

A machine learning analysis of the [UNSW-NB15](https://research.unsw.edu.au/projects/unsw-nb15-dataset) network traffic dataset, structured as a series of Jupyter notebooks. The project builds a Network Intrusion Detection System (NIDS) pipeline covering data merging, profiling, cleaning, exploratory data analysis, feature engineering, and modelling.

## Dataset

The UNSW-NB15 dataset contains labelled network traffic records with:
- **Binary target** — `label`: normal (0) vs. attack (1)
- **Multi-class target** — `attack_cat`: specific attack category (e.g., DoS, Exploits, Fuzzers)

The raw dataset consists of 4 CSV files (~2.5M records combined) plus a features definition file.

## Project Structure

```
unsw-nb15-analysis/
├── data/
│   ├── raw/                  # Original UNSW-NB15 CSV files (4 data files + features file)
│   ├── processed/            # Merged CSV (all 4 data files combined with headers)
│   └── cleaned/              # Cleaned CSV (after preprocessing pipeline)
├── notebooks/
│   ├── 01_data_merging.ipynb
│   ├── 02_data_profiling.ipynb
│   ├── 03_data_cleaning.ipynb
│   ├── 04_eda.ipynb
│   └── 05_feature_engineering.ipynb
├── CHANGELOG.md
├── README.md
└── requirements.txt
```

> **Note:** The `data/` directory is not tracked by Git. The raw, processed, and cleaned datasets are large (~1.5 GB total) and must be downloaded separately from the [UNSW-NB15 dataset page](https://research.unsw.edu.au/projects/unsw-nb15-dataset).

## Notebooks

### 01 — Data Merging
Loads the features definition file to extract column headers, concatenates the 4 raw CSV files into a single DataFrame, and exports the merged result to `data/processed/UNSW-NB15.csv`.

### 02 — Data Profiling
Inspects shape, column names, data types, and descriptive statistics. Identifies missing values (treating `'-'` as NaN), checks for infinite values in numeric columns, and analyses class distributions for both targets.

### 03 — Data Cleaning
- Drops redundant features: `srcip`, `dstip`, `sport`, `dsport`
- Strips whitespace from headers and string columns
- Handles missing values in `service`, `ct_flw_http_mthd`, `is_ftp_login`, `attack_cat`
- Removes duplicate records and impossible records (e.g., zero duration with non-zero data transfer)
- Exports cleaned data to `data/cleaned/UNSW-NB15.csv`

### 04 — Exploratory Data Analysis
- **Class distributions** — pie and bar charts for binary label; bar chart for attack categories
- **Cardinality analysis** — unique value counts per feature, classified as Binary / Low / Medium / High to guide encoding decisions
- **Feature distributions** — histograms for all 35 numeric features (log y-axis to handle skew)
- **Outlier detection** — IQR-based outlier counts; box plots for the top 12 most-outlier-affected features split by label
- **Correlation analysis** — Kendall tau correlation matrix; feature-to-label bar chart; tables of weak predictors (`|τ| < 0.05`) and redundant pairs (`|τ| > 0.9`)

Kendall tau is used over Pearson because it is rank-based and more robust to the significant outliers present in this dataset.

### 05 — Feature Engineering *(in progress)*
- Drop weak predictors and one feature from each high-correlation pair identified in EDA
- Apply log transformation to skewed features
- Encode categorical variables (`proto`, `state`, `service`)
- Variance thresholding and train/test split

## Tech Stack

| Tool | Purpose |
|---|---|
| Python 3.14 | Language |
| Jupyter Notebooks | Development environment |
| pandas | Data loading and manipulation |
| numpy | Numerical operations |
| matplotlib / seaborn | Visualisation |
| scikit-learn | Feature engineering and modelling |
| imbalanced-learn | Handling class imbalance |

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
