# 📚 Module 4: Work with Real-Time Data in an Eventhouse

---

## 1. Module Overview & Objectives

*   **Goal:** Learn how to ingest, query, and analyze real-time streaming data using Eventhouses and KQL in Microsoft Fabric.
*   **Key Objectives:**
    *   Describe the role of Eventhouses and KQL databases in Fabric Real-Time Intelligence.
    *   Ingest real-time data into an Eventhouse.
    *   Query real-time data using KQL (Kusto Query Language).
    *   Understand how Eventstreams connect sources to destinations in Fabric.

---

## 2. Real-Time Data & KQL Database Foundations (HIGH YIELD)

### 🏠 What is an Eventhouse?
*   **Definition:** A Fabric container that stores and manages one or more **KQL (Kusto Query Language) Databases**.
*   **Optimization:** Specifically designed to store, manage, and analyze high-volume, high-velocity real-time streaming data.
*   **Ingestion Paths:** Data can be streamed in continuously via **Fabric Eventstreams** or ingested directly.
*   **Downstream Capabilities:**
    *   **Querying:** Run fast KQL or T-SQL queries inside KQL querysets.
    *   **Visualizing:** Build interactive **Real-Time Dashboards**.
    *   **Automating:** Route data to **Fabric Activator** to trigger alerts and automated workflows.

### ⚙️ How KQL Databases Handle Real-Time Data
*   **Automatic Time Partitioning (Exam Alert):** Data is automatically partitioned by **ingestion time** (when it arrives) as it enters the database.
    *   *Benefit:* Restricts query scans to specific time slices (e.g., "last 5 minutes") instead of doing full-table scans across years of data, maximizing query speed.
*   **Immutability:** Real-time event logs represent snapshots of history at a specific point in time (e.g., sensor telemetry at 12:00:00). These records cannot be updated or modified after creation.
*   **Append-Only Pattern:** KQL databases are optimized for append-only data entry. Updates and deletes are extremely rare, which contrasts with traditional relational databases that optimize for transaction integrity and heavy DML updates.
*   **Time-Series Focus:** The timestamp is treated as a primary indexing and filtering dimension.

---

## 3. Working with Eventhouses & KQL Databases (HIGH YIELD)

### 🏗️ Eventhouse Components & Creation
*   **Default Setup:** Creating a new Eventhouse automatically provisions a default KQL database with the same name.
*   **Database Objects:** Within a KQL Database, you can create and manage:
    *   **Tables:** Storage locations for rows/columns.
    *   **Stored Procedures & Functions:** Saved logic blocks.
    *   **Materialized Views:** Pre-computed query tables.
    *   **Data Streams:** Active real-time ingestion paths.
    *   **Shortcuts:** References to external databases.

### 📥 Data Ingestion Pathways
Data can be loaded into KQL databases from a wide array of sources:
1.  **Files & Cloud Stores:** Local files, Azure Blob/ADLS Gen2, Amazon S3.
2.  **Streaming Services:** Fabric Eventstreams, Azure Event Hubs, Real-Time Hub.
3.  **Fabric ETL Pipeline Tools:** Copy Activity pipelines, Dataflows Gen2.
4.  **Connectors:** Apache Kafka, Confluent Cloud, Apache Flink, MQTT, Amazon Kinesis, Google Cloud Pub/Sub.

### 🔗 Database Shortcuts
*   **Definition:** Reference links to existing KQL databases in other Eventhouses or Azure Data Explorer (ADX) clusters.
*   *Key Benefit (Exam Alert):* Allows users to query external databases in-place **without copying or duplicating data** across workspaces.

### 🌐 OneLake Availability
*   **Scope:** Enabled at the KQL database level or individual table level.
*   *Purpose:* Automatically exposes KQL database/table data to the Fabric OneLake ecosystem.
*   *Value:* Other workloads (Power BI Direct Lake, Lakehouses, Warehouses) can query and join KQL tables seamlessly **without copying data**.

---

## 4. KQL (Kusto Query Language) Query Basics

KQL querysets serve as the playground for queries and support both **KQL** and **T-SQL** queries.

### 🚨 Crucial Rule: Case Sensitivity (HIGH YIELD)
*   **Everything in KQL is case-sensitive.**
*   This applies to:
    1.  Table names (e.g., `TaxiTrips` vs. `taxitrips` are treated as different tables).
    2.  Column names.
    3.  Function and operator names.
    4.  String values within query filters.

### 🔀 The Pipeline Model (Funnel Concept)
KQL query execution uses a pipeline model where data flows left-to-right or top-to-bottom using the pipe (`|`) character.
*   Each operator processes only the output returned by the previous step.
*   **Example Query:**
    ```kql
    TaxiTrips
    | where fare_amount > 20
    | project trip_id, pickup_datetime, fare_amount
    | take 10
    ```

### 🛠️ Key KQL Operators for the Exam

*   **`where`**: Filters rows matching a logical condition (comparable to SQL `WHERE`).
*   **`project`**: Selects specific columns to return or renames columns (comparable to SQL `SELECT`).
*   **`take` (or `limit`)**: Returns a specified number of rows (comparable to SQL `LIMIT` or `TOP`). Useful for exploring large tables cheaply.
*   **`summarize`**: Aggregates data (comparable to SQL `GROUP BY` with aggregations like `count()`, `sum()`, `avg()`).
    *   *Example:*
        ```kql
        TaxiTrips
        | summarize trip_count = count() by taxi_id
        ```

