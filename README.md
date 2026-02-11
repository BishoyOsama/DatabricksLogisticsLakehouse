# Databricks Logistics Lakehouse

The **Databricks-side** component of the [Logistics Lakehouse](https://github.com/BishoyOsama/LogisticsLakehouse) pipeline. This repo contains the PySpark notebooks that implement the **Bronze** and **Silver** layers of the Medallion Architecture on Databricks.

> For the full pipeline (Airflow orchestration, Kafka producer, dbt Gold layer), see the main repo: [LogisticsLakehouse](https://github.com/BishoyOsama/LogisticsLakehouse)

## How It Fits

```
LogisticsLakehouse (Airflow + Kafka + dbt)
│
├─ Orchestration & Streaming ── Airflow DAGs, Kafka Producer
├─ Bronze & Silver Layers ───── This Repo (Databricks Notebooks) ◄──
└─ Gold Layer ───────────────── dbt Models
```

## Project Structure

```
scripts/
├── bronze/
│   ├── batch_processing.ipynb           # Batch ingestion from CRM/ERP CSV files
│   └── stream_processing.ipynb          # Real-time ingestion from Confluent Kafka
└── silver/
    ├── silver_batch_orchestration.ipynb  # Orchestrator for all silver notebooks
    ├── crm/
    │   ├── silver_crm_cust_info.ipynb   # Customer info cleansing
    │   ├── silver_crm_prd_info.ipynb    # Product info cleansing
    │   └── silver_sales_details.ipynb   # Sales details cleansing
    └── erp/
    |    ├── silver_erp_cust_az12.ipynb   # Customer demographics cleansing
    |    ├── silver_erp_loc_a101.ipynb     # Location data cleansing
    |    └── silver_erp_px_cat_g1v2.ipynb # Product category cleansing
    |__ kafka_orders/
         |
         |__silver_kafka_orders.ipynb  # Clean Stream data from bronze layer
```

## Bronze Layer

| Notebook | Mode | Description |
|----------|------|-------------|
| `batch_processing` | Batch | Ingests CRM/ERP CSV files with incremental tracking (skips already-processed files) |
| `stream_processing` | Streaming | Consumes from Confluent Kafka `orders` topic with Schema Registry (JSON) |

## Silver Layer

Cleanses and standardizes Bronze data. The orchestrator runs all notebooks sequentially via `dbutils.notebook.run()`.

| Source | Notebook | Key Transformations |
|--------|----------|---------------------|
| CRM | `silver_crm_cust_info` | Trim names, remove empty IDs |
| CRM | `silver_crm_prd_info` | Deduplicate products, clean catalog data |
| CRM | `silver_sales_details` | Validate and clean sales transactions |
| ERP | `silver_erp_cust_az12` | Standardize gender values |
| ERP | `silver_erp_loc_a101` | Normalize country codes and customer IDs |
| ERP | `silver_erp_px_cat_g1v2` | Trim strings, clean category classifications |

## Tech Stack

| Component | Technology |
|-----------|------------|
| Data Platform | Databricks (Unity Catalog) |
| Processing | Apache Spark (PySpark) |
| Storage | Delta Lake |
| Streaming | Confluent Kafka + Schema Registry |
| Secrets | Databricks Secret Scopes |

## Prerequisites

- Databricks workspace with Unity Catalog (`workspace.bronze` / `workspace.silver` schemas)
- Databricks secret scope `confluent-kafka` with keys: `bootstrap-servers`, `api-key`, `api-secret`, `schema-registry-url`, `schema-registry-api-key`, `schema-registry-api-secret`
- Source CRM/ERP data files accessible from the workspace

## Usage

1. **Bronze Batch** — Run `scripts/bronze/batch_processing.ipynb` to ingest new files
2. **Bronze Stream** — Run `scripts/bronze/stream_processing.ipynb` to consume from Kafka
3. **Silver** — Run `scripts/silver/silver_batch_orchestration.ipynb` to execute all transformations
4. **Silver** _ RUN `scripts/silver/kafka_orders/silver_kafka_orders.ipynb` to execute transformations for stream orders data

These notebooks are triggered automatically by the Airflow DAGs in the [main repository](https://github.com/BishoyOsama/LogisticsLakehouse).
