# Exoplanet Habitability Prediction using Machine Learning

---

## Table of Content

- [Overview](#overview)
- [Results](#results)
- [Dataset](#dataset)
- [Methodology](#methodology)
- [Performance_Metrics](#performance_metrics)

---

## Overview

This project developes a machine learning pipeline to predict the habitability of exoplanets using stellar and planetary properties. The model was trained on data from [PHL Habitable worlds Catalog](https://phl.upr.edu/hwc/data).

### Key Achievements
- **99.84% Accuracy** on held_out test set
- **0.998 ROC_AUC** (near_perfect discirimination)
- **89.5% Recall** (finds most habitable planets)
- **94.4% Precision** (minimal false alarms)
- Only **3 errors** out of 1,844 test predictions

---

## Results

### Model Performance on the Test Set

| Metric | Score | Interpretation |
|--------|-------|----------------|
| **Accuracy** | 99.84% | 1,841/1844 correct predictions |
| **F1_Score** | 0.919 | Excellent balance |
| **Recall** | 89.47% | Finds 17/19 habitable planets |
| **Precision** | 94.44% | 17/18 predictions correct |
| **Specificity** | 99.95% | Rarely miscclassifies non-habitable |
| **ROC_AUC** | 0.998 | Near-perfect discrimination |
| **PR_AUC** | 0.9516 | Excellent on imbalanced data |

---

## Dataset

### Source
- **PHL Habitable worlds Catalog** (https://phl.upr.edu/hwc/data) 
- Includes list of about 70 potentially habitable worlds out of over five thousand known exoplanets.

### Features Used

#### Planetary Properties (6 features)
- 'P_MASS': Mass of the planet in Earth masses (Earth = 1.0 ME)
- 'P_RADIUS': Radius of the planet in Earth radii (Earth = 1.0 RE)
- 'P_ECCENTRICITY': Orbital eccentricity
- 'P_SMI_MAJOR_AXIS': Orbital semi-major axis
- 'P_GRAVITY': Surface gravity
- 'P_FLUX': Average stellar flux of the planet in Earth fluxes (Earth = 1.0 SE).

#### Stellar Properties (7 features)
- 'S_MASS': Stellar mass (Solar masses)
- 'S_RADIUS': Stellar radius (Solar radii)
- 'S_TEMPERATURE': Stellar effective temperature (K)
- 'S_DISTANCE': Distance from earth in light-years (ly)
- 'S_METALLICITY': Stellar metallicity [Fe/H]
- 'S_LUMINOSITY': Stellar luminosity (log(Solar))
- 'S_TIDAL_LOCK': Tidal locking indicator

### Target Variable
- 'HABITABILITY': Binary classification (0 = non-habitable, 1= potentially habitable)
- Potentially habitable worlds are identified using the Orbit-Size Criteria.

### Data Split
- **Train_set**: 67% (3755 planets from 2778 unique stars)
- **Test_set**: 33% (1844 planets from 1385 unique stars)
- **Validation**: 3-fold stratified group cross-validation

---

## Methodology

### 1.Feature Selection

Different planetary and stellar feature has been investigated from this dataset. Some features such as P_ESI (Earth similarity index), P_TENP_SURF (surface temperature of the planet) could be a shortcut feature derived from the habitability itself, hence they were removed from the elist of features. 
Other features with zero mutual information and features with high correlation have been removed as well. 
It is important to note some of the current features used for training the model might be derived or directly taken from different sources and papers. These features might not be available from direct observatons and measurements and might contain significant errors.
Finally, the 'S_Distance' (distance from earth) needs further investigation given that more distant stars are harder to detect which can cause selection bias for the trained model (future work).

### 2.Model Selection

Multiple algorithms were evaluated:
 - logostoc Regression
 - Random Forest
 - XGBoost (selected for best performance)

### 3.Hyperparameter tuning
Nested cross-validation approch:
 - Outer loop: 5-fold StratifiedGroupKFold (evaluation)
 - Inner loop: 3-fold StratifiedGroupKFold (hyperparameter tuning)
 - Search: Randomized search over 100 iterations per fold

#### Optimized Parameters:

{
    'n_estimators': 400,
    'max_depth': 5,
    'learning_rate': 0.1,
    'min_child_weight': 1,
    'subsample': 0.8,
    'colsample_bytree': 0.9,
    'gamma': 0,
    'reg_alpha': 0.5,
    'reg_lambda': 2,
    'scale_pos_weight': 4,
    'random_state': 42,
    'eval_metric': 'logloss',
    'tree_method': 'hist'
}

### 4.Validation Strategy

Grouped Cross-Validation to prevent data leakage:
 - All planets from the same star stay together
 - Test set contains completely unseen star systems
 - Ensures model generalizes to new discoveries

---

## Performance_Metrics

### Cross Validation Results


| model name | f1_score | roc_auc | recall | precision |
|------------|----------|---------|--------|-----------|
| Dummy | 0.0000 | 0.5000 | 0.0000 | 0.0000| 0.0136 |
| Logistic Regression | 0.1714 | 0.9375 | 0.8822 | 0.0964 |	0.3377 |
| RandomForestClassifier | 0.8233 |	0.9993 | 0.7346 | 0.9818 | 0.9659 |
| XGBClassifier | 0.8868 | 0.9975 | 0.8914 | 0.8967 | 0.9529 |



### Test Set Performance

Classification Report:
               precision    recall  f1-score   support

Not Habitable       1.00      1.00      1.00      1825
    Habitable       0.94      0.89      0.92        19

     accuracy                           1.00      1844
    macro avg       0.97      0.95      0.96      1844
 weighted avg       1.00      1.00      1.00      1844

