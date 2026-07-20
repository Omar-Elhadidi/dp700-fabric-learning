# 📚 Module 8: Work with Delta Lake tables in Microsoft Fabric

---

## 1. Module Overview & Objectives

*   **Goal:** Master the deep mechanics of Delta Lake tables in Microsoft Fabric, including ACID transaction logs, table versioning (time travel), partitioning configurations, and optimization features (V-Order).
*   **Key Objectives:**
    *   Understand the format and transaction logging of Delta Lake tables.
    *   Create and configure Delta Lake tables using Spark and SQL.
    *   Query historical data versions (Time Travel).
    *   Optimize tables using partitioning, compaction (OPTIMIZE), and V-Order writes.

---

## 2. Introduction to Delta Lake in Fabric

*   **Delta Lake Foundation:** The relational tables in a Microsoft Fabric lakehouse are built on the open-source **Delta Lake** table format (supported by the Linux Foundation).
*   **The Big Picture:** Delta Lake acts as a transactional storage abstraction layer over data lakes, enabling SQL DML operations (`UPDATE`, `DELETE`, `MERGE`) inside Apache Spark.
*   **Best of Both Worlds:** It provides traditional RDBMS guarantees (ACID transactions, structured schemas, validation) while retaining the scalability, open format, and cost-effectiveness of cloud file storage (OneLake/ADLS Gen2).
*   **The Metastore Architecture:** Fabric handles metastore registration automatically, but mastering advanced operations (time travel, partitioning, index optimization) is required for high-performance enterprise data engineering on the platform.

---

## 3. Understanding Delta Lake (Storage & Mechanics)

Delta Lake maps logical relational tables over raw physical data files in OneLake. In Fabric, Delta tables are identified in the UI by a **triangular Delta ($\Delta$) icon**.

### 📁 Physical Directory Layout
For every Delta table, the underlying storage contains:
*   **Parquet Data Files:** Columnar storage files containing the actual table data rows, compressed and optimized for fast reads.
*   **`_delta_log` Folder:** A metadata directory containing transaction log files in **JSON format** (e.g., `000000.json`, `000001.json`). These logs track every table mutation (adds/deletes of Parquet files).

### 🌟 Core Benefits of Delta Tables (HIGH YIELD)

1.  **Relational CRUD Semantics:** Apache Spark can run standard relational `INSERT`, `UPDATE`, `DELETE`, and `MERGE` operations on Delta tables.
2.  **Full ACID Guarantees:** 
    *   *Atomicity:* Operations succeed entirely or write nothing.
    *   *Consistency:* Data obeys defined constraints/schemas.
    *   *Isolation:* Delta Lake enforces **serializable isolation** for concurrent operations, preventing read/write interference.
    *   *Durability:* Once committed to the transaction log, mutations are permanent.
3.  **Time Travel & Versioning:** Every transaction is versioned. You can query table states at a specific point in time or version number using the transaction log historical record.
4.  **Batch & Streaming Unified:** Native support for Spark **Structured Streaming**. Delta tables can act as:
    *   *Sources:* Streaming inputs that feed new data into downstream pipelines.
    *   *Sinks:* Streaming destinations where incoming raw data is continually written.
5.  **Interoperability:** Data is stored as open Parquet files. Fabric automatically mounts these files as queryable tables via the read-only **SQL analytics endpoint**.

---

## 4. Creating Delta Tables: Managed vs. External (HIGH YIELD)

Data engineers can define Delta tables in the Lakehouse metastore using PySpark DataFrame API, DeltaTableBuilder, or Spark SQL.

### 🛡️ Managed vs. External Tables (Syllabus Core)

| Table Type | Storage Location | Drop Table Behavior (`DROP TABLE`) | Example Code (PySpark / SQL) |
|---|---|---|---|
| **Managed Table** | Under the `Tables` node in OneLake. Managed by Spark runtime. | **Deletes both** the table schema metadata and the physical Parquet data files. | `df.write.format("delta").saveAsTable("mytable")`<br>or<br>`CREATE TABLE sales USING DELTA` |
| **External Table** | Under the `Files` folder, or external path (`abfss://...`). | **Deletes ONLY** the metadata definition in the metastore. The physical Parquet files are preserved. | `df.write.format("delta").saveAsTable("ext_table", path="Files/path")`<br>or<br>`CREATE TABLE ext USING DELTA LOCATION 'Files/path'` |

### 🛠️ Creating Table Metadata (Without Datasets)

If you need to define an empty table schema in the metastore to populate later:

