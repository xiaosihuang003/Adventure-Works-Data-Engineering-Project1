# 🏗️ Azure End-to-End Data Engineering Project

## 📋 Project Overview
Built a complete data engineering pipeline using 
Microsoft Azure cloud services, implementing the 
Medallion Architecture (Bronze → Silver → Gold).

## 🏛️ Architecture
GitHub (Source)
    ↓
Azure Data Factory (Ingestion)
    ↓
🥉 Bronze Layer (Raw Data)
    ↓
🥈 Silver Layer (Transformed Data)
    ↓
🥇 Gold Layer (Analytics Ready)
    ↓
Power BI (Visualization)

## 🛠️ Technologies Used
- **Azure Data Factory** - Data pipeline orchestration
- **Azure Data Lake Gen2** - Cloud storage
- **Azure Databricks** - Data transformation (PySpark)
- **Azure Synapse Analytics** - Data warehousing
- **Power BI** - Data visualization
- **PySpark** - Big data processing

## 📊 Dataset
Adventure Works dataset containing:
- Sales data (2015-2017)
- Customer information
- Product catalog
- Territory data

## 🚀 Project Highlights
- Built dynamic ADF pipelines with parameterization
- Implemented Medallion Architecture
- Used Managed Identity for secure authentication
- Created Serverless SQL views and external tables
- Connected to Power BI for reporting

## 📁 Project Structure
├── data/                    # Source CSV files
│   ├── AdventureWorks_Calendar.csv
│   ├── AdventureWorks_Customers.csv
│   ├── AdventureWorks_Products.csv
│   └── ...
└── git.json                 # ADF pipeline configuration


## 🔧 Setup Instructions
1. Create Azure Resource Group
2. Create Azure Data Lake Gen2
3. Create Azure Data Factory
4. Configure Linked Services
5. Run Dynamic Pipeline
6. Setup Databricks cluster
7. Create Synapse Analytics workspace
8. Connect Power BI