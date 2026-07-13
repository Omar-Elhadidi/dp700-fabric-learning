# 📚 Module 6: Get started with lakehouses in Microsoft Fabric

---

## 1. Module Overview & Objectives

*   **Goal:** Learn how to create and manage a Lakehouse in Microsoft Fabric, including how to ingest data (files) and query them using SQL and Spark.
*   **Key Objectives:**
    *   Describe the core features of a Fabric Lakehouse.
    *   Create a Lakehouse workspace and database schema.
    *   Ingest data files into Lakehouse storage.
    *   Query Lakehouse tables using SQL Analytics Endpoint.

---

## 2. Introduction to Fabric Lakehouses

*   **The Enterprise Problem:** Traditional data warehouses handle structured transactional data well, but struggle to store/process semi-structured and unstructured data (IoT logs, API feeds, etc.), resulting in disconnected systems, data silos, and complex ETL pipelines.
*   **The Lakehouse Solution:** A unified SaaS data platform that combines the **scalable, flexible storage of a data lake** (storing any file format) with the **ACID compliance and relational query capabilities of a data warehouse**.
*   **Compute & Storage:**
    *   *Storage Layer:* Built entirely on top of **OneLake**.
    *   *Compute Engines:* Uses **Apache Spark** (for big data processing and transformations) and **SQL compute engines** (for querying and reporting) on the *same* physical files.
*   **Downstream Benefits:** Organizing and structuring files within the Lakehouse ensures they are clean, governed, and immediately ready to ground Fabric Copilots, reports, and AI analytics.

---

## 3. Lakehouse Features & Capabilities (Syllabus Core)

A Fabric Lakehouse separates physical file storage from logical table management, allowing both raw data processing and transactional queries in one item.

### 📁 Lakehouse Storage Areas: Tables vs. Files

| Feature | `Tables` Folder | `Files` Folder |
|---|---|---|
| **Data Format** | Delta Lake (Parquet + Transaction Log). | Native formats (CSV, JSON, Parquet, Images, Docs). |
| **SQL Queries** | **Yes** (via SQL Analytics Endpoint). | **No** (Direct SQL queries not supported). |
| **Schema & ACID** | Enforced schemas; ACID transactions supported. | No schema enforcement; no ACID. |
| **Power BI Access** | Yes (Direct Lake, Import, or DirectQuery modes). | No (Must be loaded into tables first). |
| **Use Case** | Cleansed, ready-to-report business data. | Landing zone, raw staging, unstructured data. |

### 🛠️ Delta Lake Architecture
Delta Lake is the foundation of Fabric tables. It stores data as compressed **Parquet data files** alongside a **`_delta_log` transaction log folder** that tracks changes.
*   **Key Advantages for the Exam:**
    1.  **ACID Transactions:** Ensures concurrent reads/writes are consistent (no partial writes).
    2.  **Schema Enforcement:** Rejects data inserts that don't match the defined table schema to prevent corruption.
    3.  **Time Travel:** Query historical states of data or roll back changes using the transaction log versioning.
    4.  **Upserts/Deletes:** Supports efficient SQL operations (`UPDATE`, `DELETE`, `MERGE INTO`) on OneLake.

### 🔒 Access Control and Security
Fabric applies permissions in layers to protect lakehouse data:
*   **Collaborator Security:** Managed through workspace roles (Admin, Member, Contributor, Viewer).
*   **Sharing Security:** Item-level sharing allows read-only access to specific tables/SQL endpoints for reports.
*   **Granular Security (HIGH YIELD):** The **SQL Analytics Endpoint** supports:
    *   *Row-Level Security (RLS):* Restricts which rows users can see based on query logic.
    *   *Column-Level Security (CLS):* Hides sensitive columns from specific users.
    *   *Schema-Level Security:* Controls permissions across table groups organized under schemas (e.g., `dbo` vs. `finance`).

### 🧠 Grounding AI (Fabric IQ & Copilot)
*   **The Concept:** Data engineering is the prerequisite for AI. Fabric IQ data agents convert natural language questions into T-SQL queries.
*   **The Rule:** If your tables have messy schemas, inconsistent casing, or obscure column names (e.g., `col_01`), the data agent's SQL conversion will fail or hallucinate. Clean, descriptive naming conventions directly determine Copilot and Agent performance.

---

## 4. Ingesting & Transforming Data inside a Lakehouse

### 🗄️ Lakehouse Schema Support (HIGH YIELD)
*   **Default Schema:** When a Lakehouse is created, schemas are enabled by default, and a default schema named **`dbo`** is created automatically.
*   **Namespace Querying:** You can query schema-enabled tables across workspaces using a **four-part namespace**:
    $$\text{workspace}.\text{lakehouse}.\text{schema}.\text{table}$$
*   **Security benefits:** Allows applying security controls at the schema level rather than managing item permissions on individual tables.

