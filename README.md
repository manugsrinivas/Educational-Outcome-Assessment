# Impact of School Resources and Demographics on District Test Score Performance

*A regional, classification-based analysis of U.S. K–12 districts using the School Finance Indicators Database (District Cost Database).*

## Overview
We study how **school resources** (e.g., funding adequacy) and **district demographics** (e.g., poverty, enrollment, racial/ethnic composition, ELL/IEP shares) relate to a district’s likelihood of performing **above the national average** on standardized tests. We cast the problem as **binary classification**—Above (1) vs. Below (0)—and run parallel models for **Northeast, Midwest, South, and West** to respect regional heterogeneity.

## Data
- **Source:** School Finance Indicators Database (SFID) – District Cost Database (DCD), pooled cross-section for ~**2009–2021**.  
- **Sample:** **94,211** district-level rows after cleaning.  
- **Outcome:** `outcomegap` (SDs from U.S. mean), binarized to `class_outcome` (1 = above national average).  
- **Predictors:** funding adequacy/gap (difference between actual and predicted “adequate” spend), poverty rate, enrollment, racial/ethnic composition, ELL, IEP, region/state indicators.  
- **Coverage gaps:** Excludes **VT, HI, AK**; some districts lack the outcome-gap measure; instruction-level variables (e.g., teacher experience, curriculum) are not included.

## Methods
We implement and compare **Logistic Regression (with stepwise selection), LDA, QDA, Classification Trees, Random Forests, Gradient Boosting, SVM, and k-NN**.  
Validation uses **k-fold cross-validation**; we **tune the classification threshold** on a 0.30–0.70 grid. For Random Forests, we check **OOB learning curves** to assess generalization.

**Primary target:** `class_outcome` = 1 if a district’s average score exceeds the national average; else 0. Logistic is the interpretable baseline; tree-based and kernel methods capture non-linearities and interactions.

## How to reproduce
1. **R 4.2+ recommended.**  
2. Install typical packages used in the Rmds: `tidyverse`, `caret`, `randomForest`, `rpart`, `rpart.plot`, `gbm`, `e1071`, `MASS`, `kknn`, `pROC`, `ROCR`, `car`, `glmnet`.  
3. Knit each regional notebook in `analysis/` (Northeast, Midwest, South, West). Each Rmd:
   - Loads `data/DistrictData.csv`, filters to its region  
   - Trains Logistic + ML alternatives  
   - Tunes the probability cutoff via CV  
   - Saves metrics to `results/` and figures to `figures/`  
4. Optional: use `renv::init()` and commit `renv.lock` for reproducibility.

## Results — high level
- **Northeast:** **Random Forest** ~**90.8% accuracy**. Socioeconomic variables (poverty, racial/ethnic composition) drive predictive power; **funding gap** is prominent on split-purity metrics.  
- **South:** **Random Forest** ~**85% accuracy**; important predictors include **state effects**, **funding gap**, **poverty**, and **Black/Asian composition**. In the logit, poverty and special-education share (and Black dummy) skew negative; Asian share positive.  
- **Midwest:** **RF** Accuracy **0.807** (CI ≈ [0.7995, 0.8143]); **Sensitivity 0.724**, **Specificity 0.855**, **AUC 0.886**. **Funding gap** and **poverty** dominate feature importance.  
- **West:** **Logit** ≈ **80.14% accuracy**, **AUC ≈ 0.8825**; **threshold = 0.45** improves sensitivity/specificity. **RF** improves further (**87.84% accuracy**, **AUC 0.9431**, **OOB error 11.91%**).  
- **Generalization:** RF OOB error converges to CV error around **11–12%**, suggesting limited overfitting and solid out-of-sample performance.

## Interpreting the models
- **Logistic regression (policy-friendly):** coefficients indicate how a **one-unit change** (e.g., in poverty) shifts **log-odds** of being above the national average—useful for direction and magnitude.  
- **Random Forest importance:**  
  - **Mean Decrease Accuracy (MDA):** loss in accuracy when a feature is permuted → contribution to prediction.  
  - **Mean Decrease Gini (MDG):** improvement in split purity.  
  Features rated high on **both** are “**gold-standard**” predictors.  
- **Threshold tuning:** Moving away from the default 0.50 cutoff (e.g., **0.45** in the West) can materially improve sensitivity/specificity trade-offs.

## Key takeaways
- **Resources + demographics both matter.** Funding adequacy (gap) and socioeconomic context (especially **poverty**) consistently explain which districts exceed the national average.  
- **Modeling choice matters.** Non-linear/ensemble methods (especially **Random Forests**) generally outperform linear baselines across regions.  
- **Policy is regional.** Importance rankings and optimal thresholds vary by region; one-size-fits-all prescriptions are unlikely to be efficient.

## Assumptions & limitations
- **Exogeneity caveat:** Spending and demographics are treated as exogenous to current outcomes; districts may **raise spending in response to past low performance** (reverse causality).  
- **Cross-district comparability:** We rely on standardized outcome gaps; **state testing differences** may still introduce residual heterogeneity.  
- **Binary outcome trade-off:** Converting a continuous gap into Above/Below eases decisions but **loses information** near the mean.  
- **Logit assumptions:** Independence and limited multicollinearity; robustness is checked with non-parametric models (RF, GBM, SVM).  
- **Pooled cross-section:** Years are pooled and state×year interactions are not modeled; **unobserved state-year trends** may remain.  
- **Data scope:** 2009–2021 cross-section; excludes **VT/HI/AK**; some missing `outcomegap`; lack of instruction-level variables may induce **omitted-variable bias**.

## Policy implications
- **Targeted supports** for high-poverty districts and those with large **funding gaps** are likely to improve outcomes.  
- Because drivers differ across regions, **state/region-specific** strategies (not national uniform rules) are most promising.

## License
Add an OSI license (e.g., MIT) at the repository root under `LICENSE`.
