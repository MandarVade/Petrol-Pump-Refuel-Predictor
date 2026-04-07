# PETROPREDICT — COMPLETE PPT CONTENT GUIDE
## All content sourced from project notebooks (01–07), model_metrics.json, model_features.json, and data files

---
## SLIDE 1 — TITLE SLIDE

Project Name: PetroPredict
Tagline: "AI-powered fuel demand prediction for smart inventory management"
Impact Line: "Reducing fuel shortages and overstock using Machine Learning"
Sub-details: Your Name / Team | College / Event Name | Date

---
## SLIDE 2 — PROBLEM STATEMENT

Headline: Petrol Pumps Operate Blindly

The Core Problem:
Petrol pump managers rely entirely on manual estimation to decide when to order fuel. This leads to:

| Problem | Real-World Impact |
|---|---|
| Fuel Shortages | Customers turn away, revenue lost instantly |
| Overstocking | Capital locked up in excess inventory |
| Revenue Loss | Both extremes cost money every day |
| Reactive Decisions | Refills happen AFTER a crisis, not before |

Key Statement: "Decisions are reactive, not predictive."
Impact Note: Even a 1-day stock mismatch per pump per week = massive cumulative revenue loss across a network.

---
## SLIDE 3 — SOLUTION OVERVIEW

Headline: PetroPredict — Your Intelligent Refill Assistant

PetroPredict uses historical sales data and stock levels to predict exactly which day a petrol pump will need its next refill — before it runs out.

Simple Flow:
Input Data (Stock, Sales, Date) → ML Model (Random Forest) → Prediction (Exact Day + Stock Level) → Refill Decision (Auto Recommendation)

Key Message: "We automate refill decisions using data — turning reactive firefighting into proactive planning."

What It Uses:
- Historical daily fuel sales records
- Opening and closing stock levels
- Day-of-week trends, seasonal patterns, rolling averages

---
## SLIDE 4 — SYSTEM ARCHITECTURE

Headline: End-to-End ML Pipeline

INPUT LAYER
  Raw Data: 806 days of operational records
  Columns: Date, Opening Stock, MS Sold, HSD1/HSD2/HSD3 Sold, Total Sold, Closing Stock, Cash/Online/Card, Dip
       |
       v
PROCESSING LAYER
  Notebook 01: Data Cleaning, Feature Engineering (25+ columns created)
  Notebook 02: Feature Selection (Pearson + RF Importance → 7 final features)
  Notebook 04: Outlier Detection (IQR + Z-Score + Isolation Forest → Winsorizing)
       |
       v
TRAINING LAYER
  Notebook 05: SMOTE Balancing (62.4%/37.6% → 50%/50%)
  Notebook 06: Random Forest (100 trees, GridSearchCV 5-fold CV)
  Train: 1,006 rows (SMOTE-balanced) | Test: 162 rows | Features: 7
       |
       v
PREDICTION ENGINE
  Notebook 07: Day-by-day stock simulation using historical day-of-week consumption averages
  ML Model confirms exact refill day with probability score
       |
       v
OUTPUT LAYER
  Exact Refill Date | Expected Closing Stock | ML Confidence %

---
## SLIDE 5 — DATASET AND FEATURES

Headline: Operational Data — Feature Engineering Pipeline

DATASET OVERVIEW (from Notebook 01):
| Property | Value |
|---|---|
| Total Records | 806 days |
| Class Split | 503 No Refill (62.4%) / 303 Refill (37.6%) |
| Raw Columns | Date, Day, Opening Stock, MS/HSD Sold, Total Sold, Closing Stock, Cash/Online/Card, Dip, Refill Required |

FEATURE ENGINEERING (Notebooks 01 and 02):
Time Features: Year, Month, DayOfWeek, Quarter, WeekOfYear, DayOfYear
Binary Flags: Is_Weekend, Is_Festival_Month, Is_Monsoon_Month, Is_Summer_Month
Lag Features: Prev_Closing, Prev_Total_Sold
Rolling Averages: Rolling_7d_Sales, Rolling_3d_Sales
Computed: Days_Since_Refill, Stock_Ratio

