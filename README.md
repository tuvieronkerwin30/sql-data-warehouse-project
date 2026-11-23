# 📊 SQL Data Warehouse & Tableau Analytics Project

An end-to-end data engineering and business intelligence portfolio project. This repository demonstrates how raw, fragmented **CRM** and **ERP** datasets are ingested, cleansed, and modeled using a SQL **Medallion Architecture (Bronze → Silver → Gold)** in **SQL Server** (`DataWarehouse`) to power an interactive **Tableau** executive dashboard.

---

## 🎯 Dashboard Showcase

[![Tableau Dashboard Preview](docs/dashboard_preview.png)](https://public.tableau.com/app/profile/kerwin.tuvieron/vizzes)

> **🔗 Live Interactive Dashboard:** [Explore the interactive visualization on Tableau Public](https://public.tableau.com/app/profile/kerwin.tuvieron/vizzes)

### Key Business Insights Highlighted
* **Sales & Revenue Performance:** Tracks overall revenue (`sales_amount`), order volume (`quantity`), and pricing trends (`price`) across order, shipping, and due timelines.
* **Customer Demographics & Behavior:** Segments customers by `country`, `gender`, `marital_status`, and age group derived from `birthdate`.
* **Product & Category Breakdown:** Evaluates profitability and performance across `product_line`, `category`, `subcategory`, `cost`, and `maintenance` classifications.

---

## 🏗️ Data Architecture

The pipeline processes six raw CSV extracts across two enterprise source systems (**CRM** and **ERP**) and refines them through three distinct architectural layers inside the `DataWarehouse` database:

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
   * Aligns cross-system keys (e.g., matching CRM `cst_key` with ERP `CID`, and parsing category prefixes from `prd_key` to join with `PX_CAT_G1V2.ID`), standardizes coded values (`M`/`F`, marital status, country names), and casts integer/string dates into proper `DATE` types.
3. **🥇 Gold Layer (`scripts/gold/`):**
   * Exposes business-ready **Fact** and **Dimension** views (`gold.fact_sales`, `gold.dim_customers`, `gold.dim_products`) with surrogate keys (`customer_key`, `product_key`) structured in a **Star Schema** optimized for Tableau.
   * Validated via automated SQL quality checks (`quality_checks_gold.sql` / `tests/`) before visualization.

---

## ⭐ Gold Layer Data Model (Star Schema)

The Gold layer transforms raw source tables into three analytics-ready views consumed directly by Tableau:

```text
       +------------------------+                 +------------------------+
       |   gold.dim_customers   |                 |   gold.dim_products    |
       +------------------------+                 +------------------------+
       | PK customer_key        |                 | PK product_key         |
       |    customer_id         |                 |    product_id          |
       |    customer_number     |                 |    product_number      |
       |    first_name          |                 |    product_name        |
       |    last_name           |                 |    category_id         |
       |    country             |                 |    category            |
       |    marital_status      |                 |    subcategory         |
       |    gender              |                 |    maintenance         |
       |    birthdate           |                 |    cost                |
       |    create_date         |                 |    product_line        |
       +-----------+------------+                 |    start_date          |
                   |                              +-----------+------------+
                   | 1                                        | 1
                   |                                          |
                   | *          +--------------------+        | *
                   +----------->|  gold.fact_sales   |<-------+
                                +--------------------+
                                |    order_number    |
                                | FK product_key     |
                                | FK customer_key    |
                                |    order_date      |
                                |    shipping_date   |
                                |    due_date        |
                                |    sales_amount    |
                                |    quantity        |
                                |    price           |
                                +--------------------+
```

### Gold Views Data Dictionary

* **`gold.fact_sales`** *(Fact Table)*:
  `order_number`, `product_key`, `customer_key`, `order_date`, `shipping_date`, `due_date`, `sales_amount`, `quantity`, `price`
* **`gold.dim_customers`** *(Dimension Table — integrates `cust_info`, `CUST_AZ12`, and `LOC_A101`)*:
  `customer_key`, `customer_id`, `customer_number`, `first_name`, `last_name`, `country`, `marital_status`, `gender`, `birthdate`, `create_date`
* **`gold.dim_products`** *(Dimension Table — integrates `prd_info` and `PX_CAT_G1V2`)*:
  `product_key`, `product_id`, `product_number`, `product_name`, `category_id`, `category`, `subcategory`, `maintenance`, `cost`, `product_line`, `start_date`

---

## 🗂️ Source Datasets & Column Schema

### 1. CRM Source (`datasets/source_crm/`)
| File Name | Columns | Description |
| :--- | :--- | :--- |
| **`cust_info.csv`** | `cst_id`, `cst_key`, `cst_firstname`, `cst_lastname`, `cst_marital_status`, `cst_gndr`, `cst_create_date` | Core customer master records, personal details, and account creation dates. |
| **`prd_info.csv`** | `prd_id`, `prd_key`, `prd_nm`, `prd_cost`, `prd_line`, `prd_start_dt`, `prd_end_dt` | Product catalog including costs, product lines, and historical effective dates. |
| **`sales_details.csv`** | `sls_ord_num`, `sls_prd_key`, `sls_cust_id`, `sls_order_dt`, `sls_ship_dt`, `sls_due_dt`, `sls_sales`, `sls_quantity`, `sls_price` | Transactional sales orders linking customers and products with order/ship/due dates and financials. |

### 2. ERP Source (`datasets/source_erp/`)
| File Name | Columns | Description |
| :--- | :--- | :--- |
| **`CUST_AZ12.csv`** | `CID`, `BDATE`, `GEN` | Supplementary ERP customer demographics (birth dates and gender). |
| **`LOC_A101.csv`** | `CID`, `CNTRY` | Customer geographic location and country mapping. |
| **`PX_CAT_G1V2.csv`** | `ID`, `CAT`, `SUBCAT`, `MAINTENANCE` | Product category hierarchy and maintenance classifications. |

---

## 📁 Repository Structure

```text
sql-data-warehouse-project/
├── datasets/
│   ├── source_crm/               # Raw CRM extracts (cust_info, prd_info, sales_details)
│   └── source_erp/               # Raw ERP extracts (CUST_AZ12, LOC_A101, PX_CAT_G1V2)
├── docs/                         # Architecture diagrams, data dictionary & dashboard images
├── scripts/
│   ├── init_database.sql         # Creates DataWarehouse DB and bronze/silver/gold schemas
│   ├── bronze/                   # DDL and bulk load procedures for raw tables
│   ├── silver/                   # Data cleansing and transformation scripts
│   └── gold/                     # Analytical views and Star Schema models
├── tests/                        # Data quality and integrity validation scripts
├── LICENSE
└── README.md
```

---

## 🛠️ Tech Stack

* **Database & ETL:** SQL Server (T-SQL), Stored Procedures, Views
* **Data Modeling:** Medallion Architecture (Bronze/Silver/Gold), Star Schema Dimensional Modeling
* **Data Quality:** Custom SQL validation scripts (surrogate key checks, referential integrity, null/duplicate detection)
* **BI & Visualization:** Tableau Desktop / Tableau Public

---

## 🚀 How to Reproduce Locally

1. **Clone the Repository**
   ```bash
   git clone https://github.com/tuvieronkerwin30/sql-data-warehouse-project.git
   cd sql-data-warehouse-project
   ```
2. **Initialize the Database**
   * Execute `scripts/init_database.sql` in SQL Server Management Studio (SSMS) to create the `DataWarehouse` database and the `bronze`, `silver`, and `gold` schemas.
3. **Load the Bronze Layer**
   * Update the local CSV file paths inside `scripts/bronze/proc_load_bronze.sql`, then execute the stored procedure to ingest raw data.
4. **Run Silver Transformations**
   * Execute the scripts inside `scripts/silver/` to cleanse, standardize, and populate the Silver tables.
5. **Build the Gold Layer & Run Quality Checks**
   * Run the scripts inside `scripts/gold/` to generate `gold.dim_customers`, `gold.dim_products`, and `gold.fact_sales`, then execute the scripts in `tests/` to verify data integrity.
6. **Connect Tableau**
   * Connect Tableau directly to the `DataWarehouse` database (`gold` views) or export the views to CSV to explore and customize the dashboard.

---

## 📄 License

This project is licensed under the MIT License — see the [LICENSE](LICENSE) file for details.

## Acknowledgments

* The data warehouse architecture, medallion pipeline design, and source datasets are inspired by [Data with Baraa](https://www.youtube.com/@DataWithBaraa)'s SQL Data Warehouse project.
* Extended with custom Tableau data modeling and interactive dashboard visualizations.
