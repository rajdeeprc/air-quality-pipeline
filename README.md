# air-quality-pipeline
# 🌍 Real-Time Air Quality Intelligence Pipeline

> An end-to-end Data Engineering project built on **Databricks** using the **Medallion Architecture** — ingesting real-time global air quality data, processing it through Bronze → Silver → Gold Delta Lake layers, and surfacing analytics-ready insights across 20 major cities worldwide.

---

## 📌 Project Overview

Air pollution is one of the world's most critical public health challenges. This pipeline automatically collects real-time air quality measurements from monitoring stations across 20 global cities via the **OpenAQ v3 API**, processes and validates the data through a three-layer Medallion Architecture, and produces business-ready KPI tables and visualizations — all built entirely within **Databricks Community Edition**.

---

## 🏗️ Architecture

```
OpenAQ v3 API (Free)
        │
        ▼
┌───────────────────┐
│   BRONZE LAYER    │  ← Raw JSON ingestion, no transformations
│  Delta Lake Table │    7,215 records across 20 cities
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│   SILVER LAYER    │  ← Cleaned, validated, AQI-categorised
│  Delta Lake Table │    7,033 valid records (182 flagged invalid)
└────────┬──────────┘
         │
         ▼
┌───────────────────┐
│    GOLD LAYER     │  ← Aggregated KPIs, trends, AQI summaries
│  3 Delta Tables   │    Business-ready for analytics & dashboards
└────────┬──────────┘
         │
         ▼
  📊 Visualizations
  (Matplotlib / Seaborn)
```

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| **Databricks Community Edition** | Unified analytics platform |
| **Apache Spark (PySpark)** | Distributed data processing |
| **Delta Lake** | ACID-compliant lakehouse storage |
| **OpenAQ v3 API** | Real-time air quality data source |
| **Python** | Pipeline logic and transformations |
| **Matplotlib / Seaborn** | Visualizations and dashboards |
| **SQL** | Aggregations and analytical queries |

---

## 📁 Project Structure

```
air-quality-pipeline/
├── 00_setup.py                  # Database creation, managed Delta table scaffolding
├── 01_bronze_ingestion.py       # OpenAQ API ingestion → Bronze Delta table
├── 02_silver_transformation.py  # Cleaning, validation, AQI categorisation → Silver
├── 03_gold_aggregation.py       # KPI aggregations → 3 Gold Delta tables
├── 04_visualization.py          # 6 analytical charts and executive dashboard
└── README.md
```

---

## 🗃️ Data Model

### Bronze — `air_quality_db.bronze_raw_measurements`
Raw measurements exactly as received from the API. Acts as the audit trail.

| Column | Type | Description |
|---|---|---|
| city | STRING | Target city name |
| country | STRING | ISO country code |
| pollutant | STRING | Pollutant type (pm25, pm10, no2, etc.) |
| value | DOUBLE | Raw measurement value |
| unit | STRING | Measurement unit (µg/m³) |
| location_name | STRING | Monitoring station name |
| latitude | DOUBLE | Station latitude |
| longitude | DOUBLE | Station longitude |
| measured_at | TIMESTAMP | When the reading was taken |
| ingested_at | TIMESTAMP | When the pipeline ingested it |
| source_url | STRING | API endpoint that served the data |
| raw_json | STRING | Complete raw API response |

### Silver — `air_quality_db.silver_clean_measurements`
Cleaned, validated and enriched data. Safe for analyst consumption.

| Column | Type | Description |
|---|---|---|
| *(all Bronze columns except raw_json, source_url)* | | |
| aqi_category | STRING | US EPA AQI category (Good → Hazardous) |
| is_valid | BOOLEAN | Passes all validation rules |

### Gold Tables

**`gold_city_rankings`** — Aggregated pollutant statistics per city  
**`gold_pollutant_trends`** — Daily average readings per city per pollutant  
**`gold_aqi_summary`** — AQI category percentage breakdown per city

---

## 📊 Key Findings

| Metric | Value |
|---|---|
| Cities monitored | 20 across 6 continents |
| Total records ingested | 7,215 |
| Valid records after cleaning | 7,033 (97.5%) |
| Invalid records flagged | 182 (2.5%) |
| Most polluted city | Delhi — 267.44 µg/m³ PM2.5 (Hazardous) |
| Cleanest city | London — 6.76 µg/m³ PM2.5 (Good) |
| Cities exceeding WHO guideline (15 µg/m³) | 15 out of 20 |

### 🔴 Top 5 Most Polluted Cities (PM2.5)
| City | Avg PM2.5 (µg/m³) | AQI Category |
|---|---|---|
| Delhi | 267.44 | Hazardous |
| Dhaka | 121.64 | Very Unhealthy |
| Lahore | 120.05 | Very Unhealthy |
| Cairo | 63.46 | Unhealthy |
| Karachi | 48.70 | Unhealthy |

