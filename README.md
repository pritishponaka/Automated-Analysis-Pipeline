# Automated Analysis Pipeline: Diabetes Prediction

**Author:** Your Name Here
**Course/Assignment:** Automated Analysis Pipeline Exploration

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://drive.google.com/file/d/1VOmG54RWPny1HEumAsZFq-jENF4JkpIw/view?usp=sharing)
---

## 1. Purpose

This notebook demonstrates a structured, reproducible, and adaptive analysis workflow for health data. It uses
the **Pima Indians Diabetes Database** (National Institute of Diabetes and Digestive and Kidney Diseases), which
contains diagnostic measurements from women aged 21 or older of Pima Indian heritage. The prediction task is
binary: classify whether a patient has diabetes (`Outcome` = 1) or not (`Outcome` = 0).

The pipeline moves through these stages, using conditional logic to decide what to do at each step:

| Stage | What it does |
|---|---|
| **Data Ingestion** | Accepts one uploaded file, selects the correct reader for its format, and re-prompts (up to 3 attempts) if the file is unsupported, unreadable, empty, or has duplicate column names |
| **Data Exploration** | Previews the data, reports types and missing values, quantifies hidden zeros, and converts impossible zeros to missing values |
| **Exploratory Data Analysis** | Correlation tables and heatmaps (Pearson, Spearman, Phi-K) and distribution and relationship plots |
| **Inferential Statistics** | One-sample t-test, Welch's independent samples t-test, one-way ANOVA, and chi-square test of independence |
| **Supervised Machine Learning** | Stratified 70/30 split, median imputation, logistic regression, and a decision tree tuned with cross-validated grid search |
| **Parameter Demonstrations** | Shows how changing a setting (missing-value threshold, tuning metric) changes the results |

The notebook's **Table of Contents** and **Summary of Modifications and Findings** (at the top) list every
section, which cells were modified or added, and which provided cells were left unchanged.

## 2. Files

| File | Description |
|---|---|
| `test.ipynb` (or your final notebook name) | The Google Colab notebook |
| `Example_Dataset_Diabetes.csv` | The dataset: 768 patients, 9 columns |
| `README.md` | This file |

## 3. How to run the analysis