FINAL 7 FEATURES SELECTED (from model_features.json):
| Feature | Role |
|---|---|
| Days_Since_Refill | Key time-based predictor — how many days since last refill |
| Opening_Stock | Current fuel level at day start |
| Prev_Closing | Previous day's closing stock (lag feature) |
| Cash | Cash payment volume — proxy for daily demand |
| HSD2_Sold | High-speed diesel type 2 sales |
| HSD1_Sold | High-speed diesel type 1 sales |
| Total_Sold | Total fuel sold in a day |

Key Insight: "Feature engineering improves prediction accuracy — we reduced 25+ columns to only 7 powerful predictors."
Note: Leaky features (Closing_Stock, Stock_Ratio, Dip) were removed to ensure real-world prediction validity.

---
## SLIDE 6 — WORKING / PIPELINE

Headline: Step-by-Step ML Pipeline

Step 1: DATA COLLECTION
  - 806 days of pump operational data

Step 2: DATA CLEANING (Notebook 01)
  - Handle missing values, parse dates, fix data types
  - Output: clean_data.csv (806 rows, 25+ columns)

Step 3: FEATURE ENGINEERING (Notebook 01)
  - Extract time features: Year, Month, DayOfWeek, Quarter, WeekOfYear
  - Create lag and rolling average features
  - Encode target: Refill_Required → Target (0/1)

Step 4: FEATURE SELECTION (Notebook 02)
  - Pearson Correlation + Random Forest Importance scores
  - Multicollinearity check — drop features correlated above 0.90
  - Remove leaky features (Closing_Stock, Dip, Stock_Ratio)
  - Output: 7 final features — selected_features.csv

Step 5: OUTLIER DETECTION (Notebook 04)
  - IQR Method + Z-Score + Isolation Forest (3 methods)
  - Decision: Winsorizing (cap at 1st-99th percentile, no rows deleted)

Step 6: CLASS IMBALANCE HANDLING (Notebook 05)
  - Problem: 503 No Refill (62.4%) vs 303 Refill (37.6%)
  - Methods tested: Baseline, Class Weights, SMOTE, Undersampling, SMOTE+Undersampling
  - All methods achieved F1 = 1.0000 on this dataset
  - SMOTE selected for robustness: 806 → 1,006 balanced training rows
  - Output: balanced_data.csv

Step 7: MODEL TRAINING (Notebook 06)
  - Train 4 models on SMOTE-balanced data (1,006 rows)
  - Test all on original unbalanced hold-out set (162 rows)
  - GridSearchCV: 5-fold CV, 24 candidates, 120 total fits
  - Best model saved as final_model.pkl

Step 8: PREDICTION (Notebook 07)
  - Load last row from cleaned data as starting point
  - Simulate stock drawdown using historical day-of-week averages
  - Apply seasonal adjustments (monsoon ×0.88, festival ×1.15, summer ×1.08)
  - Stop when closing stock drops below 2,000L
  - ML model confirms with probability score

OUTPUT: "Next Refill Needed on [DATE]" + ML Confidence %

---
## SLIDE 7 — MACHINE LEARNING MODEL

Headline: Why Random Forest? Because Fuel Demand Is Non-Linear

MODELS COMPARED (from Notebook 06 actual output):
| Model | Accuracy | F1 Score | ROC-AUC |
|---|---|---|---|
| Logistic Regression | 0.9938 | 0.9917 | 1.0000 |
| Decision Tree | 1.0000 | 1.0000 | 1.0000 |
| Random Forest (SELECTED) | 1.0000 | 1.0000 | 1.0000 |
| XGBoost | 1.0000 | 1.0000 | 1.0000 |

WHY RANDOM FOREST:
"Random Forest handles non-linear demand patterns — weekend spikes, festival surges, monsoon dips all require ensemble decision-making power."
- 100 independent decision trees — majority vote reduces errors
- Robust to noisy real-world data
- Auto-generates feature importance rankings
- Native support for class_weight='balanced'

HYPERPARAMETER TUNING (GridSearchCV — from Notebook 06):
  "Fitting 5 folds for each of 24 candidates, totalling 120 fits"
  Best params: {class_weight: balanced, max_depth: None, min_samples_split: 2, n_estimators: 100}
  Best CV F1: 0.9990

