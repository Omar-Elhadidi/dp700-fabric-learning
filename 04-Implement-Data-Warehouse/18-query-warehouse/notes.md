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

*(Waiting for next unit)*