### 🔄 Lakehouse Explorer vs. SQL Analytics Endpoint
You can interact with your Lakehouse using two different UI views:

1.  **Lakehouse Explorer (Read/Write):**
    *   *Purpose:* Data management and preparation.
    *   *Features:* Add/modify tables, upload files, write PySpark notebooks, and configure data loading.
    *   *Cross-Lakehouse Reference:* You can pin **reference lakehouses** to browse tables from other lakehouses side-by-side.
2.  **SQL Analytics Endpoint (Read-Only):**
    *   *Purpose:* Relational data querying and access control.
    *   *Features:* Query Delta tables using T-SQL. You can create SQL views, user-defined functions (UDFs), and apply row/column-level security.
    *   *Limitation:* **Read-only**. You *cannot* run `INSERT`, `UPDATE`, or `DELETE` statements (DML) through the SQL endpoint.

### 📥 Ingestion & Transformation Techniques

*   **Load to Table (No-Code UI):** Right-click a CSV or Parquet file in the `Files` folder to automatically convert it into a Delta table. Supports append and overwrite modes.
*   **Dataflows Gen2 (Low-Code):** Best for Power Query/Excel users to perform visual ETL.
*   **Spark Notebooks (Code-First):** Programmatic ingestion and complex transformations using PySpark/Scala/SQL.
*   **Data Factory Pipelines (Orchestration):** Use *Copy data* activities to ingest data from external hybrid or multi-cloud systems.

### 🔗 OneLake Schema Shortcuts (HIGH YIELD)
*   **Schema Shortcuts:** Instead of linking single tables, you can create a shortcut that maps an **entire schema folder** containing multiple Delta tables in another Lakehouse or ADLS Gen2 folder.
*   **Result:** All tables inside that folder instantly appear as local tables inside your Lakehouse schema namespace without copying any files.
*   **Access Control:** Authorizes queries using your Entra ID credentials. You must have read permissions in the target location to access data.

---

## 5. Querying & Analyzing Lakehouse Data (SQL, Spark & Power BI)

### 🛢️ 1. SQL Analytics Endpoint (T-SQL)
*   **Access Pattern:** Read-only access to delta-parquet files. 
*   **Use Cases:**
    *   Ad-hoc analysis using traditional T-SQL queries.
    *   Data verification (auditing data post-transformation).
    *   Connections to third-party tools (Excel, Azure Data Studio, SSMS).
*   **Relational Logic:** Can create SQL views and functions to abstract complex joins.
*   **Security:** Supports column-level security (CLS) and row-level security (RLS).

### 📓 2. Spark Notebooks (PySpark vs. Spark SQL)
*   **Access Pattern:** Programmatic read/write cluster environment.
*   **Use Cases:** Exploratory Data Analysis (EDA), advanced cleansing, and machine learning.
*   *Cross-Workspace Queries:* Can query across workspaces using the four-part namespace:
    $$\text{workspace}.\text{lakehouse}.\text{schema}.\text{table}$$
*   **Coding APIs:**
    *   *Spark SQL:* ANSI SQL syntax in `%%sql` cells.
    *   *PySpark:* Python DataFrame API (`df.select()`, `df.filter()`) or running SQL statements programmatically via `spark.sql("SELECT ...")`.

### 📊 3. Power BI Direct Lake Mode (CRITICAL EXAM CONCEPT)
*   **What it is:** The default connectivity engine used by Power BI when querying Fabric Lakehouse semantic models.
*   **How it works:** Reads data directly from the **Delta Lake Parquet files in OneLake** without importing or copying them.
*   **Benefits (Direct Lake vs. Import vs. DirectQuery):**
    *   *No Latency / Real-time:* Reports reflect Lakehouse changes immediately without needing scheduled dataset refreshes (unlike *Import Mode*).
    *   *High Performance:* Avoids slow relational query conversion latency (unlike *DirectQuery Mode*) because it reads highly compressed, columnar Parquet files directly in memory.
    *   *Zero Storage Overhead:* No need to duplicate Lakehouse tables in Power BI cloud memory.
*   **Semantic Model AI Grounding:** Creating clean semantic models with defined relationships and metrics enables **Copilot in Power BI** to generate visual charts and answer business questions in natural language.

---

## 6. Summary: Get Started with Lakehouses

*   **Unified Storage:** Fabric Lakehouses eliminate data silos by combining flexible file storage (Files folder) and structured transactional analytics (Tables folder) in a single platform on OneLake.
*   **Integrated Compute:** Apache Spark and SQL compute engines interact with the same physical delta-parquet files seamlessly.
*   **Downstream AI & BI Enabler:** Clean schemas, descriptive column names, and defined semantic models directly enable **Fabric IQ data agents** and **Copilots** to reason over your data and query it accurately in natural language.
