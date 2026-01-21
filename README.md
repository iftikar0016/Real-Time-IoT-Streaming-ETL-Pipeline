# Real-Time IoT Streaming ETL Pipeline with Databricks Delta Live Tables (DLT)

![Databricks](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=databricks&logoColor=white)
![PySpark](https://img.shields.io/badge/PySpark-E25A1C?style=for-the-badge&logo=apachespark&logoColor=white)
![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)
![Azure](https://img.shields.io/badge/Azure-0078D4?style=for-the-badge&logo=microsoftazure&logoColor=white)

## 📌 Project Overview
This project demonstrates an end-to-end **Real-Time Streaming ETL Pipeline** designed for a **Structural Health Monitoring (SHM)** system. It processes high-velocity IoT sensor data (Temperature, Vibration, Tilt) from 5 major European bridges to detect anomalies and aggregate safety metrics.

Built on the **Databricks Lakehouse Platform**, this solution leverages **Delta Live Tables (DLT)** for orchestration, **Unity Catalog** for governance, and **Spark Structured Streaming** for low-latency processing.

## 🏗 Architecture
The pipeline follows the **Medallion Architecture** (Bronze → Silver → Gold) to incrementally improve data quality and structure.

![Architecture Diagram](https://github.com/iftikar0016/Real-Time-IoT-Streaming-ETL-Pipeline/blob/main/images/architecture.png)
*(Figure 1: High-level architecture showing data flow from IoT simulation to Gold aggregates)*

### Data Flow
1.  **Ingestion:** Simulated sensors push data with random network lag to a Unity Catalog Volume.
2.  **Bronze (Raw):** Ingests raw JSON data into streaming Delta tables.
3.  **Silver (Enriched & Cleaned):** Joins sensor streams with static bridge metadata and applies data quality rules.
4.  **Gold (Aggregated):** Performs window-based aggregation and stream-stream joins to create a unified reporting view.

---

## 🚀 Pipeline Execution & Lineage
The core of this project is the Delta Live Tables (DLT) pipeline, which automatically manages infrastructure, orchestration, and dependencies.

![DLT Pipeline DAG](https://github.com/iftikar0016/Real-Time-IoT-Streaming-ETL-Pipeline/blob/main/images/Streaming_Pipeline_2.png)
*(Figure 2: The DLT Directed Acyclic Graph (DAG) visualizing the lineage from ingestion to final aggregation)*

---

## 🛠 Technical Implementation Details

### 1. Handling Late Data (Watermarking)
Real-world IoT data often arrives out of order due to network latency.
* **Challenge:** How to aggregate data accurately when events arrive late?
* **Solution:** Implemented a **2-minute watermark** in Spark Structured Streaming. This allows the engine to wait for late data before finalizing the 10-minute tumbling windows, ensuring data accuracy while managing state store memory efficiently.

### 2. Stream-Stream Joins
Merging three independent sensor streams (Temperature, Vibration, Tilt) is complex in a streaming context.
* **Implementation:** Joined streams based on `window_start`, `window_end`, and `bridge_id`.
* **Constraint:** Ensured all streams shared identical watermark thresholds to allow Spark to clean up state correctly and prevent Out-Of-Memory (OOM) errors.

### 3. Data Quality & Governance (DLT Expectations)
Used DLT declarative expectations to enforce data quality without stopping the pipeline.

![Data Quality Rules](https://github.com/iftikar0016/Real-Time-IoT-Streaming-ETL-Pipeline/blob/main/images/Expectation.png)

*(Figure 3: Real-time data quality enforcement showing records passing and failing validation rules)*

* **Critical Rules:** `expect_or_drop` for null timestamps (prevents downstream corruption).
* **Business Rules:** `expect` for out-of-range sensor readings (flags anomalies for analysis).

---

## 📂 Repository Structure

| File | Description |
| :--- | :--- |
| `00_data_generator.py` | Multi-threaded Python script simulating IoT sensors with random latency. |
| `01_bronze.py` | Ingests raw files from Landing Zone (Unity Catalog Volume) to Bronze tables. |
| `02_silver.py` | Performs Stream-Static joins and applies DLT Expectations for quality. |
| `03_gold.py` | Handles Window Aggregations, Watermarking, and Stream-Stream joins. |
| `config.json` | Configuration file for table names and paths. |

---

## 📊 Final Output (Gold Layer)
The final `bridge_metrics` table provides aggregated insights ready for dashboarding.

![Gold Table Results](https://github.com/iftikar0016/Real-Time-IoT-Streaming-ETL-Pipeline/blob/main/images/Query.png)
*(Figure 4: Final aggregated metrics calculated over 10-minute tumbling windows)*

---


## 📈 Future Enhancements
* **Alerting:** Integrate Databricks SQL Alerts to notify engineers when vibration exceeds safety thresholds.
* **Visualization:** Build a Databricks Lakeview Dashboard for real-time monitoring.
* **CI/CD:** Implement Databricks Asset Bundles (DABs) for automated deployment.
