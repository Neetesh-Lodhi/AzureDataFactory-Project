# AzureDataFactory-Project
Azure Data Factory End-To-End Project | PySpark | Azure Data Migration | Medallion Architecture | Azure DevOps For Data Engineers

🚀 Azure Data Factory – End-to-End Data Engineering Project

This repository demonstrates a production-style Azure Data Factory (ADF) data engineering workflow built using Medallion Architecture (Bronze → Silver → Gold).
The project covers data ingestion, orchestration, transformation, ranking, automation, Git integration, and alerting, simulating real-world enterprise pipelines used in modern lakehouse platforms like Azure + Microsoft Fabric.
🏗️ Architecture Overview
The solution follows the Medallion Architecture pattern:
Sources → Bronze → Silver → Gold → Business Views

Layer	Purpose
🥉 Bronze	Raw ingestion from On-Prem, REST API, Azure SQL
🥈 Silver	Cleaned, transformed, Delta upserts
🥇 Gold	Aggregated business views with ranking

ADF acts as the orchestrator, handling ingestion, transformation, scheduling, monitoring, and alerts.

⚙️ Core Features
✅ On-Prem File Ingestion using Self-Hosted Integration Runtime
✅ REST API Ingestion using HTTP Connector
✅ Incremental SQL Load without watermark tables
✅ Dynamic pipelines using ForEach + Parameters
✅ Parent-Child Pipeline Orchestration
✅ Failure Alerts using Logic Apps + Email
✅ Transformations using Mapping Data Flows (Spark under the hood)
✅ Ranking with Dense Rank Window Functions
✅ Delta Upsert (Silver) and Overwrite (Gold)
✅ Scheduling with Triggers
✅ Git Integration using Azure DevOps / GitHub

🧱 Azure Components Used
Azure Data Factory
Azure Data Lake Storage Gen2
Azure SQL Database
Self-Hosted Integration Runtime
Logic Apps
Azure DevOps / GitHub
Delta Lake
Mapping Data Flows

🔗 Linked Services
Linked Services define connections to sources and sinks.
Examples used:
File System (On-Prem)
HTTP (REST API / GitHub JSON)
Azure SQL Database
Azure Data Lake Storage Gen2
Without linked services, ADF cannot communicate with external systems.

⚡ Integration Runtimes
Type	Usage
AutoResolve IR	Azure-to-Azure data movement
Self-Hosted IR	On-Prem files and private network access
Self-Hosted IR bridges ADF with local machines using local CPU and memory.
📥 Data Ingestion
🗂️ On-Prem Files

Source: Local CSV / JSON files
Target: ADLS Gen2 (Bronze Layer)
Dynamic file ingestion using pipeline parameters
Parallel execution using ForEach

[
  {"filename": "dim_passenger.csv"},
  {"filename": "dim_flight.csv"}
]

🌐 REST API

HTTP Linked Service
GitHub raw JSON ingestion
Copy Activity to Bronze ADLS
Example:
https://raw.githubusercontent.com/.../dim_airport.json
🗄️ Incremental SQL Load
Modern incremental approach without watermark tables:
Lookup last load from JSON
Query new records
Copy to Bronze
Update last_load.json automatically

SELECT *
FROM dbo.fact_bookings
WHERE booking_date >
'@{activity('Lookup_last_load').output.firstRow.last_load}'

🔄 Pipeline Orchestration

Parent pipeline executes child pipelines:

On-Prem Ingestion

API Ingestion

Incremental SQL Load

Dynamic parameter passing avoids string-conversion bugs:

@pipeline().parameters.files


This ensures typed arrays and objects flow correctly.

🚨 Alerts with Logic Apps

ADF integrates with Logic Apps for failure notifications.

Flow:

ADF Web Activity → HTTP Trigger

Logic App → Send Email

Email includes pipeline name, run id, status, error

Example payload:

{
  "pipeline_name": "@pipeline().Pipeline",
  "run_id": "@pipeline().RunId",
  "status": "@activity('execute_incremental').output.status",
  "error": "@activity('execute_incremental').error.message"
}

🔧 Transformations (Silver Layer)

Mapping Data Flows provide Spark-based transformations:

Derived Columns

Select / Rename

Filter

Cast

Join

Aggregate

Window Functions

Silver uses Delta Upsert for evolving schemas.

🪟 Window Functions – Ranking

After aggregation, business metrics are ranked using:

dense_rank()

row_number()

Difference:

Function	Behavior
rank()	Skips numbers after ties
dense_rank()	No gaps in ranking

Used for business views like Top Airlines by Revenue.

🥇 Gold Layer – Business Views

Steps:

Read Silver Delta

Join Fact + Dimensions

Aggregate revenue

Apply Dense Rank

Filter Top N

Overwrite Gold Delta

Gold uses overwrite mode for freshness instead of upserts.

⏰ Scheduling

Pipelines are automated using Schedule Triggers:

Daily execution

Time-zone aware

Supports tumbling windows for backfills

🔁 Git Integration

ADF integrates with GitHub / Azure DevOps:

Feature branches

Pull Requests

Main branch publish

ARM templates auto-generated

This enables collaborative development and CI/CD.

📌 Project Flow Summary
On-Prem / API / SQL
        ↓
      Bronze
        ↓
      Silver (Delta Upsert)
        ↓
      Gold (Aggregate + Rank)
        ↓
 Business Views

🧠 Key Learnings

ADF is an orchestrator, not a processor

Mapping Data Flows use Spark internally

Delta improves performance and schema evolution

Parent-Child pipelines simplify orchestration

Dense Rank avoids confusing ranking gaps

Logic Apps enable enterprise alerting

Git is mandatory for team-scale ADF projects
