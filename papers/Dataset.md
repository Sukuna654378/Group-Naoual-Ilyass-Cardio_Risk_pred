# Dataset

## 1. Source & Provenance
* **Origin:** The dataset originates from the foundational **Framingham Heart Study (FHS)**, a landmark longitudinal prospective cohort study initiated in 1948 in Framingham, Massachusetts, conducted under the auspices of the National Heart, Lung, and Blood Institute (NHLBI) in collaboration with Boston University.
* **Distribution & License:** The study dataset is accessible as an open research extract distributed via Kaggle under the **Creative Commons CC0: Public Domain** / Open Database License (ODbL) for academic and educational benchmarking.
* **Collection Window & Modality:** Baseline physical examinations and biological measurements were administered between the 1950s and 1970s. Data collection was carried out through standardized biennial clinical visits, physician-conducted physical exams, laboratory blood tests (fasting state), and structured self-administered health behavior questionnaires. Clinical endpoints were tracked over a **10-year prospective follow-up window**.

---

## 2. Cohort Description & Target Specification
* **Sample Size:** The raw dataset comprises **3,390 patient observations** and **17 initial features** (including the administrative extraction identifier `id`).
* **Variables & Domain Categories:**
  * *Demographic:* `age` (years, integer), `male` (biological sex: 0 = Female, 1 = Male), `education` (ordinal categorical: 1 = Some high school, 2 = High school grad/GED, 3 = Some college, 4 = College grad).
  * *Behavioral / Lifestyle:* `currentSmoker` (binary indicator: 0 = No, 1 = Yes), `cigsPerDay` (cigarettes smoked per day, continuous count).
  * *Medical History / Comorbidities:* `BPMeds` (antihypertensive medication: 0 = No, 1 = Yes), `prevalentStroke` (history of stroke: 0 = No, 1 = Yes), `prevalentHyp` (prevalent hypertension: 0 = No, 1 = Yes), `diabetes` (diagnosed diabetes: 0 = No, 1 = Yes).
  * *Physical / Physiological Examination:* `sysBP` (systolic blood pressure in mmHg), `diaBP` (diastolic blood pressure in mmHg), `BMI` (body mass index in $\text{kg/m}^2$), `heartRate` (resting pulse in beats per minute).
  * *Laboratory / Biological Assays:* `totChol` (total serum cholesterol in mg/dL), `glucose` (fasting blood glucose in mg/dL).
* **Target Variable:** `TenYearCHD` — binary outcome indicating whether the participant developed clinical Coronary Heart Disease (fatal or non-fatal myocardial infarction, angina pectoris, or coronary insufficiency) within the 10-year observation period ($0 = \text{No event}$, $1 = \text{Event confirmed}$).

---

## 3. Key Descriptive Statistics & Visualizations

### Descriptive Summary (Post-Cleaning Analytical Cohort, $N = 3,189$)

| Variable | Type | Unit | Mean | Std Dev | Median | Min | Max | Missing ($N$) | Missing (\%) |
| :--- | :--- | :--- | :---: | :---: | :---: | :---: | :---: | :---: | :---: |
| `age` | Continuous | years | 49.6 | 8.6 | 49.0 | 32.0 | 70.0 | 0 | 0.0% |
| `male` | Binary | 0/1 | 0.44 | 0.50 | 0.0 | 0.0 | 1.0 | 0 | 0.0% |
| `cigsPerDay` | Discrete | cigs/day | 9.0 | 11.9 | 0.0 | 0.0 | 70.0 | 0 | 0.0% |
| `BPMeds` | Binary | 0/1 | 0.03 | 0.17 | 0.0 | 0.0 | 1.0 | 0 | 0.0% |
| `prevalentHyp` | Binary | 0/1 | 0.31 | 0.46 | 0.0 | 0.0 | 1.0 | 0 | 0.0% |
| `totChol` | Continuous | mg/dL | 236.9 | 44.4 | 234.0 | 107.0 | 696.0 | 0 | 0.0% |
| `sysBP` | Continuous | mmHg | 132.6 | 22.0 | 128.5 | 83.5 | 295.0 | 0 | 0.0% |
| `diaBP` | Continuous | mmHg | 82.9 | 11.9 | 82.0 | 48.0 | 142.5 | 0 | 0.0% |
| `BMI` | Continuous | $\text{kg/m}^2$ | 25.8 | 4.1 | 25.4 | 15.5 | 56.8 | 0 | 0.0% |
| `heartRate` | Continuous | bpm | 75.7 | 12.0 | 75.0 | 44.0 | 143.0 | 0 | 0.0% |
| `glucose` | Continuous | mg/dL | 81.9 | 23.9 | 78.0 | 40.0 | 394.0 | 304 | 9.5%* |
| `TenYearCHD` | Target | 0/1 | 0.151 | 0.358 | 0.0 | 0.0 | 1.0 | 0 | 0.0% |

