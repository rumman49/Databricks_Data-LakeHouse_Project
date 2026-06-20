# 🏗️ Databricks Medallion Architecture Project

An end-to-end **Data Engineering** project built on **Azure Databricks** following the **Medallion Architecture** (Bronze → Silver → Gold). Raw CSV source files are ingested, cleaned, and transformed into analytics-ready Delta tables using **PySpark** and **Spark SQL**.

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| Azure Databricks | Data processing & notebook environment |
| Azure Data Lake Storage Gen2 (ADLS) | Raw source file storage (`/Volumes/lakehouse/bronze/source_files/`) |
| PySpark | Data transformation & cleaning |
| Spark SQL | Querying, renaming, and normalizing columns |
| Delta Lake | ACID-compliant table storage across all layers |

---

## 🏛️ Architecture

```
ADLS Gen2 (CSV Source Files)
           │
           │  File Arrival Trigger
           ▼
   ┌───────────────┐
   │  Bronze Layer │  →  Raw ingestion → Delta tables (no transformation)
   └───────────────┘
           │
           ▼
   ┌───────────────┐
   │  Silver Layer │  →  Cleaned, normalized, typed, null-handled
   └───────────────┘
           │
           ▼
   ┌───────────────┐
   │   Gold Layer  │  →  Star schema: dim + fact tables, join-validated
   └───────────────┘
```

---

## ⚙️ Pipeline Orchestration

The entire pipeline is orchestrated as a **Databricks Job** named `Medallion Architecture - DE` with all 3 layers running as sequential tasks:

```
Bronze_Layer  →  Silver_Layer  →  Gold_Layer
```

**Trigger:** File Arrival on `/Volumes/lakehouse/bronze/source_files/`  
— the pipeline automatically triggers whenever new source files land in ADLS.

**Compute:** Single node cluster — `Standard_D4ds_v4` · Databricks Runtime `14.3.52`

**Run history:** Multiple successful end-to-end runs completed (~2 min avg duration)

---

## 📁 Project Structure

```
Databricks_DataLakeHouse_Project/
│
├── Bronze-NB.ipynb       # Ingest CSVs from ADLS → Bronze Delta tables
├── Silver-NB.ipynb       # Clean & transform Bronze → Silver Delta tables
├── Gold-NB.ipynb         # Build Star Schema → Gold Delta tables
└── README.md
```

---

## 📄 Notebooks Breakdown

### 🥉 Bronze — `Bronze-NB.ipynb`
Reads raw CSV files from ADLS Volume and writes them as Delta tables into `lakehouse.bronze`:

| Source File | Bronze Table |
|---|---|
| `cust_info.csv` | `lakehouse.bronze.cust_info` |
| `prd_info_1.csv` | `lakehouse.bronze.prd_info_1` |
| `sales_details.csv` | `lakehouse.bronze.sales_details` |

---

### 🥈 Silver — `Silver-NB.ipynb`
Applies data cleaning and transformation on all three Bronze tables:

**cust_info → `lakehouse.silver.s_cust_info`**
- Trimmed all string columns
- Renamed columns (e.g. `cst_id` → `cust_id`, `cst_gndr` → `gender`)
- Normalized `marital_status`: `M` → `Married`, else `Single`
- Normalized `gender`: `M` → `Male`, else `Female`
- Filtered out null `cust_id` rows
- Filled missing `first_name` / `last_name` with `'Unknown'`

**prd_info_1 → `lakehouse.silver.s_prd_info_1`**
- Filled null `prd_cost` with median value (`202`)
- Filled null `prd_line` with mode value (`'R'`)
- Trimmed `prd_line` column
- Renamed columns using Spark SQL

**sales_details → `lakehouse.silver.s_sales_details`**
- Replaced negative/zero `sls_sales` with median (`30`)
- Recalculated `sls_price` as `sls_sales × sls_quantity`
- Converted date columns (`sls_order_dt`, `sls_ship_dt`, `sls_due_dt`) from `yyyyMMdd` integer format to proper `DateType`

---

### 🥇 Gold — `Gold-NB.ipynb`
Builds the final **Star Schema** from Silver tables and writes to `lakehouse.gold`:

| Table | Gold Table | Key Columns |
|---|---|---|
| dim_customer | `lakehouse.gold.dim_customer` | `cust_id`, `cust_key` |
| dim_product | `lakehouse.gold.dim_product` | `prd_id`, `prd_key` |
| fact_sales | `lakehouse.gold.fact_sales` | `sls_prd_key`, `sls_cust_id` |

Join validation using `left_anti` joins confirmed **zero unmatched rows** across both dimension tables.

---

## 👤 Author

**Romaan Uddin Siddiqui**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-romaan--siddiqui-blue)](https://linkedin.com/in/romaan-siddiqui)
[![GitHub](https://img.shields.io/badge/GitHub-rumman49-black)](https://github.com/rumman49)
