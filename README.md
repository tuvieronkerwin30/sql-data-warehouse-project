# 📊 SQL Data Warehouse & Tableau Analytics Project

An end-to-end data engineering and business intelligence portfolio project. This repository demonstrates how raw, fragmented **CRM** and **ERP** datasets are ingested, cleansed, and modeled using a SQL **Medallion Architecture (Bronze → Silver → Gold)** to power an interactive **Tableau** executive dashboard.

---

## 🎯 Dashboard Showcase

[![Tableau Executive Dashboard](docs/dashboard_preview.png)](https://public.tableau.com/app/profile/YOUR_PROFILE_LINK)

> **🔗 Live Interactive Dashboard:** [Explore the interactive visualization on Tableau Public](https://public.tableau.com/app/profile/YOUR_PROFILE_LINK)

### Key Business Insights Highlighted
* **Sales & Revenue Performance:** Tracks overall revenue trends, order volume, and seasonal performance across product lines.
* **Customer Demographics & Behavior:** Segments customers by country, gender, age group, and purchasing frequency.
* **Product Category Breakdown:** Identifies top-performing categories, subcategories, and underperforming SKUs.

---

## 🏗️ Data Architecture

The pipeline processes six raw CSV extracts across two enterprise source systems (**CRM** and **ERP**) and refines them through three distinct architectural layers:

```text
+-------------------+       +-------------------+       +-------------------+       +-------------------+
|   Source Systems  |       |   Bronze Layer    |       |   Silver Layer    |       |    Gold Layer     |
|  (CRM & ERP CSVs) | ----> |  (Raw Ingestion)  | ----> | (Cleansed & Norm) | ----> | (Star Schema / BI)|
+-------------------+       +-------------------+       +-------------------+       +-------------------+
                                                                                              |
                                                                                              v
                                                                                    +-------------------+
                                                                                    | Tableau Dashboard |
                                                                                    +-------------------+
```

1. **🥉 Bronze Layer (`scripts/bronze/`):** 
   * Ingests raw data as-is from `source_crm` and `source_erp` into staging tables using stored procedures (`proc_load_bronze.sql`).
   * Preserves an exact historical snapshot of source files for traceability and debugging.
2. **🥈 Silver Layer (`scripts/silver/`):** 
   * Cleanses, standardizes, and deduplicates records.
   * Resolves formatting inconsistencies (dates, trimmed strings, standardized gender/marital status codes), handles null/invalid entries, and aligns foreign keys across CRM and ERP sources.
3. **🥇 Gold Layer (`scripts/gold/`):** 
   * Models business-ready **Fact** and **Dimension** views (`dim_customers`, `dim_products`, `fact_sales`) structured in a Star Schema optimized for Tableau queries.
   * Validated via automated SQL quality checks (`tests/`) before visualization.

---

## 📁 Repository Structure

```text
sql-data-warehouse-project/
├── datasets/
│   ├── source_crm/               # Raw CRM extracts
│   │   ├── cust_info.csv
│   │   ├── prd_info.csv
│   │   └── sales_details.csv
│   └── source_erp/               # Raw ERP extracts
│       ├── CUST_AZ12.csv
│       ├── LOC_A101.csv
│       └── PX_CAT_G1V2.csv
├── docs/                         # Architecture diagrams, data dictionary & dashboard images
├── scripts/
│   ├── init_database.sql         # Creates database and bronze/silver/gold schemas
│   ├── bronze/                   # DDL and bulk load procedures for raw tables
│   ├── silver/                   # Data cleansing and transformation scripts
│   └── gold/                     # Analytical views and Star Schema models
├── tests/                        # Data quality and integrity validation scripts
├── LICENSE
└── README.md
```

---

## 🛠️ Tech Stack

* **Database & ETL:** SQL (T-SQL / SQL Server)
* **Data Modeling:** Medallion Architecture, Star Schema Dimensional Modeling
* **Data Quality:** Custom SQL validation scripts (referential integrity, null checks, duplicate detection)
* **BI & Visualization:** Tableau Desktop / Tableau Public

---

## 🚀 How to Reproduce Locally

1. **Clone the Repository**
   ```bash
   git clone https://github.com/tuvieronkerwin30/sql-data-warehouse-project.git
   cd sql-data-warehouse-project
   ```
2. **Initialize the Database**
   * Execute `scripts/init_database.sql` in your SQL environment to set up the database and schemas (`bronze`, `silver`, `gold`).
3. **Load the Bronze Layer**
   * Update the CSV file paths inside `scripts/bronze/proc_load_bronze.sql` to match your local directory, then execute the load procedure.
4. **Run Silver Transformations**
   * Execute the scripts inside `scripts/silver/` to populate the cleansed tables.
5. **Build the Gold Layer & Run Quality Checks**
   * Run the scripts inside `scripts/gold/` to generate the final views, then execute the scripts in `tests/` to verify data integrity.
6. **Connect Tableau**
   * Connect Tableau to your `gold` schema views (or export them to CSV) to view or customize the dashboard.

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.