*\*Note: Fasting glucose missingness ($N=304$) is intentionally preserved for intra-fold $k$-NN imputation.*

### Key Exploratory Figures
1. **Figure 1 — Target Distribution & Class Imbalance:**  
   Out of 3,189 analyzed patients, **482 individuals ($15.11\%$)** experienced a CHD event within 10 years, compared to **2,707 event-free individuals ($84.89\%$)**. This 1:5.6 imbalance ratio demonstrates that overall accuracy is an uninformative metric; modeling must explicitly optimize Precision-Recall dynamics, class weighting, and decision thresholds.
2. **Figure 2 — Sex and Age Stratification of CHD Risk:**  
   Incidence displays strong divergence across biological sex: male participants exhibit a baseline CHD incidence of **18.2%**, whereas females exhibit an incidence of **12.1%**. Concurrently, risk accelerates sharply beyond age 50 in both sexes, with hypertensive systolic pressures ($\text{sysBP} \ge 140\text{ mmHg}$) compounding risk non-linearly.

---

## 4. Preprocessing & Sample Accounting

To guarantee full methodological reproducibility and eliminate silent sample attrition, all exclusion and imputation steps are documented with precise before-and-after headcounts:

```
[Raw Initial Cohort]
  N = 3,390 rows | 17 columns
        │
        ├── 1. Identifier Removal: Drop administrative 'id' column (17 -> 16 features)
        │
        ├── 2. Exact Duplicates: Drop 10 identical duplicate rows (-10)
        │      Subtotal: N = 3,380 rows
        │
        ├── 3. Complete Case Filtering on Low-Missingness Covariates (< 3% missingness):
        │      - education missing:     105 rows
        │      - BPMeds missing:         53 rows
        │      - totChol missing:        50 rows
        │      - cigsPerDay missing:     29 rows
        │      - BMI missing:            19 rows
        │      - heartRate missing:       1 row
        │      Accounting for co-occurring missingness across features:
        │      Total rows removed via listwise deletion: -191 rows
        │
        └── 4. Glucose Preservation & Imputation:
               - glucose has 304 missing values (8.97% of raw cohort).
               - Row deletion was REJECTED because it would disproportionately discard positive cases.
               - Decision: Retain missing values; apply k-NN imputation (k = 5) strictly INSIDE 
                 cross-validation training folds.
        │
[Cleaned Analytical Cohort]
  N = 3,189 rows | 16 columns
  Attrition: 201 rows discarded (5.93% total cohort reduction)
  Class Breakdown: Negative (0) = 2,707 (84.89%) | Positive (1) = 482 (15.11%)
```

---

