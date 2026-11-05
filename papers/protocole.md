# Experimental Protocol: 10-Year CHD Risk Prediction

## 1. Task Definition & Problem Formulation
- **Task Type**: Binary Classification (Supervised Learning).
- **Target Variable ($y$)**: `TenYearCHD` ($y \in \{0, 1\}$), where $1$ indicates incident Coronary Heart Disease within a 10-year prospective window.
- **Input Space ($X$)**: 14 non-invasive demographic, behavioral, and clinical markers + 4 domain-engineered interaction features ($d = 18$).
- **Analytical Cohort Size**: $N = 3,189$ post listwise deletion of low-percentage missingness (retained 94.07% of raw $N = 3,390$ cohort).
- **Class Imbalance**: Positive prevalence of 15.11% (482 events / 2,707 non-events, ~1:5.63 ratio).

## 2. Engineered Clinical Interaction Features
To capture complex non-linear physiological dynamics within linear models, four domain interaction features are computed prior to fold transformations:
1. **Pulse Pressure ($PP$)**: $\text{sysBP} - \text{diaBP}$ (Arterial stiffness).
2. **Mean Arterial Pressure ($MAP$)**: $\text{diaBP} + \frac{1}{3}PP$ (Perfusion pressure).
3. **Atherogenic Risk Index**: $\frac{\text{totChol}}{\text{BMI}}$ (Metabolic-lipid overload).
4. **Cumulative Smoking Load**: $\left(\frac{\text{cigsPerDay}}{20}\right) \times \max(\text{age} - 18, 0)$ (Estimated pack-years).

## 3. Candidate Model Architectures & Selection Rationale
1. **Baseline / Simple Model**: **ElasticNet Logistic Regression**
   - *Rationale*: Interpretable linear baseline with $L_1/L_2$ regularization to combat multicollinearity (e.g., `sysBP` vs. `diaBP`). Configured with cost-sensitive class weighting (`class_weight='balanced'`).
2. **Intermediate Model**: **Random Forest Classifier**
   - *Rationale*: Non-linear ensemble bagging classifier built on parallel decision trees to capture non-linear feature space splits. Configured with balanced class weights.
3. **Advanced Model**: **HistGradientBoosting Classifier / XGBoost**
   - *Rationale*: State-of-the-art histogram-based gradient boosted decision trees for tabular data, optimizing cross-entropy loss sequentially with histogram binning ($K=255$).

## 4. Anti-Leakage Cross-Validation Protocol
- **Splitting Strategy**: Stratified 5-Fold Cross-Validation (`StratifiedKFold(n_splits=5, shuffle=True, random_state=42)`).
- **Data Leakage Safeguard**: All preprocessing steps—including $k$-Nearest Neighbors ($k=5$) imputation for missing `glucose` values and `StandardScaler` feature normalization—are encapsulated strictly inside a `scikit-learn.pipeline.Pipeline`.
- **Rule**: Parameters ($\mu, \sigma, k\text{-NN}$ nearest points) are computed strictly on training folds and transitively applied to validation folds.

## 5. Hyperparameter Tuning Budget & Fair Comparison Strategy
- **Search Method**: 20 Randomized Search iterations (`RandomizedSearchCV`) per model architecture.
- **Evaluation Loop**: Executed strictly inside nested 5-fold cross-validation training partitions to avoid overfitting validation folds.
- **Tuning Targets**:
  - *ElasticNet*: Regularization strength $C \in [10^{-4}, 10^{1}]$, $L_1$ ratio $\alpha \in [0.0, 1.0]$.
  - *Random Forest*: `n_estimators` $\in [100, 300]$, `max_depth` $\in [3, 15]$, `min_samples_split` $\in [2, 10]$.
  - *HistGradientBoosting*: `learning_rate` $\in [0.01, 0.2]$, `max_iter` $\in [50, 200]$, `l2_regularization` $\in [10^{-3}, 10^{1}]$.

## 6. Evaluation Metrics & Decision Curve Thresholds
- **Primary Metrics**: Precision-Recall AUC (PR-AUC) and ROC-AUC (Accuracy is discarded due to the 85/15 class imbalance).
- **Secondary Metrics**: Sensitivity (Recall), Precision, $F_1$-score, and Brier Calibration Score.
- **Threshold Optimization**: Decision cutoffs ($\tau^*$) tuned explicitly on Precision-Recall curves to optimize clinical Sensitivity vs. False Positive Trade-offs (target $\tau^* \approx 0.5804$).

## 7. Computational Environment & Software Versions
- **Platform**: Google Colab (Linux / Python 3.10+).
- **Core Dependencies**: `scikit-learn==1.6.1`, `numpy==2.4.6`, `pandas==2.2.3`, `scipy==1.16.3`, `matplotlib==3.10.0`, `seaborn==0.13.2`, `shap==0.54.1`.
- **Reproducibility**: Deterministic seed set globally (`seed = 42`).
