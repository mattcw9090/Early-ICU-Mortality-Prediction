# Early ICU Mortality Prediction (MIMIC-IV Demo)

A reproducible, notebook-first data science project that builds a leakage-safe ICU cohort, engineers first-24-hour lab features, trains a baseline mortality model, and performs evaluation + error analysis using the **MIMIC-IV Clinical Database Demo (v2.2)**.

> Educational project only. Not medical advice and not intended for clinical use.

---

## Project goals

1. **Cohort definition (leakage-safe)**
   - Unit of analysis: **1 row = 1 ICU stay**
   - Prediction time: **24 hours after ICU admission**
   - Label: **in-hospital mortality** (`hospital_expire_flag`)

2. **Feature engineering**
   - Lab features derived from `labevents` within the window:
     - `intime <= charttime <= prediction_time`
   - Aggregations per ICU stay: min/max/mean/std/count
   - Missingness indicators (whether each lab was measured)

3. **Baseline model**
   - Logistic regression pipeline with:
     - median imputation
     - standard scaling
     - class balancing
   - Patient-level train/test split (`subject_id`) to reduce leakage across multiple stays

4. **Evaluation & error analysis**
   - ROC-AUC, PR-AUC, threshold analysis
   - Calibration curve
   - Review of top false positives / false negatives
   - Lightweight case review tables from raw labs (0–24h window)

---

## Repository structure

- `notebooks/` — sequential notebooks (run in order)
- `data/` — input CSVs (NOT committed)
- `artifacts/` — trained models/metrics (generated locally)
- `reports/` — optional write-ups and exported summaries

---

## Data

This repo is designed to work with:

**MIMIC-IV Clinical Database Demo (v2.2)**  
A publicly available subset of 100 patients with the same schema as MIMIC-IV, excluding free-text clinical notes.  
DOI: `10.13026/07hj-2a80`

> This repository does **not** include the dataset files. You must download them from PhysioNet and place them locally.

### Expected input files

Place these in `data/`:

- `patients.csv`
- `admissions.csv`
- `icustays.csv`
- `labevents.csv`

---

## Setup

### 1) Create environment and install dependencies

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
````

### 2) Start Jupyter

```bash
jupyter notebook
```

---

## Running the project (notebooks in order)

1. **`01_build_icustay_cohort.ipynb`**

   * Builds cohort table:

     * `cohort_icustay_mortality.csv`

2. **`02_engineer_lab_features.ipynb`**

   * Creates lab feature table and model-ready dataset:

     * `lab_features_24h.csv`
     * `dataset_model_ready.csv`

3. **`03_train_baseline_model.ipynb`**

   * Trains baseline logistic regression pipeline and saves:

     * `baseline_logreg.joblib`
     * `baseline_metrics.json`

4. **`04_model_evaluation.ipynb`**

   * Produces evaluation artifacts:

     * `eval_predictions.csv`
     * `threshold_report.csv`

5. **`05_error_analysis.ipynb`**

   * Generates case-review exports and a short write-up:

     * `case_review_false_positives.csv`
     * `case_review_false_negatives.csv`
     * `error_analysis_summary.md`

---

## Key design choices (to avoid leakage)

* **Prediction window** fixed at 24h post ICU admission: `prediction_time = intime + 24h`
* **Features restricted** to timestamps within: `intime <= charttime <= prediction_time`
* **Patient-level split** by `subject_id` to reduce overlap across multiple stays

---

## Limitations

* The demo dataset is small (100 patients), so metrics can be noisy and unstable.
* Labs alone may miss important clinical context (vitals, interventions, comorbidities).
* Threshold tuning shown is for diagnostic understanding only; operational thresholds should be set based on costs/benefits.