## 5. Splitting Protocol & Anti-Leakage Safeguards
* **Validation Strategy:** **Repeated Stratified 5-Fold Cross-Validation** (`StratifiedKFold`, 5 splits, shuffle enabled).
* **Partition Ratios:** At each fold rotation, **80% of data** ($N \approx 2,551$) is assigned to training and **20%** ($N \approx 638$) to out-of-fold validation/testing.
* **Stratification Justification:** Stratifying directly on `TenYearCHD` ensures that every single fold preserves the exact ~15.1% positive prevalence ($\approx 96$ positive events per validation fold), preventing fold-to-fold evaluation variance.
* **Temporal & Group Independence:** The dataset represents a cross-sectional snapshot per participant (one row per distinct patient). There is no longitudinal repeated-measures structure per patient ID and no chronological ordering, rendering grouped or rolling-origin splits inapplicable.
* **Anti-Leakage Execution:** Feature scaling (`StandardScaler`) and missingness imputation (`KNNImputer`, $k=5$) are encapsulated inside a scikit-learn `Pipeline`, fit **strictly on training fold partitions** and applied out-of-sample to validation folds.
* **Reproducibility Anchors:** Global pseudorandom seed is strictly locked at **`seed = 42`**. All fold partition indices are version-controlled and archived in `splits/stratified_splits_indices_seed42.json`, alongside individual fold index tables (`splits/fold_0_train_indices.csv`, `splits/fold_0_test_indices.csv`).

---

## 6. Limitations, Representation Gaps & Cohort Biases

A candid assessment of dataset biases is essential for establishing the clinical boundaries of any derived predictive model:

1. **Racial and Geographic Homogeneity (Severe Selection Bias):**  
   The Framingham Heart Study cohort was sampled exclusively from a single suburban town in Massachusetts during the mid-20th century. Over **99% of participants were of Caucasian descent and middle-class socio-economic status**. The cohort entirely excludes Black, Hispanic, Indigenous, Asian, and Middle Eastern/North African populations. Genetic predispositions, differing social determinants of health, and distinct baseline cardiovascular risk profiles mean **this model cannot be deployed in non-Caucasian or multi-ethnic populations without external validation and local recalibration**.

2. **Historical Care Drift (Temporal Obsolescence):**  
   Data collection reflects clinical and public health realities from 1950 to 1970:
   * **Statin therapy did not exist**; pharmacological lipid-lowering was virtually absent.
   * **Hypertension diagnostic thresholds** and clinical management targets were far more lenient than modern AHA/ESC guidelines ($\ge 130/80\text{ mmHg}$).
   * Cigarette smoking was highly prevalent and socially normative, with higher per-capita tar/nicotine exposure than contemporary tobacco or electronic vaping products.  
   Consequently, baseline hazard rates and relative risk weights reflect a historical population prior to modern preventive pharmacotherapy and revascularization standards.

3. **Absence of Modern Biomarkers & Granular Lipids:**  
   The open benchmark dataset records only total serum cholesterol (`totChol`). It **completely lacks High-Density Lipoprotein (HDL-C), Low-Density Lipoprotein (LDL-C), triglycerides, high-sensitivity C-reactive protein (hs-CRP), and Glycated Hemoglobin (HbA1c)**. Modern clinically validated risk scores (such as the ACC/AHA 2013 Pooled Cohort Equations, QRISK3, or ESC SCORE2) depend fundamentally on HDL and non-HDL ratios. Models trained on this dataset operate at an informational deficit compared to modern clinical guidelines.

4. **Self-Reporting and Information Bias:**  
   Behavioral markers—most notably `cigsPerDay` and `currentSmoker`—rely entirely on patient self-reports. Self-reporting is known to suffer from recall bias and social desirability underreporting, introducing measurement noise into one of the primary behavioral risk factors.

5. **Class Imbalance & Asymmetric Clinical Costs:**  
   With a positive class rate of 15.11%, default model classification thresholds ($\tau = 0.50$) induce severe **sensitivity collapse**, where models achieve misleadingly high accuracy ($\sim 85\%$) by predicting the majority negative class while missing over $70\text{--}90\%$ of individuals who will suffer a fatal or debilitating coronary event. Clinical utility requires explicit cost-sensitive reweighting and decision threshold optimization anchored to Precision-Recall and Decision Curve Analysis.