1. **Open the notebook in Google Colab** using the badge above, or upload the `.ipynb` file at
   [colab.research.google.com](https://colab.research.google.com) (File → Upload notebook).
2. **Start a clean session:** *Runtime → Restart session*.
3. **Run all cells in order:** *Runtime → Run all*.
4. **Upload the dataset when prompted.** The first code cell opens a file picker. Choose
   `Example_Dataset_Diabetes.csv` (one file only). Supported formats: `.csv`, `.tsv`, `.txt`, `.xlsx`, `.xls`,
   `.json`, `.parquet`. You have 3 attempts (`MAX_ATTEMPTS`).
5. **Review the outputs** under each section. The Markdown cells explain what each output shows and how to
   interpret it.

Notes on running:

- **Run top to bottom.** Later cells depend on earlier ones. For example, the invalid-values cell must run before
  the missing-value report. If you re-run a single cell out of order, its results can change (the replacement
  report shows 0 replaced values the second time because the zeros are already gone). If that happens, restart
  the session and run everything again.
- **Expected runtime:** a few minutes. The decision tree grid search (72 settings × 5 folds) is the slowest step.
- **Reproducibility:** all random steps use `random_state=42`, so a clean top-to-bottom run should reproduce the
  reported results (given similar library versions). The *View 15 random rows* cell is intentionally random.
- **User settings:** variables near the top of each modified cell (for example `MAX_ATTEMPTS`,
  `HIGH_MISSING_THRESHOLD`, `scoring_metric`) are meant to be changed. Code below a
  `DO NOT CHANGE ANY CODE BELOW THIS LINE` divider is provided code and should be left as is.

## 4. Required software and libraries

The notebook is written for **Google Colab** (Python 3), which includes everything below except `phik`.
No local installation is needed.

| Library | Used for |
|---|---|
| `pandas`, `numpy` | Data loading and manipulation |
| `matplotlib`, `seaborn` | Plots and heatmaps |
| `scipy` | t-tests and chi-square test |
| `statsmodels` | One-way ANOVA |
| `scikit-learn` | Train/test split, imputation, scaling, logistic regression, decision tree, grid search, metrics |
| `phik` | Phi-K correlation (installed automatically by the notebook with `pip`; needs an internet connection) |
| `openpyxl` / `pyarrow` | Reading Excel / Parquet files (only if you upload those formats) |
| `google.colab` | The file upload widget (Colab only) |

To run **outside Colab**, install the libraries (`pip install pandas numpy matplotlib seaborn scipy statsmodels
scikit-learn phik`) and replace the `google.colab.files.upload()` step with a direct `pd.read_csv(...)` call.

## 5. Dataset

| Variable | Description |
|---|---|
| `Pregnancies` | Number of pregnancies |
| `Glucose` | Plasma glucose concentration |
| `D_BP` | Diastolic blood pressure |
| `Skin_Thickness` | Skin fold thickness |
| `Insulin` | Serum insulin |
| `BMI` | Body mass index |
| `Pedigree` | Diabetes pedigree function |
| `Age` | Age in years |
| `Outcome` | 0 = no diabetes, 1 = diabetes (500 vs 268 patients) |

Column names follow this file (`D_BP`, `Skin_Thickness`, `Pedigree`). Other copies of the dataset use different
names (for example `BloodPressure`, `SkinThickness`), which would need to be updated in the code.

## 6. Assumptions and limitations

**Data assumptions**

- **Zeros in `Glucose`, `D_BP`, `Skin_Thickness`, `Insulin`, and `BMI` are treated as missing**, because these
  values are not physiologically possible. This list comes from domain knowledge. `Pregnancies` and `Outcome` keep
  their zeros, which are valid.
- **Missing values are filled with the training-set median**, learned from the training data only to avoid
  leakage. This assumes the data are reasonably missing at random and is sensitive to skewed variables.
- The outcome is binary and coded 0/1, and all predictors are numeric. The pipeline stops with an error otherwise.
- The upload step expects one file with column names in the first row. For Excel files only the first worksheet is read.

**Modeling and statistical limitations**

- **Heavy missingness:** about half of `Insulin` (48.7%) and 30% of `Skin_Thickness` (29.6%) are imputed, so
  results involving them should be read with caution. `Insulin` is flagged by the missing-value threshold for
  this reason.
- **Small test set:** the 30% test set has 231 patients, only 81 of whom have diabetes. Differences of a few
  patients between models are within random variation.
- **Class imbalance:** about 65% of patients do not have diabetes, so accuracy alone can be misleading. Recall,
  F1, and ROC AUC are reported alongside it.
- **Tuning scope:** tuned parameters and cutoffs are chosen on the training data only. They are not validated on
  external data, and results depend on the random split.
- **Multiple testing:** four hypothesis tests are run at alpha = 0.05 without correction.
- **Observational data:** the tests show associations, not causes.
- **Generalizability:** the data come from one population (women of Pima Indian heritage, aged 21 or older).
  Conclusions should not be applied to other groups.
- **The data-type cell** converts `Outcome` to a `category` (as provided). As a result, the Pearson heatmap and
  the all-variable histogram grid omit `Outcome`. The ML section is not affected.
- **Not for clinical use:** this notebook is for teaching and demonstration only. The models are not validated
  for diagnosis or patient care.

## 7. Key results (clean run, `random_state=42`)

| Model (test set, n = 231) | Accuracy | Recall | Precision | F1 | ROC AUC |
|---|---|---|---|---|---|
| Logistic regression (cutoff 0.5) | 0.745 | 0.531 | 0.672 | 0.593 | 0.836 |
| Logistic regression (cutoff 0.443) | 0.740 | 0.568 | 0.648 | 0.605 | 0.836 |
| Decision tree (default) | 0.693 | 0.531 | 0.566 | 0.548 | 0.655 |
| Decision tree (tuned for F1) | 0.719 | 0.802 | 0.570 | 0.667 | 0.775 |

`Glucose` and `BMI` are the most influential predictors in both the correlation analysis and the models.
See the notebook for the full interpretation.
