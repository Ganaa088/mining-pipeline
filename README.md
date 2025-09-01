# Mining Silica Quality KPIs — Azure Data Pipeline

## 📌 Problem
Flotation plants generate thousands of sensor readings every hour. Engineers need a reliable way to turn this raw historian data into **daily KPIs** that highlight ore quality risks (e.g., high silica levels).  

This project demonstrates how to build an **end-to-end Azure data pipeline** that ingests, cleans, validates, and aggregates mining sensor data into business-ready insights.

---

## 📊 Dataset
- **Source:** [Kaggle — Quality Prediction in a Mining Process](https://www.kaggle.com/datasets/edumagalhaes/quality-prediction-in-a-mining-process)  
- **Size:** ~737,000 time-series flotation plant readings (Mar–Sep 2017)  
- **Features:** % iron/silica feed, pulp flow, pulp density, reagent flows, flotation column air/levels, concentrate grades  

---

## ⚙️ Architecture
Azure Data Lake Gen2 (**Bronze**) → Databricks (**Silver cleaning**) → Databricks (**Gold KPIs**)  
Orchestrated with **Azure Data Factory** (scheduled, retries, alerts). Stored in **Delta Lake**.

---

## 🔨 What I built
- **Bronze:** Raw CSVs stored in ADLS, normalized headers.  
- **Silver:** Parsed timestamps, casted numeric fields, and applied domain checks (pH 0–14, no negatives).  
- **Gold:** Aggregated daily KPIs — avg/max/min silica, % readings above 2% threshold, record counts.  
- **Monitoring:** Delta log tracking row counts, nulls, range violations, and time coverage.  
- **ADF:** 2-step pipeline (Silver → Gold) with retries, timeouts, daily schedule, and failure alerts.  

---

## 📈 Results
- `silver_flotation`: **737,453 rows** (cleaned sensor data)  
- `gold_daily_kpis`: **172 daily records** (daily KPI aggregates)  

**Example insights:**  
- *Mar 14:* avg silica ~3.26%, ~80% of readings above 2% (risk day)  
- *Mar 15:* avg silica ~1.55%, ~12.5% of readings above 2% (stable day)  

---

## 📊 Visuals
- **Line chart:** Daily average silica vs 2% threshold
 - ![alt text](https://github.com/Ganaa088/mining-pipeline/blob/main/visuals/silica_line_chart.png)

- **Bar chart:** % of readings above 2% silica per day
  - ![alt text](https://github.com/Ganaa088/mining-pipeline/blob/main/visuals/silica_bar_chart.png)
 

*(see `/visuals/` folder for images)*

---

## 📒 Notebooks
- **bronze_ingest.ipynb**  
  Connects to ADLS with SAS token, ingests ~737k flotation sensor rows, inspects schema, renames problematic columns, and saves Bronze Delta table.  
  *Includes early exploratory plots to confirm distributions.*  

- **silver_build.ipynb**  
  Cleans and casts numeric fields, validates domain constraints (pH range, negatives), and saves Silver Delta.  

- **gold_build.ipynb**  
  Aggregates daily KPIs (avg/max/min silica, % above 2% threshold), writes Gold Delta, and prepares for dashboards.  

---

## 🛠️ Tech Stack
- **Azure Databricks** — data cleaning, validation, transformations (PySpark, Delta Lake)  
- **Azure Data Lake Gen2** — raw + cleaned storage (Bronze)  
- **Azure Data Factory** — orchestration, scheduling, retries, alerts  
- **Python (PySpark, matplotlib)** — transformations + visualizations  

---

## ✅ Why this matters
This project mirrors real **historian-to-cloud workflows** in the mining industry:  
- Reliable ingestion and schema normalization  
- Data quality validation (range checks, nulls, anomalies)  
- Automated daily KPI generation  
- Monitoring + orchestration for production readiness  

> **Key takeaway:** An end-to-end Azure pipeline that turns raw mining process data into actionable KPIs — the same type of system engineers at Teck could use to ensure product quality.
