# M17: Load Data into a Microsoft Fabric Data Warehouse

Learn how to load data into a Fabric warehouse using Data Pipelines, T-SQL, and Dataflows Gen2, exploring strategies for full loads, incremental loads, surrogate key generation, and Slowly Changing Dimensions (SCD).

## Learning Objectives
In this module, you learn how to:
- Identify when to use **Full Loads** vs. **Incremental Loads**.
- Describe how **Copy jobs** in data pipelines ingest data into a warehouse.
- Explain how T-SQL `COPY INTO`, `CTAS` (`CREATE TABLE AS SELECT`), and `INSERT...SELECT` load data.
- Describe how **Dataflows Gen2** imports and transforms data using Power Query before loading.

---

## 1. Introduction

* **Core Challenge:** Consolidating multi-source enterprise data (cloud CSVs, on-prem SQL Server, web logs) into a Fabric Data Warehouse for daily analytics refreshes.
* **Key Ingestion Methods:**
  1. **Copy Jobs in Data Pipelines:** High-throughput wizard-driven ingestion.
  2. **T-SQL Statements (`COPY INTO`, `CTAS`, `INSERT...SELECT`):** Code-first high-performance loading.
  3. **Dataflow Gen2:** Visual Power Query transformations prior to landing.
* **Key Load Strategies:** Full vs. Incremental loading, Slowly Changing Dimension (SCD) patterns, and surrogate key assignment.

---

## 2. Explore Data Load Strategies

Data warehouse loading follows a strict sequential pipeline: **Ingest -> Stage -> Load Dimensions -> Load Fact Tables**.

### 🔄 Full Load vs. Incremental Load
| Load Pattern | Mechanism | Best Used For |
| :--- | :--- | :--- |
| **Full Load** | Truncates and completely re-populates target tables. No change tracking needed. | Initial DW migration, small lookup tables, or complete periodic refreshes. |
| **Incremental Load** | Processes only new or updated records since the last load timestamp/watermark. Preserves historical records. | Regular ongoing loads (daily/hourly). Faster execution, lower compute consumption. |

### 🛠️ Staging Layer & Key Mapping
* **Staging Purpose:** Landing area (`dbo.StageSales`) to transform, validate data types, and resolve keys *without* affecting production table query performance.
* **Business Key (Natural Key):** Source system identifier (`ProductID`, `CustomerID`). Preserves source lineage.
* **Surrogate Key:** Integer key generated **inside the DW** (`ProductKey`). Decouples warehouse relationships from source system key changes or re-uses.

### 📜 Slowly Changing Dimensions (SCD) Types (HIGH YIELD)
Dimensions change over time (e.g., customer address move, product reclassification).

* **Type 0:** Retain Original. Attributes never change.
* **Type 1 (Overwrite):** Overwrites the existing record in-place. **No history preserved.** Best for correcting data entry errors.
* **Type 2 (Add New Row - MOST COMMON):** Adds a new row for each attribute change. Uses validity dates (`StartDate`, `EndDate`) and status flags (`IsActive = 'True'/'False'`). **Preserves full historical accuracy.**
* **Type 3:** Adds a `PreviousValue` column to the existing row (tracks limited history).
* **Type 6 (Hybrid):** Combines Type 2 (new rows) and Type 3 (separate columns).

#### T-SQL Pattern for Type 2 SCD:
```sql
IF EXISTS (SELECT 1 FROM Dim_Products WHERE SourceKey = @ProductID AND IsActive = 'True')
BEGIN
    -- Expire the current active version
    UPDATE Dim_Products
    SET ValidTo = GETDATE(), IsActive = 'False'
    WHERE SourceKey = @ProductID AND IsActive = 'True';
END

-- Insert the new active version
INSERT INTO Dim_Products (SourceKey, ProductName, StartDate, EndDate, IsActive)
VALUES (@ProductID, @ProductName, GETDATE(), '9999-12-31', 'True');
```

### 📊 Loading Fact Tables
* **Load Order Rule:** **Dimension tables MUST be loaded BEFORE Fact tables.**
* **Key Lookup Logic:** Staged fact data contains business keys (`CustNo`, `ProductID`). Fact load queries perform joins/lookups against Dimension tables to fetch the corresponding **Surrogate Keys** (and current Type 2 SCD version):

```sql
-- Lookup surrogate keys from dimensions during Fact table insert
INSERT INTO dbo.FactSales
SELECT  
    (SELECT MAX(CustomerKey) FROM dbo.DimCustomer WHERE CustomerAlternateKey = stg.CustNo) AS CustomerKey,
    (SELECT MAX(ProductKey) FROM dbo.DimProduct WHERE ProductAlternateKey = stg.ProductID) AS ProductKey,
    stg.OrderNumber,
    stg.OrderQuantity,
    stg.SalesAmount
FROM dbo.StageSales AS stg;
```

---

## 3. Use Data Pipelines to Load a Warehouse

Fabric Data Pipelines provide a visual, low-code orchestration engine (Data Factory) for moving data at scale, chaining activities (Copy, Dataflows, Stored Procedures, Notebooks), and scheduling recurring workflows.

### 📋 Copy Job Wizard
A guided wizard experience specifically designed for high-throughput source-to-destination data movement.

* **Configuration Steps:**
  1. **Select Source:** Choose from OneLake Catalog, Azure Blob Storage, SQL DBs, etc.
  2. **Select Destination:** Choose the target Fabric Data Warehouse.
  3. **Choose Copy Mode:** 
     * **Full Copy:** Reloads all source records on every pipeline execution.
     * **Incremental Copy:** Ingests only new or modified rows since the previous run.
  4. **Column Mapping:** Rename columns, alter target data types, or exclude unneeded fields from the load.
  5. **Run & Automate:** Save and execute. The job compiles into a pipeline Copy activity on the canvas.

