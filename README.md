# Azure End-to-End Data Engineering Project

A production-style data engineering project built on Microsoft Azure, implementing the Medallion Architecture to process the Adventure Works dataset through a complete ETL pipeline from raw ingestion to analytics-ready data.

---

## Architecture Overview

```
Data Source (GitHub API)
        │
        ▼
Azure Data Factory
(Dynamic Pipeline with ForEach + Parameterization)
        │
        ▼
Azure Data Lake Gen2 ── Bronze Layer (Raw CSV)
        │
        ▼
Azure Databricks + PySpark
(Transformations, Cleaning, Feature Engineering)
        │
        ▼
Azure Data Lake Gen2 ── Silver Layer (Parquet)
        │
        ▼
Azure Synapse Analytics
(Serverless SQL Pool, Views, External Tables)
        │
        ▼
Azure Data Lake Gen2 ── Gold Layer (Analytics Ready)
        │
        ▼
Power BI (Reporting)
```

---

## Technologies

| Layer | Technology | Purpose |
|-------|-----------|---------|
| Orchestration | Azure Data Factory | Pipeline scheduling and data movement |
| Storage | Azure Data Lake Gen2 | Scalable cloud storage with hierarchical namespace |
| Processing | Azure Databricks + PySpark | Distributed data transformation |
| Warehousing | Azure Synapse Analytics | Serverless SQL querying (Lakehouse pattern) |
| Visualization | Power BI | Business intelligence reporting |
| Security | Azure Managed Identity | Passwordless authentication between services |

---

## Dataset

**Adventure Works** — a Microsoft sample dataset simulating a retail company's operations.

| Table | Description | Type |
|-------|-------------|------|
| Sales 2015–2017 | Transaction records | Fact |
| Returns | Return transactions | Fact |
| Customers | Customer demographics | Dimension |
| Products | Product catalog | Dimension |
| Product Categories | Product classification | Dimension |
| Product Subcategories | Product sub-classification | Dimension |
| Territories | Geographic regions | Dimension |
| Calendar | Date dimension | Dimension |

---

## Project Structure

```
Adventure-Works-Data-Engineering-Project/
├── data/
│   ├── AdventureWorks_Calendar.csv
│   ├── AdventureWorks_Customers.csv
│   ├── AdventureWorks_Products.csv
│   ├── AdventureWorks_Product_Categories.csv
│   ├── AdventureWorks_Product_Subcategories.csv
│   ├── AdventureWorks_Returns.csv
│   ├── AdventureWorks_Sales_2015.csv
│   ├── AdventureWorks_Sales_2016.csv
│   ├── AdventureWorks_Sales_2017.csv
│   └── AdventureWorks_Territories.csv
├── git.json                  # ADF pipeline parameter file
└── README.md
```

---

## Implementation Details

### 1. Data Ingestion (Bronze Layer)

Built two ADF pipelines:

- **Static pipeline** — single file copy for initial testing
- **Dynamic pipeline** — parameterized ForEach loop reading from `git.json`, copying all 10 source files to the bronze container in one execution

Key ADF concepts applied:
- Linked Services (HTTP + Azure Data Lake Gen2)
- Parameterized Datasets
- Lookup Activity → ForEach Activity → Copy Activity

### 2. Data Transformation (Silver Layer)

Used Azure Databricks with PySpark to:

- Read CSV files from the bronze container using ABFS protocol
- Apply transformations: date extraction, string splitting, column concatenation, type casting, regex replacement
- Write cleaned data to the silver container in Parquet format (columnar, compressed)

Authentication between Databricks and Data Lake was handled via **Managed Identity**, eliminating the need for hardcoded credentials.

### 3. Data Warehousing (Gold Layer)

Created a Serverless SQL Pool in Azure Synapse Analytics implementing the **Lakehouse pattern**:

- Data physically resides in Azure Data Lake (low cost)
- SQL abstraction layer enables standard query access
- Created a `gold` schema with views over each silver table using `OPENROWSET()`
- Created External Tables using CETAS (Create External Table As Select) to persist query results in Parquet format

```sql
-- Example: Creating a view over silver layer
CREATE OR ALTER VIEW gold.sales
AS
SELECT *
FROM OPENROWSET(
    BULK 'https://<storage>.dfs.core.windows.net/silver/Sales/',
    FORMAT = 'PARQUET'
) AS result;
```

```sql
-- Example: Creating an external table
CREATE EXTERNAL TABLE gold.ext_sales
WITH (
    LOCATION = 'ext_sales',
    DATA_SOURCE = source_gold,
    FILE_FORMAT = format_parquet
)
AS
SELECT * FROM gold.sales;
```

### 4. Reporting (Power BI)

Connected Power BI to Synapse Analytics via the **Serverless SQL Endpoint**, querying the gold layer external tables directly without data movement.

---

## Key Design Decisions

**Why Medallion Architecture?**
Separating raw, cleaned, and serving layers ensures data lineage, allows reprocessing at any stage, and maintains a single source of truth.

**Why Managed Identity over Service Principal?**
Managed Identity eliminates credential management overhead and reduces security risk. Azure handles token rotation automatically.

**Why Serverless SQL Pool over Dedicated Pool?**
The dataset size does not justify a dedicated pool. Serverless scales on demand and charges only per query — appropriate for intermittent analytical workloads.

**Why Parquet in Silver/Gold?**
Parquet is a columnar format optimised for analytical queries. Compared to CSV, it offers significant performance improvements for aggregation-heavy workloads and supports schema enforcement.

---

## Setup

### Prerequisites
- Azure subscription (Azure for Students or Pay-As-You-Go)
- Power BI account

### Steps

1. Create a Resource Group in Azure Portal
2. Create an Azure Data Lake Gen2 storage account with containers: `bronze`, `silver`, `gold`
3. Create an Azure Data Factory instance and configure Linked Services
4. Upload `git.json` to a `parameters` container in the Data Lake
5. Build and run the dynamic ADF pipeline
6. Create an Azure Databricks workspace, configure cluster, run the silver layer notebook
7. Create an Azure Synapse Analytics workspace
8. Assign `Storage Blob Data Contributor` role to the Synapse Managed Identity on the Data Lake
9. Create `aw_database` (Serverless), run SQL scripts to create schema, views, and external tables
10. Connect Power BI using the Serverless SQL endpoint

---

## Notes

- No credentials, access keys, or connection strings are committed to this repository
- All inter-service authentication uses Azure Managed Identity
- The `git.json` file serves as the parameter configuration for the ADF dynamic pipeline