<img width="1549" height="1019" alt="data_architecture" src="https://github.com/user-attachments/assets/d131e8a3-8ab2-454b-ba76-0c185e269744" />


🏭 Azure SQL Data Warehouse — End-to-End Data Engineering Project

Show Image Show Image Show Image Show Image Show Image Show Image

📑 Table of Contents
Project Overview
Data Architecture
Project Structure
Tools & Technologies
Data Sources
Data Flow — Layer by Layer
Bronze Layer
Silver Layer
Gold Layer
Data Model — Star Schema
ADF Pipeline Orchestration
Naming Conventions
Data Catalog
Power BI Dashboard
How to Run
Author
📌 Project Overview

An end-to-end data warehouse built from scratch on Azure SQL Database, implementing the Medallion Architecture (Bronze → Silver → Gold) using T-SQL stored procedures and orchestrated via Azure Data Factory.

The project simulates a real-world scenario where a retail business receives raw data from two source systems — a CRM system (customer and product data) and an ERP system (sales transactions) — and needs a clean, analytics-ready data warehouse to power business reporting.

Key engineering decisions demonstrated:

Medallion Architecture on Azure SQL Database (not SQL Server — fully cloud-native)
Stored procedures for modular, reusable ETL logic
Star schema dimensional modelling for optimised analytical queries
ADF pipelines for automated orchestration of all layers
Naming conventions and data catalog for production-grade documentation
📐 Data Architecture

Show Image

The pipeline follows a three-layer Medallion Architecture within Azure SQL Database, orchestrated end-to-end by Azure Data Factory:

Layer	Purpose	Load Pattern
Bronze	Raw data — as-is from source	Full Load, Truncate & Insert
Silver	Cleaned, standardised data	Full Load, Truncate & Insert
Gold	Business-ready star schema	Views — No Load
📂 Project Structure
azure-sql-data-warehouse/
│
├── datasets/                        # Source CSV files (CRM + ERP)
│   ├── crm_cust_info.csv
│   ├── crm_prd_info.csv
│   ├── crm_sales_details.csv
│   ├── erp_cust_az12.csv
│   ├── erp_loc_a101.csv
│   └── erp_px_cat_g1v2.csv
│
├── docs/
│   ├── data_architecture.png        # High-level architecture diagram
│   ├── data_model_star_schema.png   # Star schema ERD
│   ├── data_integration.png         # Data integration flow
│   ├── data_layers.pdf              # Layer-by-layer documentation
│   ├── data_catalog.md              # Column-level data dictionary
│   └── naming_conventions.md        # Naming standards used throughout
│
├── scripts/
│   ├── init_database.sql            # Database and schema initialisation
│   ├── bronze/                      # DDL + load stored procedures (Bronze)
│   ├── silver/                      # Cleansing stored procedures (Silver)
│   └── gold/                        # Star schema views (Gold)
│
├── adf_pipelines/                   # Azure Data Factory pipeline JSON exports
│
├── tests/                           # Data quality validation SQL scripts
│
└── README.md
🛠️ Tools & Technologies
Tool	Purpose
Azure SQL Database	Cloud-native data warehouse — Bronze, Silver, Gold layers
T-SQL	DDL, stored procedures, data cleansing, star schema views
Azure Data Factory	Pipeline orchestration — automates Bronze → Silver → Gold
Power BI	Business intelligence dashboard connected to Gold layer
Draw.io	Architecture and data model diagrams
SSMS / Azure Data Studio	Query development and testing
Git	Version control
📁 Data Sources

Two source systems feed the pipeline via CSV files:

CRM System

File	Contents
crm_cust_info.csv	Customer demographics and profile data
crm_prd_info.csv	Product catalogue and pricing
crm_sales_details.csv	Sales order transactions

ERP System

File	Contents
erp_cust_az12.csv	Customer records from ERP
erp_loc_a101.csv	Location and geography data
erp_px_cat_g1v2.csv	Product category hierarchy
⚙️ Data Flow — Layer by Layer
Bronze Layer — Raw Ingestion

Scripts: scripts/bronze/

Raw CSV data is loaded directly into Bronze tables with no transformations — exactly as received from source systems. Each load uses Truncate & Insert for a clean full refresh.

