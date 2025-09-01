# Mining Silica Quality KPIs — Azure Data Pipeline

## Problem
Convert raw flotation plant sensor data into daily quality KPIs so engineers can spot instability (silica > 2%) early.

## 📊 Dataset
- **Source:** [Kaggle — Quality Prediction in a Mining Process](https://www.kaggle.com/datasets/edumagalhaes/quality-prediction-in-a-mining-process)  
- **Size:** ~737,000 rows of time-series flotation plant readings (2017-03 → 2017-09)  
- **Features:** iron/silica feed %, pulp flow, pulp density, reagent flows, flotation column levels, concentrate grades  

## Architecture
Azure Data Lake Gen2 (Bronze) → Databricks (Silver cleaning) → Databricks (Gold KPIs)  
Orchestrated by Azure Data Factory (scheduled, retries, alerts). Stored as Delta tables.

## What I built (highlights)
- **Bronze:** raw CSV in ADLS, normalized headers.
- **Silver:** parsed timestamps; cast/cleaned numeric fields; domain checks (pH 0–14, no negatives).
- **Gold:** daily KPIs — avg/max/min silica, **% readings above 2%** threshold, record counts.
- **Monitoring:** Delta log with row counts, nulls, range violations, time coverage.
- **ADF:** 2-step pipeline (Silver → Gold) with retries, timeout, daily schedule, failure alert.

## Results
- `silver_flotation`: **737,453 rows**  
- `gold_daily_kpis`: **172 daily records**  
- Example insights:
  - Mar 14: avg silica ~3.26%, **~80%** of readings above 2% (risk day)
  - Mar 15: avg ~1.55%, **~12.5%** above 2% (stable day)
- Example KPI rows:
  | day        | avg\_silica | max\_silica | min\_silica | pct\_above\_threshold | records |
| ---------- | ----------- | ----------- | ----------- | --------------------- | ------- |
| 2017-03-10 | 1.74        | 3.57        | 1.09        | 21.8%                 | 4,134   |
| 2017-03-14 | 3.26        | 5.30        | 1.01        | 79.1%                 | 4,320   |
| 2017-03-15 | 1.55        | 3.86        | 1.01        | 12.5%                 | 4,320   |


## Visuals
- Line: daily average silica vs 2% threshold  
- Bar: % of readings above 2% per day

## Notebooks

- **bronze_ingest.ipynb**  
  Connects to Azure Data Lake using SAS token, ingests ~737k flotation sensor rows, inspects schema, renames problematic columns, and saves a Bronze Delta table.  
  *Includes early exploratory plots to confirm data distribution before moving to Silver.*

- **silver_build.ipynb**  
  Cleans and casts numeric fields, validates domain constraints (pH, negatives), and saves to Silver Delta.

- **gold_build.ipynb**  
  Aggregates daily KPIs (avg/max/min silica, % above 2% threshold), writes Gold Delta table, and supports dashboards.


## Tech
Azure Databricks, ADLS Gen2, Delta Lake, ADF; Python (PySpark).

## Why this matters
This mirrors historian-to-cloud workflows: reliable ingestion, quality validation, daily KPIs, and monitoring—so engineers can react before quality slips.