# Data Lake Concepts & Reference Architecture

A **Data Lake** is a centralized repository that allows you to store all your structured, semi-structured, and unstructured data at any scale. Unlike a traditional data warehouse, which requires data to be structured before storage (Schema-on-Write), a Data Lake allows you to store data in its raw format and define the structure only when reading it (Schema-on-Read).

---

## Data Lake vs. Data Warehouse

| Feature | Data Lake | Data Warehouse |
| :--- | :--- | :--- |
| **Data Format** | Raw, unstructured, semi-structured, structured | Structured, processed |
| **Schema** | Schema-on-Read (defined when querying) | Schema-on-Write (defined before loading) |
| **Storage Cost** | Relatively low (Object storage like S3, GCS) | Higher (Optimized database storage) |
| **Primary Users** | Data Scientists, Data Engineers | Business Analysts, BI Developers |

---

## Core Architecture Layers

1. **Ingestion Layer**: Imports raw data from various sources (databases, APIs, IoT devices, logs).
2. **Storage Layer**: Scalable, low-cost object storage (e.g., AWS S3, Google Cloud Storage, Azure ADLS Gen2).
3. **Processing/Compute Layer**: Decoupled query engines that process data (e.g., Apache Spark, Trino, Databricks, BigQuery).
4. **Metadata & Catalog Layer**: Tracks schemas and file paths (e.g., AWS Glue Catalog, Apache Hive Metastore).
5. **Consumption Layer**: Analytics tools, BI dashboards, and ML training pipelines.
