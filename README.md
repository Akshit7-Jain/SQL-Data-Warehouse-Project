# 📦 SQL Data Warehouse Project

A modern data warehouse built in SQL Server, designed to consolidate sales data from CRM and ERP source systems into a clean, analytics-ready model using the Medallion Architecture (Bronze → Silver → Gold).

This project covers the full pipeline: raw data ingestion, cleansing and standardization, and dimensional modeling for BI and reporting.

---
## 🎯 Objective

Develop a modern data warehouse using SQL Server to consolidate sales data, enabling analytical reporting and informed decision-making — covering both the data engineering (warehouse build) and data analysis (BI/reporting) sides of the project.

---
## 🏗️ Data Architecture

The project follows the Medallion Architecture with three layers:

🥉 BronzeRaw, unprocessed data ingested as-is from source CSV files (CRM & ERP)  <br>
🥈 SilverCleansed, standardized, and business-rule-validated data <br>
🥇 GoldBusiness-ready, dimensional model (star schema) exposed as views for reporting <br>

![Dashboard Preview](https://github.com/Akshit7-Jain/SQL-Data-Warehouse-Project/blob/main/Docs/High%20level%20Architecture.png)

```
Source Files (CSV)  →  Bronze (Raw)  →  Silver (Cleansed)  →  Gold (Star Schema / Views)
   CRM + ERP             Staging          Business Rules         Reporting Layer
```

---
### 📋 Specifications


- **Data Sources:** Two source systems — CRM and ERP — provided as CSV files
- **Data Quality:** Issues identified and resolved during the Bronze → Silver transformation
- **Integration:** CRM and ERP sources combined into a single, unified data model
- **Scope:** Latest dataset snapshot only — historical tracking not required
- **Documentation:** Data model documented for both business stakeholders and analytics teams


---
## 🗂️ Repository Structure
```
sql-data-warehouse-project/
│
├── datasets/
│   ├── source_crm/          # CRM CSV files (cust_info, prd_info, sales_details)
│   └── source_erp/          # ERP CSV files (loc_a101, cust_az12, px_cat_g1v2)
│
├── scripts/
│   ├── bronze/               # DDL + stored procedure to load raw data
│   ├── silver/               # DDL + stored procedure with cleansing logic
│   └── gold/                 # Views forming the star schema
│
└── README.md
```


### 🔧 Bronze Layer — Raw Ingestion
---

- Raw tables created per source file: crm_cust_info, crm_prd_info, crm_sales_details, erp_loc_a101, erp_cust_az12, erp_px_cat_g1v2
- Loaded via a reusable stored procedure (bronze.load_bronze) using BULK INSERT
- Tables are truncated and reloaded on each run (full load, no incremental logic)
- Includes error handling (TRY...CATCH) and ETL duration logging for each table and the overall batch



### 🧹 Silver Layer — Cleansing & Standardization
---

Each Bronze table is transformed into its Silver counterpart, with an added dwh_create_date column to track load time. Key transformations include:


1.  **Deduplication:** ROW_NUMBER() partitioned by customer ID to keep only the latest record

2.  **Whitespace handling:** TRIM() applied across text fields

3.  **Standardized categorical values:** e.g. marital status (S/M → Single/Married), gender (M/F → Male/Female), country codes (US/USA → United States)

4.  **Key normalization:** product keys split into category ID and product key; customer IDs stripped of prefixes (e.g. NAS prefix removed)

5.  **Date validation & casting:** integer date fields (YYYYMMDD) converted to proper DATE type, invalid values set to NULL


Business rule enforcement:

Product end_date derived using LEAD() when missing or inconsistent
- **sales = quantity × price** validated and recalculated when null, negative, or inconsistent
- **price = sales / quantity** recalculated when missing or invalid
- **Future/invalid birth dates** set to NULL



Implemented in a reusable stored procedure (silver.load_bronze) with error handling and duration logging



### 🥇 Gold Layer — Business-Ready Star Schema
---

**Exposed as SQL views for direct consumption by BI tools and analysts:**

View-Type Description gold.dim_customers Dimension Unified customer profile (CRM + ERP demographic & location data)gold.dim_products Dimension Current product catalog with category and subcategory enrichment gold.fact_sales. FactSales transactions linked to customer and product dimension keys


- Surrogate keys generated via ROW_NUMBER()
- CRM treated as the primary source of truth for fields like gender, with ERP data used as fallback
- Historical/inactive products filtered out of the product dimension


---
## 📊 BI: Analytics & Reporting

Built on top of the Gold layer to deliver insights into:


- **Customer Behavior** — demographics, purchase patterns
- **Product Performance** — sales by category, line, and product
- **Sales Trends** — revenue and order trends over time


---
### 🛠️ Tools Used


- **SQL Server / SSMS** — database engine and development environment
- **T-SQL** — DDL, stored procedures, window functions, views
- **CSV** — raw source file format from CRM and ERP systems


---
## ▶️ How to Run


- Run the database & schema creation script (bronze, silver, gold schemas) <br>
- Execute EXEC bronze.load_bronze; to ingest raw CSV data<br>
- Execute EXEC silver.load_bronze; to cleanse and load the Silver layer <br>
- Gold layer views are created automatically and ready to query <br>
 

---
## 📌 Notes


This project focuses on the latest snapshot of data only — no slowly changing dimension (SCD) logic is implemented. <br>
File paths in the BULK INSERT statements are local paths and will need to be updated to match your environment.


---
## 👤 Author

Akshit Jain B.Com Graduate | Data & Business Analytics enthusiast
