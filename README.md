# AzureDataFactory-Project
Azure Data Factory End-To-End Project | PySpark | Azure Data Migration | Medallion Architecture | Azure DevOps For Data Engineers

 🚀 Azure Data Factory – End-to-End Data Engineering Project

![Azure](https://img.shields.io/badge/azure-%230072C6.svg?style=for-the-badge&logo=microsoftazure&logoColor=white)
![Azure Data Factory](https://img.shields.io/badge/Data%20Factory-0078D4?style=for-the-badge&logo=microsoft-azure&logoColor=white)
![Azure SQL](https://img.shields.io/badge/Azure%20SQL-0056D2?style=for-the-badge&logo=microsoft-sql-server&logoColor=white)
![Spark](https://img.shields.io/badge/Spark-E25A1C?style=for-the-badge&logo=apache-spark&logoColor=white)

This repository demonstrates a **production-style Azure Data Factory (ADF) data engineering workflow** built using the **Medallion Architecture** (Bronze → Silver → Gold).

The project simulates real-world enterprise pipelines found in modern lakehouse platforms (Azure + Microsoft Fabric), covering data ingestion, orchestration, transformation, ranking, automation, Git integration, and alerting.

---

## 🏗️ Architecture Overview

The solution follows the **Medallion Architecture** pattern where ADF acts as the primary orchestrator.

| Layer | Purpose | Tech |
| :--- | :--- | :--- |
| **🥉 Bronze** | Raw ingestion from On-Prem, REST API, & Azure SQL. | ADLS Gen2 (Parquet/JSON) |
| **🥈 Silver** | Cleaned, transformed data with Delta Upserts. | Mapping Data Flows (Spark) |
| **🥇 Gold** | Aggregated business views with ranking & analytics. | Delta Lake (Overwrite) |

### 📌 High-Level Data Flow
```mermaid
graph LR
    A[On-Prem Files] -->|Self-Hosted IR| B(Bronze Layer)
    C[REST API] -->|HTTP Connector| B
    D[Azure SQL] -->|Incremental Load| B
    B -->|Clean & Upsert| E(Silver Layer)
    E -->|Aggregate & Rank| F(Gold Layer)
    F -->|Power BI / Reporting| G[Business Views]
⚙️ Core Features✅ Hybrid Ingestion: On-Prem File Ingestion using Self-Hosted Integration Runtime.✅ API Integration: REST API Ingestion using HTTP Connector (GitHub raw JSON).✅ Smart SQL Loading: Incremental SQL Load without traditional watermark tables.✅ Dynamic Orchestration: Parent-Child Pipelines using ForEach loops and Parameter passing.✅ Enterprise Alerting: Failure monitoring via Logic Apps + Email.✅ Spark Power: Transformations using Mapping Data Flows (Spark under the hood).✅ Advanced Analytics: Ranking metrics using Dense Rank Window Functions.✅ CI/CD: Git Integration via Azure DevOps / GitHub.🧱 Azure Components UsedAzure Data Factory: Pipeline orchestration and monitoring.Azure Data Lake Storage Gen2: Storage for Bronze, Silver, and Gold layers.Azure SQL Database: Source system for transactional data.Self-Hosted Integration Runtime: Bridge for local/private network access.Logic Apps: Serverless workflow for email alerts.Mapping Data Flows: Visual data transformation (running on Spark clusters).📥 Data Ingestion Strategy1. On-Prem Files (Self-Hosted IR)Source: Local CSV/JSON files.Target: ADLS Gen2 (Bronze).Mechanism: Used Self-Hosted IR to bridge ADF with local machines using local CPU/Memory.Dynamic Loading: Utilized pipeline parameters to load files in parallel.JSON// Example Parameter Array for ForEach Activity
[
  {"filename": "dim_passenger.csv"},
  {"filename": "dim_flight.csv"}
]
2. REST API (HTTP)Ingests raw JSON from GitHub via the HTTP Linked Service.Copies data directly to the Bronze container.3. Incremental SQL Load (No Watermark Table)A modern approach to incremental loading without maintaining a separate watermark table in the database.Lookup: Check the last_load timestamp from a JSON file in storage.Query: Select only new records from the source.Update: Refresh last_load.json automatically after success.SQL-- Query used in Copy Activity
SELECT * FROM dbo.fact_bookings 
WHERE booking_date > '@{activity('Lookup_last_load').output.firstRow.last_load}'
🔧 Transformations (Silver & Gold)Transformations are handled by Mapping Data Flows, providing the power of Spark clusters without writing Scala/Python code.🥈 Silver Layer (Delta Upsert)Operations: Derived Columns, Select/Rename, Cast, and Filter.Write Mode: Upsert (Update if exists, Insert if new) to handle evolving schemas and data changes.🥇 Gold Layer (Aggregation & Ranking)Operations: Joins (Fact + Dimensions), Aggregation (Revenue), and Window Functions.Write Mode: Overwrite (Ensures freshness for reporting views).Ranking LogicUsed dense_rank() to calculate "Top Airlines by Revenue".FunctionBehaviorUse Caserank()Skips numbers after ties (1, 1, 3)Olympic Medalsdense_rank()No gaps in ranking (1, 1, 2)Business Reporting🔄 Orchestration & MonitoringParent-Child PipelinesThe Parent pipeline executes child pipelines (Ingestion, SQL Load, Processing) ensuring modularity. Parameters are passed dynamically to avoid string-conversion bugs.JSON// Passing array parameters to child pipeline
@pipeline().parameters.files
🚨 Alerting with Logic AppsADF Web Activity triggers a Logic App via HTTP POST on pipeline failure.Payload Example:JSON{
  "pipeline_name": "@pipeline().Pipeline",
  "run_id": "@pipeline().RunId",
  "status": "@activity('execute_incremental').output.status",
  "error": "@activity('execute_incremental').error.message"
}
🧠 Key Learnings"ADF is an orchestrator, not a processor."Mapping Data Flows utilize Spark internally, abstracting the complexity of cluster management.Delta Lake format is essential for performance and handling schema evolution (Upserts).Parent-Child Hierarchies significantly simplify debugging and maintenance.Dense Rank is preferred over standard Rank for business views to avoid confusing gaps in reporting.Git Integration is mandatory for any team-scale ADF project to handle versioning and CI/CD.Created by [Your Name]
### How to use this:

1.  Go to your GitHub repository.
2.  Click on the `README.md` file (or create one).
3.  Click the **Pencil icon** (Edit).
4.  Paste the code above.
5.  **(Optional but recommended):** I added a `mermaid` diagram code block in the Architecture section. GitHub renders this automatically as a flow chart. It will look very impressive!
6.  Commit changes.

Would you like me to help you write a "How to Run" section for users who might want to clo