### ⏰ Scheduling & Monitoring Pipelines
* **Scheduling:**
  * Configure recurring frequencies (**Hourly, Daily, Weekly**), start/end dates, and time zones via the pipeline toolbar `Schedule` button.
* **Monitoring & Troubleshooting:**
  * Tracks execution run status, activity durations, and error details in the **Monitoring Hub** to catch and resolve ETL failures before downstream Power BI reports refresh.

---

## 4. Load Data Using T-SQL

T-SQL provides direct, high-performance programmatic control over bulk loading and cross-asset data movement.

### 📦 Bulk Ingestion with the `COPY INTO` Statement
High-throughput bulk ingestion from files (CSV, Parquet) in Azure Storage or OneLake directly into warehouse tables.

```sql
-- Bulk load CSV files using SAS Token authentication
COPY INTO dbo.my_table
FROM 'https://myaccount.blob.core.windows.net/mycontainer/folder0/*.csv',
     'https://myaccount.blob.core.windows.net/mycontainer/folder1/'
WITH (
    FILE_TYPE = 'CSV',
    CREDENTIAL = (IDENTITY = 'Shared Access Signature', SECRET = '<SAS_Token>'),
    FIELDTERMINATOR = '|'
);
```

#### Authentication & Parameter Rules
* **Azure Storage (Blob/ADLS Gen2):** Must supply SAS Token or Account Key in `CREDENTIAL`.
* **OneLake Lakehouse Folders:** **No CREDENTIAL parameter needed** — workspace identity is used automatically.
* **Parquet Files:** `FILE_TYPE` parameter can be omitted (auto-inferred from `.parquet` extension).
* **`REJECTED_ROW_LOCATION` (CSV Only):** Routes failed/malformed rows to a separate error storage folder so the overall bulk load does not abort.

### 🌐 Cross-Database & Cross-Asset Loading (3-Part Naming)
Because all Fabric items share a single TDS endpoint, you can query across different Warehouses and Lakehouses using **3-part naming** (`[database].[schema].[table]`):

| T-SQL Command | Behavior | When to Use |
| :--- | :--- | :--- |
| **`CTAS` (`CREATE TABLE AS SELECT`)** | Creates a **new** warehouse table from a query result. | Initial load of transformed/joined datasets. |
| **`INSERT...SELECT`** | Appends rows into an **existing** target table. | Incremental loads into existing staging/dimension/fact tables. |

```sql
-- Join a Warehouse table with a Lakehouse table to create a new DW table (CTAS)
CREATE TABLE [analysis_warehouse].[dbo].[combined_data]
AS
SELECT 
    sales.product_id,
    sales.sales_amount,
    social.sentiment_score
FROM [sales_warehouse].[dbo].[sales_data] AS sales
INNER JOIN [social_lakehouse].[dbo].[social_data] AS social
    ON sales.product_id = social.product_id;
```

---

## 5. Load and Transform Data with Dataflow Gen2

**Dataflow Gen2** provides a visual, low-code Power Query Online interface to connect, clean, reshape, and load data into Fabric targets.

### 🛠️ Key Capabilities & Transformations
* **Copilot Integration:** Generates Power Query M transformation steps from natural language prompts (e.g., *"Calculate TotalRevenue by multiplying Quantity by UnitPrice"*).
* **Wide Destination Support:** Fabric Warehouse, Lakehouse (Tables/Files), KQL Database, SQL Database, ADLS Gen2, SharePoint, Snowflake.

### 🔄 Destination Update Methods (EXAM CRITICAL)
When setting a Fabric Data Warehouse as a destination, choose between two update methods:

1. **Append:**
   * Adds new incoming rows to an existing table.
   * *Use Case:* Transactional accumulation (e.g., daily sales logs, streaming events).
   * *Note:* **Append** is the **only** update method supported for KQL Databases and Azure Data Explorer destinations!
2. **Replace:**
   * Truncates and replaces the entire table contents on every run.
   * *Use Case:* Full reference/dimension reloads (e.g., daily product catalog refreshes).

### 🚀 Publishing Behavior
* Edits auto-save as drafts during development.
* Data movement and destination loading **only execute after selecting Publish**.

---

## 6. Module Summary

* **Load Strategies:**
  * **Full Load:** Truncates and reloads entire table (initial setup or small reference tables).
  * **Incremental Load:** Processes only changed/new rows (ongoing updates).
  * **Load Order:** Dimensions MUST be loaded *before* Fact tables.
* **Key Management & SCD:**
  * **Surrogate Keys:** DW-generated integer keys (`ProductKey`) that decouple DW relationships from source changes.
  * **SCD Type 1:** Overwrites existing row in-place (no history).
  * **SCD Type 2:** Adds a new row with `StartDate`/`EndDate`/`IsActive` flags (full historical tracking).
* **Ingestion Tooling Options:**
  1. **Data Pipeline Copy Job:** Wizard-driven, high-throughput bulk copying (supports Full vs. Incremental copy modes).
  2. **T-SQL (`COPY INTO` & `CTAS` / `INSERT...SELECT`):** High-performance programmatic loading. OneLake files require **no credentials**; Azure Storage requires SAS Token/Key. `CTAS` creates a new table; `INSERT...SELECT` appends.
  3. **Dataflow Gen2:** Visual Power Query transformations with Copilot M code generation. Supports **Append** and **Replace** update methods (*Append* is the only method for KQL databases).