---

## 5. KQL Query Optimization Best Practices (CRITICAL EXAM VALUE)

KQL queries are resource-billed and performance-bound by the **volume of data scanned**. Applying these exact syntax rules keeps queries fast.

### ⏳ Rule 1: Filter Early and Reorder Logical Checks
*   **Time-Based Filtering First:** Place time filters (like `ago(30min)`, `ago(1d)`) at the absolute top of the pipeline. Since Eventhouses partition by ingestion time, this restricts KQL search blocks immediately.
*   **Filter Funneling (Order of Filters):** Order `where` clauses by their filtering power. Put filters that eliminate the *most* rows first.
    ```kql
    // CORRECT Funnel Ordering
    TaxiTrips
    | where pickup_datetime > ago(1d)    // Time filter first - eliminates most data
    | where vendor_id == "VTS"           // Specific value - eliminates some data  
    | where fare_amount > 0              // Detail check - eliminates least data
    ```

### 📋 Rule 2: Project Columns Early
*   For wide tables, select only necessary columns as early as possible using `project`. This lowers the memory footprint for all downstream aggregation, sorting, or joins.
    ```kql
    TaxiTrips
    | project trip_id, pickup_datetime, fare_amount  // Select columns early
    | where pickup_datetime > ago(1d)                // Then filter
    ```

### 🔗 Rule 3: The Left-Side Join Rule (MUST KNOW)
*   **The Rule:** When joining two tables in KQL, **always place the smaller table on the left** (before the `join` operator), and the larger table on the right.
*   **Why:** KQL processes the left table first to build an in-memory lookup index. Running a small lookup against a streaming large dataset is much more resource-efficient than loading a large dataset into memory.
    ```kql
    // ✅ CORRECT: Small lookup table on the left
    VendorInfo        
    | join kind=inner TaxiTrips on vendor_id

    // ❌ INCORRECT: Large transaction table on the left (Avoid)
    TaxiTrips         
    | join kind=inner VendorInfo on vendor_id
    ```

### 📊 Rule 4: Aggregation Capping
*   When performing intensive grouping and aggregation (`summarize`), always use `limit` or `take` at the end to restrict the final result volume.

---

## 6. Materialized Views & Stored Functions (HIGH YIELD)

### 📈 Materialized Views

*   **Definition:** Precomputed aggregation tables in a KQL database designed to accelerate reports/dashboards querying massive datasets (millions/billions of rows).
*   **The Dual-Part Architecture (CRITICAL EXAM CONCEPT):**
    A materialized view is split into two physical parts to provide speed and accuracy on streaming data:
    1.  **Materialized Part:** Historical precomputed aggregate data that has already been processed in background cycles.
    2.  **Delta Part:** Newly ingested data that arrived *after* the last background process.
*   **Query-Time Merge:** When a user queries a materialized view, KQL automatically merges the Materialized Part and Delta Part *on-the-fly*. This guarantees that queries always return **fresh, 100% current data**, while still executing at the high speed of precomputed results.
*   **Background Maintenance:** A background process periodically sweeps new entries from the Delta Part, aggregates them, and writes them into the Materialized Part.
*   **Creation Syntax:**
    ```kql
    .create materialized-view TripsByVendor on table TaxiTrips
    {
        TaxiTrips
        | summarize trips = count(), avg_fare = avg(fare_amount), total_revenue = sum(fare_amount)
        by vendor_id, pickup_date = format_datetime(pickup_datetime, "yyyy-MM-dd")
    }
    ```
*   **Querying:** Query it like a regular table using the view name directly:
    ```kql
    TripsByVendor | where pickup_date >= ago(7d)
    ```

### ⚙️ Stored Functions

*   **Definition:** Reusable query definitions (comparable to SQL User Defined Functions or Views) that standardize logic, parameters, and filters.
*   **Creation Syntax (with parameters):**
    Use the `.create-or-alter function` command. Parameters must specify names and types (e.g. `num_passengers:long`).
    ```kql
    .create-or-alter function trips_by_min_passenger_count(num_passengers:long)
    {
        TaxiTrips
        | where passenger_count >= num_passengers 
        | project trip_id, pickup_datetime
    }
    ```
*   **Execution:** Call it like a table, passing arguments in parentheses:
    ```kql
    trips_by_min_passenger_count(3)
    | take 10
    ```

---

## 7. Module Summary

*   **Eventhouses** house KQL databases designed for append-only, immutable time-series events.
*   **Time partitioning** automatically index events by ingestion time, making range filters like `ago(1d)` highly efficient.
*   **Key Optimization Rules:**
    1. Filter early with time checks.
    2. Project columns early to reduce memory foot-print.
    3. Join with the **smaller table on the left** of the `join` operator.
*   **Materialized Views** precompute aggregates, combining historical blocks and delta streaming logs on-the-fly at query time to ensure both speed and freshness.
*   **Stored Functions** save reusable query templates that accept parameters (e.g. `.create-or-alter function`).
