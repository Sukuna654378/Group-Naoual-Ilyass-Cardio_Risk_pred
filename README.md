# When Class Weighting Breaks Calibration: 10-Year CHD Risk Prediction

[![Paper Status](https://img.shields.io/badge/Paper-JAIR%20Format-blue)](papers/Cardiovascular_Risk_Prediction.pdf)
[![Python Version](https://img.shields.io/badge/Python-3.10%2B-brightgreen)](requirements.txt)
[![Reproducibility](https://img.shields.io/badge/Random%20Seed-42-orange)](#-reproducibility--execution)
[![License](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey)](LICENSE)

**Authors:** Naoual Houssni, Ilyass El Bakkali  
**Institution:** SUP'RH School of Management & AI, Morocco  
**Track:** JAIR Track on Data Science for Health  

---

## 📌 Executive Summary

This repository contains the complete, leak-free, uncertainty-quantified analytical pipeline comparing regularized linear models (Elastic-Net Logistic Regression) against tree-based ensembles (Random Forest and Histogram Gradient Boosting) for 10-year Coronary Heart Disease (CHD) risk prediction on the Framingham Heart Study cohort.

### Key Takeaways
1. **Linear Model Dominance:** Regularized Elastic-Net Logistic Regression achieves superior rank discrimination (**ROC-AUC: 0.730**, 95% CI [0.677, 0.786]; **Recall: 0.600**) compared to tree-based ensembles on tabular clinical data.
2. **The Calibration Trap:** Applying `class_weight='balanced'` artificially inflates predicted log-odds by $\approx +1.74$, resulting in negative Brier Skill Scores ($BSS = -0.70$) and negative net clinical benefit across relevant decision thresholds ($p_t \in [0.10, 0.25]$).
3. **Subgroup Recall Disparity:** Operating at a single threshold ($\tau^* = 0.54$) causes a 25.9% sensitivity gap between men (70.9%) and women (45.0%) due to baseline prevalence differences.

---

## 📁 Repository Structure

```text
├── README.md               # Main project documentation & dataset schema
├── requirements.txt        # Python package dependencies
├── ETHICS.md               # Ethical evaluation & subgroup fairness matrix
├── AI_USAGE.md             # Disclosure of Generative AI tool usage
├── notebooks/
│   ├── cardiorisk_pipeline_complet1.ipynb  # Master reproducible notebook
│   └── data_cardiovascular_risk.csv        # Cohort dataset (N=3,390 raw)
├── papers/
│   ├── main.tex                            # LaTeX source manuscript (JAIR template)
│   ├── references.bib                      # Checked BibTeX references
│   ├── jair.cls                            # JAIR style file
│   └── Cardiovascular_Risk_Prediction.pdf  # Compiled manuscript PDF
├── slides/
│   ├── CHD_risk.tex                        # Presentation slide deck LaTeX
│   └── CHD_risk.pdf                        # Compiled slide deck PDF
└── results/
    └── results.csv                         # Saved metrics & bootstrap CIs
```

---

## ⚙️ Reproducibility & Execution

### 1. Installation

Clone the repository and install the required dependencies:

```bash
git clone [https://github.com/Group-Naoual-Ilyass-Cardio_Risk_pred.git](https://github.com/Group-Naoual-Ilyass-Cardio_Risk_pred.git)
cd Group-Naoual-Ilyass-Cardio_Risk_pred
pip install -r requirements.txt
```

### 2. Pipeline Execution & Seed Details

All experiments utilize a fixed global random seed (`random_state = 42`) for train/test partitioning and cross-validation splits.

To re-run the entire data cleaning, nested cross-validation, holdout evaluation, bootstrap hypothesis testing, and Decision Curve Analysis:

1. Launch Jupyter Notebook or Google Colab:
   ```bash
   jupyter notebook notebooks/cardiorisk_pipeline_complet1.ipynb
   ```
2. Execute all cells sequentially. The execution will automatically fit preprocessing steps inside outer-training folds (preventing data leakage) and output `results/results.csv`.

---

## 📊 Key Results Summary

### Holdout Test Set Performance ($N=638$, $95$ events)

| Model Arm | ROC-AUC (95% CI) | PR-AUC | Recall | F1 Score | Brier Score | BSS | Threshold ($\tau^*$) |
| :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| **Elastic-Net LR (balanced)** | **0.730** [0.677, 0.786] | **0.354** | **0.600** | **0.369** | 0.215 | -0.70 | 0.54 |
| **Random Forest (balanced)** | 0.693 [0.640, 0.746] | 0.286 | 0.232 | 0.263 | 0.159 | -0.26 | 0.54 |
| **Hist Gradient Boosting (balanced)** | 0.641 [0.580, 0.698] | 0.271 | 0.284 | 0.283 | 0.176 | -0.39 | 0.59 |
| *Null Prevalence Baseline* | *0.500* | *0.149* | *0.000* | *0.000* | *0.127* | *0.000* | --- |

---

## 🗃️ Dataset Documentation

### 1. Dataset Summary

- **Dataset Name:** Cardiovascular Risk Prediction Dataset (Framingham Heart Study Cohort Subset)
- **Source / URL:** [Kaggle - Cardiovascular Risk Prediction](https://www.kaggle.com/code/rushikeshmuley/coronary-heart-disease-prediction-ipynb) / [NIH Framingham Heart Study](https://www.bumc.bu.edu/camed/news-events/articles/2026/donald-lloyd-jones-md-scm-elected-to-the-national-academy-of-medicine/)
- **License:** Open Database License (ODbL) for Academic and Educational Research
- **Dataset Size:** 3,390 rows × 17 columns (~460 KB)
- **Target Variable:** `TenYearCHD` (Binary: `1` = High Risk, `0` = Low Risk)

### 2. Checklist Validation

| Checklist Criterion | Status | Verification & Evidence |
| :--- | :---: | :--- |
| **1. Academic Reuse License** | **PASS** | Originates from the public Framingham cohort funded by NHLBI/NIH, made available for academic research. |
| **2. Sample Size ($\ge 1,000$)** | **PASS** | Contains **3,390 samples**, exceeding minimum threshold requirements. |
| **3. Clearly Defined Target Task** | **PASS** | Binary classification target (`TenYearCHD`) measuring 10-year risk of developing coronary heart disease. |
| **4. Documentation & Schema** | **PASS** | All 17 variables fully documented with types, units, missingness counts, and clinical ranges. |
| **5. No PII (Anonymized)** | **PASS** | 100% de-identified. Contains only age, sex, and physiological indicators. No names, IDs, or birth dates. |
| **6. Avoid Saturated/Flawed Standards** | **PASS** | Specifically addresses prior benchmark flaws (data leakage, uncalibrated thresholds, lack of uncertainty metrics). |

### 3. Variable Descriptions & Schema

| # | Column Name | Data Type | Missing Count (%) | Description & Clinical Range |
| :---: | :--- | :---: | :---: | :--- |
| 1 | `id` | `int64` | 0 (0.0%) | Row index assigned during extraction *(Excluded prior to modeling)*. |
| 2 | `age` | `int64` | 0 (0.0%) | Age of patient in years (Range: 32 – 70). |
| 3 | `education` | `float64` | 87 (2.57%) | Ordinal code: 1 = High School, 2 = GED, 3 = Some College, 4 = College Grad+. |
| 4 | `sex` | `object` | 0 (0.0%) | Biological sex (`'M'` = Male, `'F'` = Female). |
| 5 | `is_smoking` | `object` | 0 (0.0%) | Smoking status (`'YES'`, `'NO'`). |
| 6 | `cigsPerDay` | `float64` | 22 (0.65%) | Average cigarettes smoked per day (Range: 0 – 70). |
| 7 | `BPMeds` | `float64` | 44 (1.30%) | On BP medication (`0.0` = No, `1.0` = Yes). |
| 8 | `prevalentStroke` | `int64` | 0 (0.0%) | History of stroke (`0` = No, `1` = Yes). |
| 9 | `prevalentHyp` | `int64` | 0 (0.0%) | Hypertensive status (`0` = Normotensive, `1` = Hypertensive). |
| 10 | `diabetes` | `int64` | 0 (0.0%) | Diabetic diagnosis (`0` = No, `1` = Yes). |
| 11 | `totChol` | `float64` | 38 (1.12%) | Total cholesterol in mg/dL (Range: 107 – 696). |
| 12 | `sysBP` | `float64` | 0 (0.0%) | Systolic blood pressure in mmHg (Range: 83.5 – 295.0). |
| 13 | `diaBP` | `float64` | 0 (0.0%) | Diastolic blood pressure in mmHg (Range: 48.0 – 142.5). |
| 14 | `BMI` | `float64` | 14 (0.41%) | Body Mass Index in kg/m² (Range: 15.54 – 56.80). |
| 15 | `heartRate` | `float64` | 1 (0.03%) | Resting heart rate in bpm (Range: 44 – 143). |
| 16 | `glucose` | `float64` | 304 (8.97%) | Fasting glucose in mg/dL *(Imputed in-fold via $k$-NN, $k=5$)*. |
| 17 | `TenYearCHD` | `int64` | 0 (0.0%) | **Target Class:** 10-year risk of CHD (`1` = 511 cases [15.07%], `0` = 2,879 cases [84.93%]). |

---

## ⚖️ Ethics, Fairness & Limitations

A complete analysis of ethical considerations, clinical risks, and limitations is maintained in [ETHICS.md](ETHICS.md).

- **Clinical Misuse Risk:** Raw outputs from `class_weight='balanced'` models overestimate risk probabilities and must not be used for bedside treatment decisions without post-hoc recalibration.
- **Subgroup Disparities:** Unified operating thresholds cause significant sensitivity gaps across biological sex.
- **Provisional Status:** All estimates are derived from a public mirror lacking an official BioLINCC Data Use Agreement.

---

## 🤖 Generative AI Disclosure

In accordance with academic standards, AI systems were used for literature discovery and LaTeX formatting assistance. Full details and prompts are logged in [AI_USAGE.md](AI_USAGE.md).

---

## 📜 Citation

```bibtex
@article{houssni2026classweighting,
  title={When Class Weighting Breaks Calibration: A Leak-Free, Uncertainty-Quantified Comparison of Linear and Tree-Based Models for 10-Year Coronary Heart Disease Risk Prediction on the Framingham Cohort},
  author={Houssni, Naoual and El Bakkali, Ilyass},
  journal={Journal of Artificial Intelligence Research},
  volume={0},
  pages={1--13},
  year={2026}
}
```
