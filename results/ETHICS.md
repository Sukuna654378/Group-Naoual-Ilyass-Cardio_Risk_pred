# Ethics, Limitations & Impact Audit

This document outlines the ethical evaluation, limitations, and subgroup fairness analysis for the Framingham 10-Year Coronary Heart Disease (CHD) risk prediction study, following course laboratory guidelines.

| Question | Team Evaluation / Response |
| :--- | :--- |
| **1. Who is represented in the data, and who is not?** | **Represented:** 3,189 adult participants from the Framingham, MA cohort (USA).<br>**Not Represented:** Non-white ethnic populations (historically Caucasian cohort), individuals aged $<30$ or $>70$, and non-Western / North African populations. |
| **2. Which decision could rely on this model, and who suffers from errors?** | **Decision:** Primary care screening to initiate 10-year CHD preventive statin/lifestyle therapy.<br>**Impact:** *False Negatives* leave high-risk patients without preventive care (leading to cardiac events); *False Positives* subject low-risk patients to unnecessary medication side-effects and psychological stress. |
| **3. Can a variable act as a proxy for a sensitive feature?** | Yes. Age, systolic blood pressure (`sysBP`), and BMI proxy socioeconomic status. `sex_M` is explicitly included, dictating baseline risk and inducing severe recall gaps when using static decision thresholds. |
| **4. Do performances differ across subgroups (with sizes & 95% CIs)?** | Yes. At operating threshold $\tau^* = 0.54$, Elastic-Net LR achieves **0.709** [0.586, 0.833] Recall in Men ($N=287$, 55 events) vs. **0.450** [0.289, 0.606] in Women ($N=351$, 40 events), despite equal ROC-AUC (0.730 vs. 0.709). |
| **5. What prevents generalization?** | Reliance on a single historical geographic cohort (Framingham, MA; 1950s–1970s), single country (USA), single Kaggle mirror snapshot, and absence of modern biomarkers (e.g., HDL cholesterol, hs-CRP). |
| **6. What potential misuse or abuse is possible?** | Deploying raw uncalibrated probabilities from `class_weight='balanced'` models directly at the bedside. Because class weighting artificially inflates risk log-odds by $\approx +1.74$, using raw outputs would trigger systemic over-prescription of statins. |
| **7. Are data personal, and does the license permit this work?** | Data are fully de-identified secondary public data ($N=3,390$). Research/educational use is allowed under open mirror terms, but lack of an official BioLINCC DUA requires marking all results as provisional. |
