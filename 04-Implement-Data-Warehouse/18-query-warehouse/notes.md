# M18: Query a Microsoft Fabric Data Warehouse

Learn how to query a Fabric Data Warehouse using the built-in SQL query editor, the no-code Visual query editor, and external client tools (SSMS / Azure Data Studio) using Microsoft Entra ID authentication.

## Learning Objectives
In this module, you learn how to:
- Use the **SQL query editor** to write and run T-SQL queries against a data warehouse.
- Build queries using the no-code **Visual query editor**.
- Connect to a data warehouse from **SQL Server Management Studio (SSMS)** using Microsoft Entra ID authentication.

---

## 1. Introduction

* **Core Purpose:** Querying data warehouses to extract business intelligence insights from structured and semi-structured data.
* **Star Schema Query Context:**
  * **Fact Table (Center):** Measurable, quantitative data (e.g., revenue, quantities sold).
  * **Dimension Tables (Points of Star):** Descriptive context answering *Who* (Customer), *What* (Product), *Where* (Store), *When* (Date), and *How much* (Currency).

---

## 2. Query Data (Analytical T-SQL Patterns)

Analytical querying in Fabric Data Warehouse relies on Star/Snowflake schema joins, aggregations, window ranking functions, and high-speed approximate counts.

### 📊 Aggregating Facts across Dimensions
Slicing measures by joining facts to multiple dimension tables:
```sql
SELECT  dates.CalendarYear,
        dates.CalendarQuarter,
        custs.City,
        SUM(sales.SalesAmount) AS TotalSales
FROM dbo.FactSales AS sales
JOIN dbo.DimDate AS dates ON sales.OrderDateKey = dates.DateKey
JOIN dbo.DimCustomer AS custs ON sales.CustomerKey = custs.CustomerKey
GROUP BY dates.CalendarYear, dates.CalendarQuarter, custs.City
ORDER BY dates.CalendarYear, dates.CalendarQuarter, custs.City;
```

### ❄️ Snowflake Schema Joins
Traversing multi-level normalized dimensions requires chaining joins:
```sql
SELECT  cat.ProductCategory,
        SUM(sales.OrderQuantity) AS ItemsSold
FROM dbo.FactSales AS sales
JOIN dbo.DimProduct AS prod ON sales.ProductKey = prod.ProductKey
JOIN dbo.DimCategory AS cat ON prod.CategoryKey = cat.CategoryKey
GROUP BY cat.ProductCategory;
```

### 🏆 Ranking Window Functions (HIGH YIELD FOR EXAM)
Evaluates ordinal positions across partitions using `OVER (PARTITION BY ... ORDER BY ...)`:

| Ranking Function | Behavior | Tie Example (Prices: 8.99, 8.49, 5.99, 5.99, 2.99) |
| :--- | :--- | :--- |
| **`ROW_NUMBER()`** | Sequential integer per row. No ties. | 1, 2, 3, 4, 5 |
| **`RANK()`** | Ranks with ties; **skips subsequent ranks (gaps)**. | 1, 2, **3, 3, 5** (Skips 4) |
| **`DENSE_RANK()`** | Ranks with ties **without gaps**. | 1, 2, **3, 3, 4** (No gap) |
| **`NTILE(n)`** | Distributes rows into `n` equal buckets/percentiles. | `NTILE(4)` assigns Quartiles 1, 1, 2, 2, 3 |

```sql
SELECT  ProductCategory, ProductName, ListPrice,
        ROW_NUMBER() OVER (PARTITION BY ProductCategory ORDER BY ListPrice DESC) AS RowNum,
        RANK()       OVER (PARTITION BY ProductCategory ORDER BY ListPrice DESC) AS RankVal,
        DENSE_RANK() OVER (PARTITION BY ProductCategory ORDER BY ListPrice DESC) AS DenseRankVal,
        NTILE(4)     OVER (PARTITION BY ProductCategory ORDER BY ListPrice DESC) AS Quartile
FROM dbo.DimProduct;
```

### ⚡ Fast Data Exploration: `APPROX_COUNT_DISTINCT`
* **Problem:** Exact `COUNT(DISTINCT column)` scans full massive tables and can be slow during exploratory analysis.
* **Solution:** `APPROX_COUNT_DISTINCT(column)` uses the **HyperLogLog algorithm** for near-instant estimates.
* **Accuracy Guarantee:** Maximum error rate **< 2%** with **97% probability**.

```sql
SELECT dates.CalendarYear,
       APPROX_COUNT_DISTINCT(sales.OrderNumber) AS ApproxOrders
FROM FactSales AS sales
JOIN DimDate AS dates ON sales.OrderDateKey = dates.DateKey
GROUP BY dates.CalendarYear;
```