### 🟢 Top 5 Cleanest Cities (PM2.5)
| City | Avg PM2.5 (µg/m³) | AQI Category |
|---|---|---|
| London | 6.76 | Good |
| New York | 7.09 | Good |
| Tokyo | 7.55 | Good |
| Nairobi | 10.83 | Good |
| Beijing | 15.58 | Good |

---

## 🔄 Pipeline Stages

### Stage 1 — Bronze Ingestion (`01_bronze_ingestion.py`)
- Queries OpenAQ v3 API using **coordinate-based station discovery** (lat/lon + radius)
- Navigates the v3 data model: Location → Sensors → Measurements
- Implements **retry logic with exponential backoff** for 429 rate limit handling
- Appends raw records to Bronze Delta table with full audit trail

### Stage 2 — Silver Transformation (`02_silver_transformation.py`)
- **Deduplication** — removes duplicate readings by city + location + pollutant + timestamp
- **Pollutant filtering** — keeps only the 6 target pollutants
- **Value validation** — flags nulls, negatives, and unrealistically high readings (≥10,000)
- **AQI categorisation** — assigns US EPA PM2.5 categories via Spark UDF
- Writes with `overwriteSchema=true` for safe reruns

### Stage 3 — Gold Aggregation (`03_gold_aggregation.py`)
- **City Rankings** — avg/min/max per city per pollutant + dominant AQI via `ROW_NUMBER()` window function
- **Pollutant Trends** — daily averages grouped by city and pollutant for time series analysis
- **AQI Summary** — percentage distribution of AQI categories per city using `SUM() OVER (PARTITION BY)` window

### Stage 4 — Visualization (`04_visualization.py`)
Six charts produced from Gold tables:
1. PM2.5 City Rankings Bar Chart (AQI colour-coded)
2. Multi-Pollutant Heatmap (normalised across all cities)
3. AQI Category Stacked Horizontal Bar
4. PM2.5 Trend Lines Over Time (top polluted vs cleanest)
5. Grouped Bar — Multiple Pollutants for Top 10 Cities
6. Executive KPI Dashboard

---

## ⚙️ Setup & Usage

### Prerequisites
- Databricks Community Edition account ([sign up free](https://community.cloud.databricks.com))
- OpenAQ v3 API key ([register free](https://explore.openaq.org/register))
- Databricks Runtime 12.x or higher (Delta Lake included)

### Steps

**1. Clone this repository**
```bash
git clone https://github.com/YOUR_USERNAME/air-quality-pipeline.git
```

**2. Import notebooks to Databricks**
- In Databricks workspace: File → Import → upload each `.py` file

**3. Create and attach a cluster**
- Compute → Create Cluster → Databricks Runtime 12.x+ → Create

**4. Run notebooks in order**
```
00_setup.py              → Creates database and Delta tables
01_bronze_ingestion.py   → Ingests data from OpenAQ API
02_silver_transformation → Cleans and validates records
03_gold_aggregation.py   → Builds KPI tables
04_visualization.py      → Renders charts and dashboard
```

**5. Add your API key**

In `00_setup.py` and `01_bronze_ingestion.py`, replace:
```python
OPENAQ_API_KEY = "PASTE_YOUR_API_KEY_HERE"
```

---

## 💡 Engineering Decisions & Lessons Learned

**Managed Tables over DBFS paths** — Databricks has deprecated public DBFS root access in newer workspaces. Managed Delta tables are cleaner, more secure, and align with modern Databricks best practices.

**Explicit schema definition** — Spark cannot infer types from `None` values. Defining schemas explicitly prevents silent type inference failures on real-world messy data.

**Coordinate-based station discovery** — OpenAQ v3's city name search returns unreliable results (searching "London" returned stations in Ghana). Using lat/lon + radius produces accurate, city-specific results.

**Sensor-level API calls** — OpenAQ v3 restructured its data model. Measurements are now served per sensor ID, not per location. The pipeline navigates Location → Sensor → Measurements accordingly.

**Retry logic with city-level delays** — Without deliberate pacing, 20 cities × multiple stations × multiple sensors exhausts the API rate limit (429). A 1s sensor delay + 5s city delay + 60s backoff on 429 allows the full pipeline to complete cleanly.

**Flag don't drop invalid records** — The Silver layer marks records as `is_valid = false` rather than deleting them. This preserves data lineage and allows downstream investigation of why records failed validation.

---

## 🌐 Data Source

[OpenAQ](https://openaq.org/) is an open-source platform aggregating air quality data from government monitoring stations worldwide. The v3 API is free with registration and provides real-time and historical measurements for PM2.5, PM10, NO2, O3, CO, and SO2.

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

## 🙋 Author

Built as a portfolio Data Engineering project demonstrating end-to-end pipeline development using Databricks, Delta Lake, PySpark, and real-world REST API integration.

> ⭐ If you found this project useful, consider giving it a star!
