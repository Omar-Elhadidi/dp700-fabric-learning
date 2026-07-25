# M16: Get Started with Data Warehouses in Microsoft Fabric

Microsoft Fabric provides a fully managed, enterprise relational data warehouse with full transactional T-SQL capabilities (including `INSERT`, `UPDATE`, `DELETE`, and `MERGE`). Data is natively stored in **Delta Lake format on OneLake**, providing seamless integration across all Fabric workloads.

---

## 1. Introduction

* **Relational Warehousing in Fabric:** Provides a structured, SQL-based environment for enterprise BI and analytics at scale.
* **Full T-SQL Support:** Supports complete DDL and DML data operations (`CREATE TABLE`, `INSERT`, `UPDATE`, `DELETE`, `MERGE`).
* **Open Storage:** Backed by **Delta Lake format on OneLake**, allowing Lakehouse engines (Spark, KQL) and BI reporting tools to query the data without data movement or copy overhead.
* **AI & Copilot Integration:** Native Copilot assistance for T-SQL query authoring, model building, and AI-driven insights.

---

## 2. Understand Data Warehouses (Dimensional Modeling)

Data warehouses structure data using **dimensional modeling** (Fact and Dimension tables) to optimize read performance and simplify analytical queries.

### Fact vs. Dimension Tables
* **Fact Tables:** Store quantitative/numerical measurements of business events (e.g., `SalesAmount`, `OrderQuantity`). Characterized by high row volume and foreign keys connecting to dimensions.
* **Dimension Tables:** Store qualitative/descriptive attributes giving context to facts (e.g., Customer Name, Product Category, Store Region). Characterized by lower row volume and denormalized fields.

### Dimension Keys: Surrogate vs. Alternate Keys
* **Surrogate Key:** An integer key generated **inside the data warehouse** when inserting rows. Maintains internal DW consistency and handles changing business logic independent of source systems.
* **Alternate (Natural / Business) Key:** The original key from the transactional source system (e.g., `CustomerCode`, `ProductSKU`). Preserves data lineage and traceability back to source DBs.

