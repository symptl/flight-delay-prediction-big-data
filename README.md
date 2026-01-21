# Near-Real-Time Flight Delay Prediction at Scale

A PySpark-based machine learning pipeline for predicting U.S. domestic flight delays using 30M+ flight records and historical weather data. The project emphasizes feature engineering on distributed infrastructure through Databricks, with a focus on temporal integrity to avoid data leakage.

This repository contains the feature engineering pipeline and supporting notebooks. The project was completed as part of UC Berkeley's MIDS W261 (Machine Learning at Scale) course.

All file paths referenced within notebooks point to a now defunct workspace in Databricks and its associated storage, they cannot be reached.

Full project report is available in `notebooks/Final_Report.ipynb`.

---

## Key Results

- **Classification:** Random Forest achieved AUC 0.82, F1 0.76 on 2019 holdout data
- **Regression:** MLP achieved RMSE 43.81 minutes (vs. 48.88 baseline)
- **Top predictive features:** Preceding flight delays and turnaround time—both engineered in this pipeline

---

## My Contributions

This repository represents my individual contributions to a 5-person team project:

- **Feature Engineering Pipeline:** End-to-end PySpark pipeline processing 30M+ flights with strict temporal filtering (2-hour prediction window)
- **Preceding Flight Features:** Joined flights by tail number to extract delay propagation signals—these became the most predictive features in all models
- **Graph-Based Features:** Built monthly airport connectivity graphs and computed centrality metrics (betweenness, closeness, degree) for origin/destination airports
- **Feature Dictionary:** Documented all engineered features with definitions and leakage considerations

---

## Overview of Notebooks

### 1. Feature Engineering Pipeline

`notebooks/Data_Preprocessing_and_Feature_Engineering_Pipeline_v3.ipynb`

The main pipeline that processes raw flight and weather data into model-ready features. All operations respect the 2-hour prediction window constraint.

**Key Operations:**
- Temporal joins for preceding/following flight information by tail number
- Lagged weather variable extraction (2–6 hours prior to departure)
- Graph feature computation and time-slice joining to prevent leakage
- Holiday proximity flags
- Aircraft utilization metrics

**Infrastructure:**
- PySpark on Databricks
- Input: ~30M flight records (2015–2019) from DOT + NOAA weather data
- Output: Parquet files for downstream modeling

---

### 2. Graph-Based Features

`notebooks/Graph_Based_Features.ipynb`

Constructs directed graphs from airport flight patterns and computes node centrality metrics.

**Method:**
- Partition flights into monthly time slices
- Build weighted directed graphs (nodes = airports, edges = routes, weights = inverse flight count)
- Compute betweenness, closeness, and degree centrality for each airport
- Join features to the *following* month's flights to avoid leakage

**Features Generated:**
- `ORIGIN_BETWEENNESS`, `ORIGIN_CLOSENESS`, `ORIGIN_DEGREE`
- `DEST_BETWEENNESS`, `DEST_CLOSENESS`, `DEST_DEGREE`

---

### 3. Preceding Flight Delay Features

`notebooks/Previous_Flight_Delay_Features.ipynb`

Extracts delay propagation signals by joining each flight to prior flights on the same aircraft (tail number).

**Features Generated:**
- `PREV_DEP_DELAY` — departure delay of previous flight on tail
- `TIME_BETWEEN_ARR_AND_SCHEDULED_DEP` — turnaround time available
- `PREV2_DEP_DELAY`, `PREV2_ARRIVAL_DELAY` — two-flights-back signals
- `NEXT_FLIGHT_FLAG` — whether a subsequent flight exists (scheduling pressure)

**Why This Matters:**

These features consistently ranked as the top predictors across all model types. The intuition: delays propagate through an aircraft's daily schedule, and tight turnarounds amplify risk.

> **Note:** `Previous_Flight_Delay_Features.html` is exported from Databricks with cell outputs preserved. Large notebooks with visualizations can only be exported as HTML, conversion to .ipynb was avoided to retain execution results.

---

### 4. Feature Dictionary

`docs/feature_dictionary.md`

Documentation of all categorical, numerical, graph, and weather features with definitions and temporal constraints.

---

### 5. Final Report

`notebooks/Final_Report.ipynb`

Complete project writeup including EDA, methodology, model comparisons, and results. Covers classification (Logistic Regression, Random Forest, CatBoost, XGBoost, MLP) and regression approaches.

---

## Tech Stack

- **Processing:** PySpark on Databricks
- **Data:** US DOT On-Time Performance (~30M flights), NOAA weather records
- **Models:** Spark ML, XGBoost, CatBoost, TensorFlow/Keras
- **Validation:** Time-series cross-validation with 2019 holdout

---

## Data Sources

- [US DOT Bureau of Transportation Statistics](https://www.transtats.bts.gov/) — On-Time Performance data
- [NOAA](https://www.ncdc.noaa.gov/) — Historical weather observations

---

## Team

This was a group project for UC Berkeley MIDS W261. Team members: Alex Lewis, Missael Isaac Vasquez, Rohan Krishnamurthi, Matthew Paterno, and Shyam Patel.

The notebooks in this repository represent my individual contributions to the project.