---

## 3. Use the SQL Query Editor

The built-in SQL query editor provides a full T-SQL scripting experience directly in the Fabric portal—no external tools required.

### 🚀 Launching & Auto-Save
* Open your warehouse from the workspace → editor connects automatically (no manual connection strings).
* New queries are created via **Home menu → New SQL query** or **Queries node → `...` → New SQL Query**.
* Queries are **auto-saved** to the `My queries` folder in the Explorer as you work.

### 📊 Working with Results
Results preview is capped at **10,000 rows**. The results toolbar provides:

| Action | What It Does |
| :--- | :--- |
| **Open in Excel** | Opens as a **live connection** (not static download). Supports refresh inside Excel. |
| **Explore this data (Preview)** | Side-by-side matrix + visual for quick ad-hoc trend spotting. |
| **Visualize results** | Creates a Power BI visual inline. ⚠️ Does **NOT** support queries with `ORDER BY`. |
| **Copy** | Copy results with headers, results only, or headers only. |
| **Save as View** | Persists the highlighted `SELECT` as a warehouse View object. |
| **Save as Table** | Creates a new physical table from the query results (similar to CTAS). |

### 🤖 Copilot in the SQL Query Editor
* **Explain query:** Adds natural language comments describing each part of your SQL.
* **Fix query errors:** Auto-detects and fixes syntax/logic errors (button grays out when no errors exist).
* **Chat pane:** Generate T-SQL from natural language questions.
* **Capacity Requirement:** Copilot requires **F2 or P1 capacity or higher** — NOT available on trial SKUs.

---

## 4. Explore the Visual Query Editor

A no-code, drag-and-drop interface for building queries visually — ideal for analysts who prefer graphical design over writing T-SQL.

### How It Works
1. **Open:** `New SQL query` dropdown → **New visual query**.
2. **Build:** Drag tables from Explorer onto the canvas.
3. **Join:** Right-click a table → **Merge queries** → Select common key column + join type → OK.
4. **Run:** Select **Run** to see results in the pane below.

### Key Features
* **Auto-Generated T-SQL:** As you design visually, equivalent T-SQL is generated behind the scenes.
* **View SQL / Edit SQL Script:** Inspect the generated T-SQL or open it in the SQL query editor for manual refinement.
* **Result Actions:** Same as SQL editor — **Save as Table**, **Save as View**, **Download Excel file**, **Visualize results**.

---

## 5. Use Client Tools to Query a Warehouse

External SQL client tools (SSMS, Azure Data Studio, 3rd-party) can connect to Fabric warehouses via standard TDS endpoints.

### 🔌 Connecting from SSMS
1. In the Fabric workspace, select `...` next to warehouse → **Copy SQL connection string**.
2. In SSMS **Connect to Server** dialog:
   * **Server name:** Paste the copied connection string.
   * **Database Name:** Enter the **exact warehouse name** (e.g., `sample-dw`). ⚠️ **Must not be left blank** — connection may fail even if auth succeeds.
   * **Authentication:** Select a **Microsoft Entra ID** method.

### 🔐 Authentication Rules (EXAM CRITICAL)
* Fabric Data Warehouse supports **Microsoft Entra ID authentication ONLY**.
* **SQL Authentication (username/password) is NOT supported.**
* Any 3rd-party tool can connect using **ODBC or OLE DB drivers** with Entra ID auth.

### 🌐 Network Requirement
* **TCP port 1433** must be open in your network firewall for connectivity.

---

## 6. Module Summary

* **Analytical T-SQL Patterns:**
  * Star/Snowflake schema joins, multi-dimension aggregations.
  * Ranking Window Functions (`OVER (PARTITION BY ... ORDER BY ...)`): `ROW_NUMBER()` (sequential, no ties), `RANK()` (ties with gaps), `DENSE_RANK()` (ties without gaps), `NTILE(n)` (quartiles/percentiles).
  * `APPROX_COUNT_DISTINCT`: HyperLogLog algorithm providing <2% error at 97% probability for high-speed exploratory analysis on massive tables.
* **SQL Query Editor:** Built-in web editor, results capped at 10k rows preview, Open in Excel (live connection), Save as View/Table. Copilot requires F2/P1 capacity minimum.
* **Visual Query Editor:** No-code drag-and-drop experience generating T-SQL automatically.
* **Client Tools (SSMS/ADS):** Connect via TDS endpoint on TCP port 1433 using **Microsoft Entra ID authentication ONLY** (SQL Auth is NOT supported). Must specify exact database name in connection dialog.