sql
-- Pattern used in Bronze stored procedures
TRUNCATE TABLE bronze.crm_cust_info;
BULK INSERT bronze.crm_cust_info
FROM 'path/to/crm_cust_info.csv'
WITH (FIRSTROW = 2, FIELDTERMINATOR = ',', TABLOCK);

What Bronze stores:

Raw string values — no type casting
All source columns preserved as-is
No deduplication or null handling
Silver Layer — Cleansing & Standardisation

Scripts: scripts/silver/

Silver stored procedures read from Bronze and apply data quality rules before writing to Silver tables:

Issue	Handling
Duplicate records	Deduplication using ROW_NUMBER() window function
Inconsistent gender values	Standardised to 'M' / 'F' / 'n/a'
Invalid date formats	Cast and validated to DATE type
NULL or blank values	Replaced with meaningful defaults
Whitespace in strings	Trimmed using TRIM()
Mismatched product keys	Resolved via cross-source join logic
Gold Layer — Star Schema

Scripts: scripts/gold/

Gold layer objects are views built on top of Silver tables — no physical data movement. Views are recalculated on every query, ensuring the Gold layer always reflects the latest Silver data.

⭐ Data Model — Star Schema

Show Image

The Gold layer implements a Sales Data Mart using a classic star schema:

        [gold.dim_customers]
               │
               │ customer_key
               │
[gold.dim_products]──[gold.fact_sales]──[gold.dim_date]
               
               │ 
               │
        [gold.dim_location]

Fact Table:

gold.fact_sales — one row per sales order line

order_date, shipping_date, due_date
sales_amount, quantity, price
Foreign keys: customer_key, product_key

Dimension Tables:

Table	Description
gold.dim_customers	Customer demographics — name, gender, marital status, country
gold.dim_products	Product details — name, category, subcategory, cost, line
gold.dim_date	Date dimension — year, month, quarter, week number
🔄 ADF Pipeline Orchestration

Pipelines: adf_pipelines/

Azure Data Factory automates the full pipeline execution:

Trigger → Bronze Pipeline → Silver Pipeline → Gold Pipeline → Power BI Refresh

Each pipeline calls the relevant stored procedures in sequence. Dependency chains ensure Silver only runs after Bronze completes successfully, and Gold only runs after Silver.

📋 Naming Conventions

Full reference: docs/naming_conventions.md

Object	Convention	Example
Bronze tables	bronze.source_tablename	bronze.crm_cust_info
Silver tables	silver.source_tablename	silver.crm_cust_info
Gold views	gold.dim_entity / gold.fact_entity	gold.dim_customers
Stored procedures	load_layername_tablename	load_silver_crm_cust_info
Primary keys	entity_key	customer_key
📖 Data Catalog

Full catalog: docs/data_catalog.md

The data catalog documents every column in the Gold layer — data type, description, and example values — so analysts can self-serve without needing to read the SQL.

📊 Power BI Dashboard

🚧 In Progress — connecting to Azure SQL Database Gold layer views.

Planned dashboard pages:

Executive Overview — Revenue KPIs, YoY growth, MoM trends
Sales Performance — Revenue by product, category, and time period
Customer Insights — Top customers, segmentation, geographic distribution
Product Analysis — Best sellers, category performance, pricing analysis

DAX measures planned:

Total Revenue, Revenue YTD, Revenue PY, YoY %
Rolling 3-Month Average
Top N Products using RANKX
Dynamic metric switcher using Field Parameters
🚀 How to Run

Prerequisites: Azure subscription, Azure SQL Database, SSMS or Azure Data Studio, Azure Data Factory

Initialise the database
sql
-- Run in SSMS connected to your Azure SQL Database
scripts/init_database.sql
Load Bronze layer
sql
EXEC load_bronze;
Load Silver layer
sql
EXEC load_silver;
Gold layer views are created once and auto-refresh
sql
-- Run once to create views
scripts/gold/ddl_gold.sql
Deploy ADF pipelines
Import JSON files from adf_pipelines/ into your ADF workspace
Update linked service connection strings
Trigger the master pipeline
Connect Power BI
Open Power BI Desktop
Get Data → Azure SQL Database
Connect to your server and import Gold layer views
👩‍💻 Author

Pranusha Tirunagari
Senior Data Engineer | Azure • Databricks • Synapse • Power BI
📍 LinkedIn | GitHub