TRAINING DETAILS:
- Training Rows (post-SMOTE): 1,006
- Test Rows (original distribution): 162
- Features: ['Days_Since_Refill', 'Opening_Stock', 'Prev_Closing', 'Cash', 'HSD2_Sold', 'HSD1_Sold', 'Total_Sold']

Charts to include: viz_model_comparison.png | viz_roc_curves.png

---
## SLIDE 8 — RESULTS AND OUTPUT

Headline: Perfect Predictions on Real Test Data

FINAL MODEL PERFORMANCE (from model_metrics.json):
  accuracy : 1.0
  f1_score : 1.0
  roc_auc  : 1.0

CLASSIFICATION REPORT (from Notebook 06 actual output):
| Class | Precision | Recall | F1-Score | Support |
|---|---|---|---|---|
| No Refill | 1.00 | 1.00 | 1.00 | 101 |
| Refill | 1.00 | 1.00 | 1.00 | 61 |
| Overall | | | 1.00 | 162 |

KEY ACHIEVEMENTS:
| Metric | Score |
|---|---|
| Accuracy | 100% |
| F1 Score | 100% |
| ROC-AUC | 100% |
| False Alarms (FP) | ZERO |
| Missed Refills (FN) | ZERO |

"Our model achieved 100% prediction accuracy — zero missed refills, zero false alarms on 162 real test records."

CONFUSION MATRIX (from Notebook 06):
- TN = 101 (correctly identified as No Refill)
- TP = 61 (correctly identified as Refill Needed)
- FP = 0 (no unnecessary orders)
- FN = 0 (no missed critical refills)

Charts to include: viz_confusion_matrix.png | viz_roc_curves.png | viz_model_comparison.png

---
## SLIDE 9 — DEMO / PREDICTION OUTPUT

Headline: See It in Action — Real Prediction Example

HOW THE PREDICTION WORKS (Notebook 07):
1. Read last entry from clean_data_no_outliers.csv as starting point
2. Tank Capacity = 12,000L | Refill Threshold = 2,000L
3. If last day was refill → opening stock = 12,000L, else use last closing stock
4. Calculate real average daily consumption per day-of-week from historical data
5. Apply seasonal adjustments (monsoon ×0.88, festival ×1.15, summer ×1.08)
6. Simulate stock decreasing day by day
7. Stop when closing stock drops below 2,000L — that is the refill day
8. ML model confirms with a probability score

ACTUAL PREDICTION OUTPUT (from Notebook 07):
=======================================================
  NEXT REFILL PREDICTION
  Current stock : 7687 L
  From date     : 17-03-2026
=======================================================

Day-by-day simulation:
      Date       Day  Opening_Stock  Sold  Closing_Stock  ML_Prob  ML_Pred
17-03-2026   Tuesday           7687  4810           2877     0.80        1
18-03-2026 Wednesday           2877  2877              0     0.51        1

  Next refill needed on:
  Date          : 18-03-2026  (Wednesday)
  Days from now : 2 day(s)
  Opening stock : 2877 L
  Expected sold : 2877 L
  Closing stock : 0 L  (below 2000L threshold)
  ML confidence : 51%
=======================================================

BUSINESS DECISION FLOW:
Manager checks current stock → System simulates daily drawdown → PetroPredict: "Order refill by [DATE]" → Pump avoids going dry

REFILL THRESHOLD RULE:
Closing Stock < 2,000L → Refill Required = YES
Closing Stock ≥ 2,000L → Refill Required = NO

Chart to include: viz_next_refill_prediction.png

---
## SLIDE 10 — ADVANTAGES

Headline: Why PetroPredict Wins

| Advantage | Detail |
|---|---|
| Reduces Fuel Shortage | Predicts exact refill day — never run dry |
| Saves Cost | Prevents overstocking — no capital locked in excess inventory |
| Automates Decisions | No guesswork — data drives every order |
| Seasonal Awareness | Monsoon dips, festival surges, summer increases all captured |
| Confidence Score | ML probability with every prediction |
| No New Hardware | Works with existing daily operational records |
| Scalable | Same architecture adaptable for any pump or region |
| Full Pipeline | 7 notebooks — end-to-end reproducible workflow |

