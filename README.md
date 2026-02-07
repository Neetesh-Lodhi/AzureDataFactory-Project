# AzureDataFactory-Project
Azure Data Factory End-To-End Project | PySpark | Azure Data Migration | Medallion Architecture | Azure DevOps For Data Engineers

# 🚀 Azure Data Factory – End-to-End Data Engineering Project

![Azure](https://img.shields.io/badge/Cloud-Azure-0078D4?logo=microsoft-azure&logoColor=white)
![ADF](https://img.shields.io/badge/Data%20Factory-Orchestration-0078D4?logo=microsoft-azure&logoColor=white)
![Spark](https://img.shields.io/badge/Transformation-Spark%20%2F%20Data%20Flows-E25A1C?logo=apache-spark&logoColor=white)
![SQL](https://img.shields.io/badge/Database-Azure%20SQL-0056D2?logo=microsoft-sql-server&logoColor=white)

This repository demonstrates a **production-ready** data engineering pipeline using **Azure Data Factory (ADF)**. The project implements a **Medallion Architecture** (Bronze → Silver → Gold) to ingest, clean, and analyze data from multiple sources.

It simulates a real-world enterprise scenario including **Self-Hosted Integration Runtimes**, **Incremental Loading**, and **Logic App Alerting**.

---

## 🏗️ Architecture Overview

The solution follows the Lakehouse pattern using the **Medallion Architecture**:

**1. 🥉 Bronze Layer (Raw):** Direct ingestion from On-Prem files, REST APIs, and SQL (Parquet/JSON).
**2. 🥈 Silver Layer (Clean):** Transformed data with Delta Upserts using Mapping Data Flows.
**3. 🥇 Gold Layer (Curated):** Aggregated business views with Ranking metrics ready for reporting.

### 📌 Data Flow Diagram
> **Source Systems** (On-Prem, API, SQL)  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;⬇️ *Ingestion (Copy Activity)* > **Bronze Layer** (ADLS Gen2)  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;⬇️ *Transformation (Spark Data Flows)* > **Silver Layer** (Delta Upsert)  
> &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;⬇️ *Aggregation (Window Functions)* > **Gold Layer** (Business Views)

---

## ⚙️ Key Features

* **Hybrid Ingestion:** Securely moves data from local On-Prem machines to Cloud using **Self-Hosted Integration Runtime**.
* **API Integration:** Ingests complex JSON data via REST API (HTTP Connector).
* **Incremental SQL Loading:** Implements high-performance incremental loads **without** using legacy watermark tables.
* **Dynamic Pipelines:** Uses `ForEach` loops and Parameterization for reusable, metadata-driven pipelines.
* **Automated Alerting:** Integrates with **Logic Apps** to send email notifications upon pipeline failure.
* **Advanced Analytics:** Uses `Dense Rank` window functions for business ranking logic.
* **CI/CD:** Fully integrated with Git (Azure DevOps / GitHub) for version control.

---

## 🧱 Tech Stack

| Component | Usage |
| :--- | :--- |
| **Azure Data Factory** | Primary Orchestration & Scheduling |
| **Azure Data Lake Gen2** | Storage for Bronze, Silver, and Gold layers |
| **Mapping Data Flows** | Low-code Spark transformations |
| **Azure SQL Database** | Source for transactional data |
| **Logic Apps** | Serverless workflow for Email Alerts |
| **Self-Hosted IR** | Bridge for private network/local file access |

---

## 📥 Data Ingestion Strategy

### 1. On-Premise Files
* **Source:** Local CSV/JSON files.
* **Technique:** Self-Hosted Integration Runtime (IR) installed on a local VM.
* **Logic:** Dynamic file selection using pipeline parameters.

```json
/* Example Parameter passed to Pipeline */
[
  {"filename": "dim_passenger.csv"},
  {"filename": "dim_flight.csv"}
]
2. REST API (GitHub JSON)Source: Raw JSON data from GitHub via HTTP connector.Technique: Copy Activity to Sink (ADLS Bronze).3. Smart Incremental SQL LoadInstead of maintaining a complex watermark table in the database, the pipeline:Lookups the last_load_date from a JSON file in ADLS.Queries only new records.Updates the JSON file upon success.SQL-- Dynamic Query in ADF Copy Activity
SELECT * FROM dbo.fact_bookings 
WHERE booking_date > '@{activity('Lookup_last_load').output.firstRow.last_load}'
🔧 Transformation Logic (Silver & Gold)Transformations are performed using Mapping Data Flows (running on Spark clusters).🥈 Silver LayerGoal: Clean and Normalize.Method: Delta Upsert (Update if row exists, Insert if new).Steps: Derived Columns -> Cast Types -> Filter -> Upsert.🥇 Gold LayerGoal: Business Aggregation.Method: Overwrite (Ensures reporting data is always fresh).Ranking Logic: Used dense_rank() to handle "Top Airlines by Revenue" to avoid gaps in numbering.FunctionOutput ExampleWhy we used it?rank()1, 1, 3, 4Skips numbers after ties.dense_rank()1, 1, 2, 3Best for business reporting (No gaps).🔄 Orchestration & MonitoringParent-Child Pipeline PatternThe master pipeline acts as a controller, triggering child pipelines for Ingestion, Processing, and Loading. This ensures:Better Error Handling.Modular Design.Easy Debugging.🚨 Alerting SystemIf a pipeline fails, ADF triggers a Web Activity that calls a Logic App webhook.Payload sent to Logic App:JSON{
  "pipeline_name": "@pipeline().Pipeline",
  "run_id": "@pipeline().RunId",
  "status": "Failed",
  "error": "@activity('Execute Pipeline').error.message"
}
🧠 Key LearningsADF is an Orchestrator: Heavy transformations should be delegated to Data Flows (Spark) or Databricks, not done inside the control flow.Parameterization: Hard-coding values is a bad practice. Always use pipeline parameters for flexibility.Git Integration: Essential for saving work, version history, and collaboration.Schema Evolution: Using Delta format allows the schema to evolve over time without breaking pipelines.Project maintained by [Your Name]