#### Option A: DeltaTableBuilder API (PySpark)
```python
from delta.tables import *

DeltaTable.create(spark) \
  .tableName("products") \
  .addColumn("Productid", "INT") \
  .addColumn("ProductName", "STRING") \
  .addColumn("Price", "FLOAT") \
  .execute()
```

#### Option B: Spark SQL
```sql
%%sql
CREATE TABLE salesorders
(
    Orderid INT NOT NULL,
    OrderDate TIMESTAMP NOT NULL,
    SalesTotal FLOAT
)
USING DELTA
```

### 💾 Saving Delta Files directly
To write DataFrames as delta-parquet files directly without registering them in the metastore:
```python
delta_path = "Files/mydatatable"

# Write new files
df.write.format("delta").save(delta_path)

# Overwrite existing folder
df.write.format("delta").mode("overwrite").save(delta_path)

# Append rows to existing folder
df.write.format("delta").mode("append").save(delta_path)
```
> [!TIP]
> **Fabric Table Auto-Discovery:** If you save delta files directly to the **`Tables/`** location in the lakehouse (e.g. `df.write.format("delta").save("Tables/my_new_data")`), Fabric will **automatically discover** the directory and create the metastore metadata schema for you without writing any SQL definition code.

---

## 5. Optimizing Delta Tables (CRITICAL EXAM CONCEPTS)

Because Parquet files are immutable, every `UPDATE` or `DELETE` creates new files. This leads to the **Small File Problem**, which slows down queries. Fabric provides multiple features to mitigate this.

### ⚡ 1. OptimizeWrite
*   **What it does:** Consolidates data *during the write phase* to write fewer, larger files rather than many tiny files.
*   **Fabric Default:** Enabled by default.
*   **Session Toggle (PySpark):**
    ```python
    spark.conf.set("spark.microsoft.delta.optimizeWrite.enabled", True)
    ```

### 🧹 2. Compaction (OPTIMIZE)
*   **What it does:** Consolidates existing small files within a table into larger ones post-load.
*   **How to Run:** Via the UI (Table > `...` > Maintenance > Run OPTIMIZE) or SQL command.

### 🚀 3. Microsoft V-Order (HIGH YIELD)
*   **What it is:** A Microsoft-proprietary sorting, row group distribution, encoding, and compression technique applied to the Parquet format. It is **100% compliant** with open-source Parquet (external engines can still read V-Ordered files).
*   **Verti-Scan Technology:** Power BI (Direct Lake mode) and the Fabric SQL engine use Verti-Scan, which natively exploits V-Order to achieve in-memory-like speed.
*   **Overhead:** Incurs a ~15% write performance penalty.
*   **Best Practice Decision:**
    *   *Enable (Default):* On final reporting tables queried by Power BI/SQL.
    *   *Disable:* For write-intensive staging tables where data is only written and read once/twice.

### 🗑️ 4. VACUUM (Housekeeping)
*   **What it does:** Permanently deletes historical Parquet data files that are no longer referenced by the current transaction log state.
*   **Default Retention Period:** **7 days (168 hours)**. Fabric enforces a safety baseline and blocks you from specifying a shorter retention window via standard commands.
*   *Time Travel Warning:* Running VACUUM permanently deletes files, preventing you from using "Time Travel" to query versions older than the retention threshold.
*   **SQL Commands:**
    ```sql
    -- Delete files older than 7 days
    VACUUM lakehouse.products RETAIN 168 HOURS;
    
    -- Check historical operations log
    DESCRIBE HISTORY lakehouse.products;
    ```

### 📂 5. Partitioning Delta Tables
*   **Concept:** Physically splits data on disk into subfolders based on column values (e.g. `Year=2024/`).
*   **Benefit:** Enables **data skipping** (read engines ignore irrelevant partition folders based on metadata).
*   *When to Partition:* Very large datasets with low-cardinality columns (few distinct values, like Year or Region).
*   *When NOT to Partition:* Small datasets (worsens small file problem), high-cardinality columns (creates too many folders), or multi-level partition nesting.
*   **Syntax:**
    *   *PySpark:* `df.write.format("delta").partitionBy("Category").saveAsTable("partitioned_products")`
    *   *Spark SQL:*
        ```sql
        CREATE TABLE partitioned_products (
            ProductID INT,
            ProductName STRING,
            Category STRING
        ) USING DELTA PARTITIONED BY (Category);
        ```

---

## 6. Modifying & Querying Delta Tables in Spark

You can interact with, update, and inspect the transaction log history of Delta Tables using either Spark SQL, the native programmatic Delta API, or Time Travel parameters.