### Special Types of Dimensions
* **Time / Date Dimension:** Explicit table containing calendar/temporal attributes (Year, Quarter, Month, Weekday, IsHoliday) allowing analysts to aggregate facts across time periods.
* **Slowly Changing Dimensions (SCD):** Techniques used to track historical changes to dimension attributes over time (e.g., keeping track of a customer's old and new address for historical reporting accuracy).

### Schema Design: Star Schema vs. Snowflake Schema
* **Star Schema (Preferred for DW):**
  * Fact table sits at the center, directly connected to surrounding **denormalized** dimension tables.
  * *Benefits:* Reduces table joins, speeds up query execution, simplifies BI semantic modeling.
* **Snowflake Schema:**
  * Dimensions are partially **normalized** into sub-dimensions (e.g., `DimProduct` links to `DimCategory`, which links to `DimSupplier`).
  * *Trade-off:* Reduces data redundancy/storage space, but increases query complexity due to additional `JOIN` operations.

---

## 3. Understand Data Warehouses in Fabric

A **Fabric Data Warehouse** is a fully managed enterprise relational database storing data natively in open **Delta format on OneLake**.

### ⚔️ Warehouse vs. SQL Analytics Endpoint (EXAM CRITICAL)
| Feature / Capability | Warehouse | SQL Analytics Endpoint |
| :--- | :--- | :--- |
| **Read Data** | Yes | Yes |
| **Write Data (INSERT/UPDATE/DELETE/MERGE)** | **Yes** (Full DML) | **No** (Read-Only) |
| **Create Tables (DDL - CREATE TABLE)** | **Yes** | **No** |
| **Create Views & Stored Procedures** | Yes | Yes |
| **Primary Data Source** | Native Warehouse Delta tables | Lakehouse Delta tables |

### Key Fabric DW Capabilities
* **Full T-SQL & ACID Compliance:** Supports complete DDL/DML, including `MERGE` for upsert/SCD patterns.
* **Separation of Compute & Storage:** Compute scales automatically and independently from storage.
* **Cross-Database Querying:** Query tables across different Warehouses and Lakehouses using **3-part naming** (`database.schema.table`) in a single query without data duplication.
* **Tooling & TDS Support:** Connects natively via SQL Server Management Studio (SSMS), Azure Data Studio, and TDS endpoints.

### Data Ingestion Methods
1. **`COPY INTO` (T-SQL):** High-performance bulk loading from external files (CSV, Parquet) in Azure Storage directly into warehouse tables.
2. **`OPENROWSET`:** Query external storage files ad-hoc without pre-defining table structures.
3. **Data Pipelines & Dataflows Gen2:** Visual ingestion and orchestration.
4. **Cross-Database T-SQL:** Query Lakehouse Delta tables directly using 3-part syntax without copying data.

### Loading Pattern: Staging Tables
A best practice in Fabric DW is to land raw source data in staging tables (e.g., `StgSales`) via `COPY INTO` or pipelines, then transform and load into Star Schema facts/dimensions:

```sql
-- Load Star Schema Fact table from Staging table with key lookups
INSERT INTO dbo.FactSales (SalesKey, CustomerKey, ProductKey, DateKey, SalesAmount, Quantity)
SELECT
    s.OrderID,
    c.CustomerKey,
    p.ProductKey,
    d.DateKey,
    s.Amount,
    s.Qty
FROM dbo.StgSales AS s
INNER JOIN dbo.DimCustomer AS c ON s.CustomerID = c.CustomerAltKey
INNER JOIN dbo.DimProduct AS p ON s.ProductID = p.ProductAltKey
INNER JOIN dbo.DimDate AS d ON s.OrderDate = d.DateValue;
GO
```

### Zero-Copy Table Clones
* **Syntax:** `CREATE TABLE dbo.Employee AS CLONE OF dbo.EmployeeUSA;`
* **Mechanism:** Copies only the table metadata while referencing the identical underlying Delta files on OneLake.
* **Benefits:** Zero additional storage cost, near-instant creation.
* **Use Cases:** Dev/test environments, instant data recovery prior to batch updates, point-in-time snapshot reporting.

---

## 4. Query and Transform Data

Fabric Data Warehouse provides both code-based and no-code tools for exploring and transforming data.

### Query Editors
1. **SQL Query Editor:** Code-first T-SQL experience with IntelliSense, syntax highlighting, client-side validation, and Copilot integration (generating, explaining, or fixing SQL).
2. **Visual Query Editor:** No-code diagrammatic experience (similar to Power Query Online). Drag-and-drop tables onto a visual canvas to filter, merge, aggregate, and transform data without writing SQL code.

### Reusable Transformation Objects
* **Views (`CREATE VIEW`):**
  * Virtual tables representing a saved SELECT query.
  * Standardizes business metrics (e.g., aggregating sales by region).
  * Hides complex `JOIN` logic from end-user analysts.
  * Enhances AI/Copilot accuracy by providing clean, well-named semantic layers for natural language queries.
* **Stored Procedures (`CREATE PROCEDURE`):**
  * Procedural T-SQL scripts executed on demand.
  * Used for repeatable ETL processes, data loading from staging to dimensional tables, and applying complex business rules in batch.

---

## 5. Model Data in a Warehouse

Data modeling embeds business logic, relationships, and metadata directly inside the Fabric Data Warehouse, serving T-SQL consumers, Power BI reports, and Copilot AI agents.

### 🧹 Preparing Data for Consumption & AI Readiness
* **Hide Internal Artifacts:** Hide staging tables, ETL flags, and internal surrogate key columns from consumer field lists.
* **Business-Friendly Renaming:** Rename technical abbreviations to human-readable names (e.g., `CustRgn` -> `Customer Region`).
* **Metadata & Descriptions:** Add descriptions to tables/columns.
  * *AI Impact:* Copilot for Power BI and Fabric IQ data agents rely heavily on clear names and descriptions to generate accurate natural language SQL and DAX queries.

### 🔗 Table Relationships in Star Schema
* **Cardinality:** Standard star schema relationships are **Many-to-One** (`* : 1`) connecting Fact tables to Dimension tables via shared surrogate keys (`CustomerKey`).
* **Cross-Filter Direction:** **Single direction** (Dimension filters Fact table) is the standard for query predictability and optimal performance.
* *Impact:* Encodes join logic once so users and AI data agents don't have to rewrite explicit `JOIN` logic.

### 📏 Standardizing Access: Views vs. DAX Measures
* **Views (T-SQL Standardization):** Encapsulates joins and filtering rules into reusable objects for SQL analysts and report data sources.
* **DAX Measures (BI Calculation Standardization):** Created directly in the warehouse **Model View** (e.g., `Total Sales = SUM(FactSales[SalesAmount])`). Centralizes metric calculations so business logic changes update in one place across all reports.

### ⚡ Direct Lake Mode Semantic Models
* Semantic models created from a Fabric Warehouse operate in **Direct Lake mode**.
* **Zero Import / Zero Refresh:** Direct Lake queries OneLake Delta Parquet files directly without importing data into Power BI memory or requiring scheduled refresh schedules.

---

## 6. Secure and Monitor a Warehouse

Fabric Data Warehouses enforce multi-layered security and provide rich real-time and historical monitoring diagnostics.

### 🛡️ Multi-Layer Security Architecture

#### 1. Workspace & Item Permissions
* **Workspace Roles:** Admin, Member, Contributor, Viewer.
* **Item Permissions (Warehouse-Level Granularity):**
  * `Read`: Grants permission to connect via the TDS / SQL endpoint. *(Connection fails without this!)*
  * `ReadData`: Grants permission to read data from all tables/views in the warehouse.
  * `ReadAll`: Grants permission to read raw Delta/Parquet files directly in OneLake.

#### 2. Granular T-SQL Security (Enforced Everywhere, including AI/Copilot)
* **Object-Level Security (OLS):** Controls access to specific tables, views, or stored procedures.
* **Row-Level Security (RLS):** Restricts row access based on user context using `WHERE` predicate functions.
* **Column-Level Security (CLS):** Restricts visibility of sensitive columns.
* **Dynamic Data Masking (DDM):** Obfuscates sensitive fields (e.g., masking emails or account numbers) for non-privileged users.

---

### 📊 Monitoring Diagnostics

#### 1. Query Insights (`queryinsights` Schema - 30-Day History)
System views retaining **30 days** of historical execution performance:
* `queryinsights.exec_requests_history`: Details on completed SQL requests.
* `queryinsights.long_running_queries`: Queries ranked by execution duration.
* `queryinsights.exec_sessions_history`: Details on completed user sessions.

#### 2. Dynamic Management Views (DMVs - Real-Time Activity)
Monitors live active connections and executing queries:
```sql
-- Identify currently running long queries
SELECT request_id, session_id, start_time, total_elapsed_time
FROM sys.dm_exec_requests
WHERE status = 'running'
ORDER BY total_elapsed_time DESC;
```

#### 3. Session Control (`KILL` Command)
* **Admin Requirement:** Only workspace **Admins** have permission to execute the `KILL <session_id>` statement to cancel long-running queries. Members, Contributors, and Viewers can only view their own session outputs.

---

## 7. Module Summary

* **Fabric Data Warehouse:** Fully managed relational DB storing open Delta format on OneLake with full T-SQL DDL & DML (`INSERT`, `UPDATE`, `DELETE`, `MERGE`).
* **Dimensional Modeling:** Fact tables (numerical measures) connected to denormalized Dimension tables (descriptive attributes) via Surrogate keys (DW-internal) and Alternate keys (source lineage).
* **Warehouse vs. SQL Analytics Endpoint:** Warehouse supports Full Read/Write & DDL; SQL Analytics Endpoint provides Read-Only access to Lakehouse Delta tables.
* **Ingestion & Staging:** High-performance bulk loading via `COPY INTO`, cross-database 3-part naming (`database.schema.table`), and staging table load patterns.
* **Performance & Cloning:** Metadata-only zero-copy table clones (`CREATE TABLE AS CLONE OF`) for instant zero-storage dev/test copies.
* **Semantic Models & Direct Lake:** Warehouse models operate natively in **Direct Lake mode** without data import or refresh schedules.
* **Security & Monitoring:** Granular T-SQL security (OLS, RLS, CLS, Dynamic Data Masking) enforced across all engines & AI agents; 30-day historical execution diagnostics via `queryinsights` schema; real-time DMVs (`sys.dm_exec_requests`); `KILL` session command restricted to workspace **Admins**.
