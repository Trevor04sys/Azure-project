# Azure-project
# Azure End-to-End Data Pipeline — Music Streaming Analytics

![Azure](https://img.shields.io/badge/Azure-Data_Factory-0089D6?logo=microsoftazure)
![Azure](https://img.shields.io/badge/Azure-Data_Lake_Gen2-0089D6?logo=microsoftazure)
![Azure](https://img.shields.io/badge/Azure-Synapse_Analytics-0089D6?logo=microsoftazure)
![PowerBI](https://img.shields.io/badge/Power_BI-Reporting-F2C811?logo=powerbi)
![Status](https://img.shields.io/badge/Status-Complete-brightgreen)

## 📌 Project Overview

An end-to-end cloud data pipeline built on Microsoft Azure that ingests, 
transforms, and serves analytics on a music streaming dataset 
(modeled after Spotify-style data). Implements CDC-based incremental 
loading, Medallion Lakehouse architecture, and automated multi-table 
orchestration using Azure Data Factory.

---

## 🏗️ Architecture
Azure SQL Database
│
▼
Azure Data Factory (ADF)
├── Incremental Ingestion (CDC Watermark)
└── Multi-Table Loop Pipeline (ForEach)
│
▼
Azure Data Lake Gen2 (ADLS)
├── 🥉 Bronze  → Raw Parquet (incremental snapshots)
├── 🥈 Silver  → Cleaned & transformed (Databricks/PySpark)
└── 🥇 Gold    → Aggregated, analytics-ready
│
▼
Azure Synapse Analytics
│
▼
Power BI Dashboards

![Architecture Diagram](./pipeline/architecture.png)
---

## 📊 Data Model (Star Schema)

| Table | Type | CDC Column |
|---|---|---|
| `factstream` | Fact | `stream_timestamp` |
| `dimuser` | Dimension | `updated_at` |
| `dimtrack` | Dimension | `updated_at` |
| `dimartist` | Dimension | `updated_at` |
| `dimdate` | Dimension | `date` |

---

## ⚙️ Key Pipeline Features

### 1. Watermark-Based Incremental Ingestion
- Reads last CDC timestamp from a JSON watermark file stored in ADLS
- Dynamically constructs SQL queries filtering records newer than 
  the watermark
- After successful load, captures `MAX(cdc_col)` from source and 
  updates the watermark file
- If no new data: automatically deletes the empty Parquet file 
  (clean storage management)

### 2. Multi-Table Loop Orchestration
- Single parameterized pipeline handles all 5 tables via ForEach loop
- Table config passed as JSON array — no hardcoding, fully reusable
- Each iteration runs independently with its own CDC state

### 3. Parameterized Dynamic Datasets
- `Json_dynamic` and `Parquet_dynamic` datasets accept 
  `container`, `folder`, and `file` as runtime parameters
- Enables full reusability across tables and pipeline runs

### 4. Security
- Azure Data Factory uses **System-Assigned Managed Identity** 
  for authentication — no credentials stored in code
- Encrypted credentials for all linked services

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| Ingestion & Orchestration | Azure Data Factory |
| Storage | Azure Data Lake Storage Gen2 |
| Transformation | Azure Databricks (PySpark) |
| Serving | Azure Synapse Analytics |
| Visualization | Power BI |
| Source | Azure SQL Database |
| Format | Parquet (Snappy compression) |

---

## 📁 Repository Structure
├── datasets/
│   ├── AzureSql.json           # Azure SQL source dataset
│   ├── Json_dynamic.json       # Parameterized JSON dataset (CDC state)
│   └── Parquet_dynamic.json    # Parameterized Parquet sink dataset
├── linkedServices/
│   ├── Azuresql.json           # Azure SQL linked service
│   └── DataLake.json           # ADLS Gen2 linked service
├── pipelines/
│   ├── Incremental_ingestion.json          # Single-table CDC pipeline
│   └── Incremental_ingestion_using_loop.json  # Multi-table loop pipeline
├── factory/
│   └── raj-df-Azureproject.json           # ADF factory config
└── publish_config.json


---

## 🚀 How to Deploy

1. Clone this repository
2. Import pipeline JSONs into your Azure Data Factory instance
3. Update linked services with your own connection strings
4. Create the CDC watermark JSON files in your Bronze container:
   `bronze/cdc/{table_name}/cdc.json` with initial value `{"cdc": "1900-01-01"}`
5. Trigger the `Incremental_ingestion_using_loop` pipeline

---

## 📈 Results

- Automated incremental ingestion for 5 tables with zero manual 
  intervention
- Snappy-compressed Parquet reduces storage costs vs raw CSV by ~70%
- Watermark logic ensures no duplicate data across pipeline runs
- Reusable pipeline design — add new tables by updating config array only

- > ⚠️ Note: Azure free trial has expired. 
> All pipeline logic is fully documented in the 
> `/pipeline` and `/dataset` folders as ADF JSON 
> exports, which can be imported into any ADF instance.