### ✍️ 1. Running DML Operations (SQL)
Spark SQL allows standard relational modifications on both managed tables and external file locations:
*   *Using PySpark Library:*
    ```python
    spark.sql("INSERT INTO products VALUES (1, 'Widget', 'Accessories', 2.99)")
    ```
*   *Using Notebook Magic Cells:*
    ```sql
    %%sql
    UPDATE products
    SET ListPrice = 2.49 
    WHERE ProductId = 1;
    ```

### 🐍 2. Using the Programmatic Delta Lake API
When working directly with storage paths instead of catalog tables, load a `DeltaTable` reference to run code-first updates:
```python
from delta.tables import *
from pyspark.sql.functions import *

# Load DeltaTable by location path
delta_path = "Files/mytable"
deltaTable = DeltaTable.forPath(spark, delta_path)

# Execute updates programmatically
deltaTable.update(
    condition = "Category == 'Accessories'",
    set = { "Price": "Price * 0.9" }
)
```

### ⏳ 3. Time Travel & History Operations (HIGH YIELD)
Every table operation logs details to the `_delta_log` metadata folder.

#### A. Review Transaction History
To audit past runs, check versions, and identify who/what modified data:
*   *For Managed Tables:*
    ```sql
    %%sql
    DESCRIBE HISTORY products;
    ```
*   *For External Paths:*
    ```sql
    %%sql
    DESCRIBE HISTORY 'Files/mytable';
    ```

#### B. Querying Historical States (Time Travel)
You can load past versions of your data into a DataFrame by referencing either a specific **version index** or a **timestamp string**:
*   *Option 1: Version-based lookup (`versionAsOf`):*
    ```python
    df_v0 = spark.read.format("delta").option("versionAsOf", 0).load(delta_path)
    ```
*   *Option 2: Timestamp-based lookup (`timestampAsOf`):*
    ```python
    df_date = spark.read.format("delta").option("timestampAsOf", '2026-07-14').load(delta_path)
    ```

---

## 7. Delta Lake & Spark Structured Streaming (HIGH YIELD)

Delta tables can be utilized as both **sources** (streaming inputs) and **sinks** (streaming outputs) in near-real-time data workflows using **Spark Structured Streaming**.

### 📥 1. Delta Table as a Streaming Source
*   **What it does:** Reads newly appended rows from a Delta table in real time into a boundless DataFrame.
*   **Syntax:**
    ```python
    stream_df = spark.readStream.format("delta") \
        .option("ignoreChanges", "true") \
        .table("orders_in")
    ```
*   *Validation:* Call `stream_df.isStreaming` to verify (should return `True`).
*   > [!IMPORTANT]
    > **Handling Data Mutations:** By default, Spark streaming expects only **append** operations. If data in the source table is updated or deleted, the stream will fail with an error. To prevent this, you **must** pass `.option("ignoreChanges", "true")` or `.option("ignoreDeletes", "true")`.

### 📤 2. Delta Table as a Streaming Sink
*   **What it does:** Continuously writes transformed data stream records into a target Delta table.
*   **Syntax:**
    ```python
    output_table_path = 'Tables/orders_processed'
    checkpointpath = 'Files/delta/checkpoint'
    
    deltastream = transformed_df.writeStream.format("delta") \
        .option("checkpointLocation", checkpointpath) \
        .start(output_table_path)
    ```
*   > [!IMPORTANT]
    > **Checkpointing (`checkpointLocation`):** You **must** specify a checkpoint folder in OneLake. This logs metadata tracking the progress of processed batches, enabling the streaming job to recover from cluster failure without duplicating or losing data.

### 🛑 3. Stopping the Stream
Streaming queries run continuously on active compute resources. You must stop them when finished to avoid runaway capacity consumption costs:
```python
deltastream.stop()
```

---

## 8. Summary: Working with Delta Lake Tables

*   **ACID on Files:** Delta Lake adds full transactional consistency, version control (Time Travel), and schema validation directly over open Parquet data files.
*   **File Layout:** Composed of physical data Parquet files and a metadata JSON transaction log folder named `_delta_log`.
*   **Performance Maintenance:**
    *   *OptimizeWrite:* Combines small file sizes during writes.
    *   *OPTIMIZE:* Compacts historical small files.
    *   *V-Order:* Proprietary columnar compression that Verti-Scan exploits for in-memory-like Power BI Direct Lake reads.
    *   *VACUUM:* Purges orphan files older than the default 7-day threshold, clearing space but ending old time travel queries.
    *   *Partitioning:* Physically partitions folder structures for data skipping (useful for low-cardinality, large volumes).
*   **Real-time Streaming:** Integrates natively as streaming sources (`ignoreChanges` option for updates) and sinks (`checkpointLocation` for state recovery).
