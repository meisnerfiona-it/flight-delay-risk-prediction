# ✈️ Flight Delay Risk Prediction & Operational Decision System

> Transforming historical flight operations data into a proactive delay management framework — from raw data to business-ready risk intelligence.

---

## 📌 Project Overview

Flight delays cost the U.S. aviation industry an estimated **$28 billion annually** (FAA, Bureau of Transportation Statistics). The vast majority of delay management is reactive — airlines respond to disruption after it has already cascaded through their network.

This project builds a **predictive decision-support system** that identifies high-risk flights *before* they depart, using only pre-flight operational data. The output is not just a model — it is an operational framework that translates predictions into concrete scheduling, resourcing, and passenger communication decisions.

---

## 🎯 Business Problem

| Problem | Impact |
|---|---|
| Delays identified too late for proactive intervention | Cascading network disruption |
| Reactive resource allocation | Higher operational costs |
| Passengers notified only after delays occur | Poor customer experience |
| No systematic early-warning framework | Missed cost mitigation opportunities |

**Goal:** Move delay management from reactive to predictive using structured machine learning.

---

## 🔬 Methodology

### Data Pipeline

**Source:** U.S. Bureau of Transportation Statistics — 2015 domestic flight operations  
**Scale:** 5.8 million flight records across 14 airlines and 322 airports

```
Raw Data (5.8M rows)
    ↓
Missing value treatment (median fill / zero fill for delay causes)
    ↓
Column removal (post-event leakage columns excluded)
    ↓
Target creation: DELAY_RISK = 1 if DEPARTURE_DELAY > 15 minutes
    ↓
Feature engineering (6 operational features)
    ↓
Encoding + stratified train/test split (80/20)
    ↓
Model training → evaluation → threshold optimisation
    ↓
Business decision framework
```

### Feature Engineering

| Feature | Logic | Business Rationale |
|---|---|---|
| `IS_PEAK_HOUR` | Scheduled departure 07:00–10:00 | Morning rush = peak airport congestion |
| `IS_WEEKEND` | Day of week in [6, 7] | Leisure demand surge affects operations |
| `IS_LATE_NIGHT` | Scheduled departure ≥ 21:00 | Accumulated delays propagate to late flights |
| `IS_LONG_HAUL` | Distance > 1,500 miles | Greater operational complexity and rotation dependency |
| `IS_MAJOR_HUB` | Origin airport in top 6 US hubs | Hub congestion amplifies delay risk |
| `EARLY_MORNING_FLIGHT` | Scheduled departure ≤ 06:00 | First departures of day — lowest delay accumulation |
| `SEASON` | Month grouped into 4 seasons | Captures seasonal demand and weather patterns |
| `DEPARTURE_HOUR` | Scheduled departure // 100 | Non-linear delay risk pattern across the day |

### ⚠️ Data Leakage — Critical Note

The following columns are **excluded from all model features**. They are recorded only *after* a delay occurs and would constitute data leakage if used as predictors:

`AIRLINE_DELAY` · `WEATHER_DELAY` · `LATE_AIRCRAFT_DELAY` · `AIR_SYSTEM_DELAY` · `SECURITY_DELAY`

These columns are used in EDA for diagnostic insight only.

---

## 🤖 Modelling

### Models Trained

| Model | Purpose | Scaling Required |
|---|---|---|
| Logistic Regression | Baseline / performance floor | Yes (StandardScaler) |
| Random Forest | Primary model | No |

### Model Selection — Random Forest

Random Forest was selected as the production candidate for the following reasons:

- Higher recall on the delayed class — operationally more valuable than raw accuracy
- Handles class imbalance natively via `class_weight='balanced'`
- No feature scaling required
- Produces interpretable feature importances
- Robust to outliers and non-linear relationships

### Class Imbalance Strategy

The dataset is moderately imbalanced (~80% on-time, ~20% delayed). Strategy applied:
- `class_weight='balanced'` on both models
- Evaluation prioritises **recall** and **F1-score** over accuracy
- Threshold tuned to 0.40 (below default 0.50) to improve recall on delayed class

## ⚙️ Operational Decision Framework

### Risk Segmentation

| Segment | Probability | Action |
|---|---|---|
| 🟢 Low Risk | < 30% | Standard operating procedure |
| 🟡 Medium Risk | 30–60% | Place on monitoring watch list |
| 🔴 High Risk | > 60% | Trigger pre-emptive operational response |

### Threshold Analysis

Rather than using the default 0.50 threshold, the model uses **0.40** to prioritise recall — capturing more true delays at the cost of more false alarms. This trade-off is intentional:

> In aviation operations, **missing a true delay** (false negative) is more operationally costly than **an unnecessary alert** (false positive).

### High-Risk Response Playbook

| Trigger | Action | Timing |
|---|---|---|
| P(delay) > 0.60 | Ground crew pre-positioning | T−90 min |
| P(delay) > 0.60 | Proactive passenger notification | T−60 min |
| P(delay) > 0.60 | Connection risk review | T−60 min |
| P(delay) > 0.60 | Catering and fuelling schedule review | T−45 min |
| P(delay) 0.30–0.60 | Monitoring watch list | T−120 min |

---

## 📊 Key EDA Findings

- **Late aircraft delays and airline operational delays** are the largest contributors to total delay minutes
- **Peak hour departures** (07:00–10:00) carry elevated delay risk due to congestion
- **Early morning flights** (≤ 06:00) are the most punctual — lowest delay accumulation
- **Delay risk by airline** varies meaningfully — carrier identity is a significant predictor
- **Long-haul routes** show higher delay risk due to rotation dependency and operational complexity
- All engineered features confirmed statistically significant via chi-square tests (p < 0.001)

---

## 🛠️ Tech Stack

| Category | Tools |
|---|---|
| Language | Python 3.13 |
| Data manipulation | Pandas, NumPy |
| Visualisation | Matplotlib, Seaborn |
| Machine learning | Scikit-learn |
| Statistical testing | SciPy |
| Environment | Jupyter Notebook / Anaconda |


## 💡 Business Value

This project demonstrates that a structured, data-driven delay risk framework is technically feasible and operationally deployable using existing airline data infrastructure.

The primary value is not perfect prediction. The value is **earlier, better-informed decision-making** — giving operations teams more time to act before disruption cascades through the network.

---

## 📋 Limitations & Next Steps

| Limitation | Proposed Solution |
|---|---|
| Trained on 2015 data — patterns may have shifted | Annual retraining, quarterly validation |
| No live weather data | Integrate NOAA / weather API feed |
| No aircraft rotation tracking | Add tail number rotation chain as future feature |
| Single-year training data | Expand to multi-year dataset for seasonal robustness |