---
## SLIDE 11 — LIMITATIONS

Headline: Honest Assessment — What We Know and Don't

"A strong AI system knows its own boundaries."

| Limitation | Explanation |
|---|---|
| Data Quality | Model depends on accurate daily entries — garbage in, garbage out |
| External Events | Strikes, disasters, sudden price hikes not accounted for |
| Single Pump Scope | Calibrated for one pump — multi-site needs per-site retraining |
| Fixed Threshold | 2,000L hardcoded — different pump capacities need adjustment |
| Perfect Metrics | 100% scores on test set may indicate overfitting — real-world validation needed |

---
## SLIDE 12 — FUTURE SCOPE

Headline: PetroPredict 2.0 — What Is Next

| Enhancement | Description |
|---|---|
| Real-Time IoT Integration | Tank dip sensors → live stock readings → instant predictions |
| Weather-Based Prediction | Weather API integration — rain/heat impacts demand |
| Mobile App | Daily manager push notification: "Order fuel by Thursday" |
| Multi-Fuel Optimization | Separate models for Petrol (MS), HSD1, HSD2, HSD3 |
| Multi-Pump Dashboard | Chain owner sees all sites on one web dashboard |
| Price Signal Integration | Alert before price hikes — order ahead strategically |
| AutoML Retraining | Model auto-retrains monthly as new data arrives |
| Volume Forecasting | Predict exact liters needed per refill, not just Yes/No |
| Real-World Data Validation | Deploy with actual pump data to validate beyond current dataset |

---
## SLIDE 13 — CONCLUSION

Headline: From Reactive to Intelligent

WHAT WE BUILT:
| Component | Achievement |
|---|---|
| Dataset | 806 days of operational data |
| Pipeline | 7 notebooks, fully automated end-to-end |
| Features | 25+ engineered → 7 selected by importance |
| Model | Random Forest, 100 trees, GridSearchCV-tuned |
| Accuracy | 100% accuracy, F1, and ROC-AUC on 162 test records |
| Output | Exact refill date + ML confidence percentage |

THE TRANSFORMATION:
| Before PetroPredict | After PetroPredict |
|---|---|
| Manual estimation | AI-driven prediction |
| Reactive refills | Proactive planning |
| Stock mismatches | Optimized inventory |
| Revenue losses | Protected earnings |
| Daily guesswork | Data-backed confident decisions |

CLOSING STATEMENT (BOLD, LARGE FONT):
"PetroPredict transforms fuel management from reactive to intelligent decision-making — one refill at a time."

---
## APPENDIX — QUICK STATS FOR JUDGES

| Metric | Value |
|---|---|
| Dataset | 806 rows × 13 raw columns |
| Class split | 503 No Refill (62.4%) / 303 Refill (37.6%) |
| Notebooks | 7 (full end-to-end pipeline) |
| Models trained | 4 (Logistic Regression, Decision Tree, Random Forest, XGBoost) |
| Final model | Random Forest (100 trees) |
| Training set | 1,006 rows (post-SMOTE balanced) |
| Test set | 162 rows (original unbalanced distribution) |
| Accuracy | 100% (1.0000) |
| F1 Score | 100% (1.0000) |
| ROC-AUC | 100% (1.0000) |
| Confusion matrix | TN=101, FP=0, FN=0, TP=61 |
| Model features | 7 (Days_Since_Refill, Opening_Stock, Prev_Closing, Cash, HSD2_Sold, HSD1_Sold, Total_Sold) |
| Best hyperparams | class_weight=balanced, max_depth=None, min_samples_split=2, n_estimators=100 |
| GridSearchCV | 5-fold CV, 24 candidates, 120 total fits, best CV F1=0.9990 |
| Tank capacity | 12,000L |
| Refill threshold | 2,000L (Closing Stock) |
| Imbalance fix | SMOTE → 1,006 balanced training rows from 806 original |
| Outlier method | Winsorizing (IQR + Z-Score + Isolation Forest, 3-method approach) |
| Tech stack | Python, Scikit-learn, XGBoost, Pandas, NumPy, SMOTE (imbalanced-learn), Matplotlib, Seaborn |
