# 📚 DP-700 Custom Study Notes

This is your living study guide. We will build this out surgically as you study, ensuring you have a highly compressed, zero-fluff review sheet before August 11.

---

## 📥 Section 1: Data Ingestion & Dataflows Gen2

### 1. What is Ingestion?
Simply put, **ingest** means to take data in or bring data into a system.

#### Analogy 1: Eating
- You eat food → your body ingests food.
- Microsoft Fabric ingests data → it brings data into Fabric.

#### Analogy 2: Moving Books
- **Garage** = External data source (outside Fabric)
- **House** = Microsoft Fabric (inside OneLake)
- **Carrying the books inside** = Data ingestion
- Once the books are inside your house, you can organize them (transform), put them on shelves (store), or read them (analyze).

#### Everyday Example
Imagine you have a raw Excel file on your laptop. 
- The Excel file is outside Fabric.
- When Dataflow Gen2 reads that file and brings its contents into Fabric, it is **ingesting** the data.

---

### 2. Power Query vs. Dataflow Gen2
They are closely related, but distinct:
- **Power Query** = The visual tool/engine you use to clean and transform data.
- **Dataflow Gen2** = The Microsoft Fabric service that *uses* Power Query to connect to sources, transform the data, and load it into a destination.

#### Analogy: Cooking & Restaurants
- **Power Query** is the **kitchen** where you prepare and chop the food (clean/transform).
- **Dataflow Gen2** is the **entire restaurant process**:
  1. Receive ingredients (Ingest data from source)
  2. Prepare them in the kitchen (Transform via Power Query Editor)
  3. Serve the meal (Load the clean data to a Fabric destination like a Lakehouse)

#### Visual Workflow
```
Data Source ──> Dataflow Gen2 [ Power Query Editor (Clean/Transform) ] ──> Lakehouse/Warehouse
```

#### Comparison Table

| Feature / Aspect | Power Query | Dataflow Gen2 |
|---|---|---|
| **Role** | Cleans and transforms data | Connects to sources, runs Power Query, and loads to destination |
| **Type** | Transformation engine | Complete ETL/ELT solution/service |
| **Where it's used** | Excel, Power BI Desktop, Fabric | Microsoft Fabric exclusively |
| **Primary Focus** | Shaping data | Ingesting, transforming, and loading data |

---

### 3. Real-World Context: Global Retail Scenario
- **The Challenge:** A global retail company has sales and inventory data scattered across various sources in different stores. The business requests a **semantic model** that consolidates all these disparate sources.
- **The Problem with Manual Work:** Without an automated dataflow, you would have to manually extract and transform data from every source daily, which is slow and error-prone.
- **The Solution (Dataflow Gen2):** It allows you to prepare and clean the data to ensure consistency, and stage it in your preferred destination (Lakehouse/Warehouse).

---

### 4. Dataflows Gen2 Architecture: ETL vs. ELT

You can configure your Fabric pipelines in two ways depending on the order of operations:

#### Pattern A: ETL (Extract, Transform, Load)
- **Workflow:** Dataflow Gen2 connects directly to the source system ──> transforms and cleans the data using Power Query ──> loads the cleaned data into the Lakehouse/Warehouse.
- **Best for:** Low-code cleaning directly during ingestion.

#### Pattern B: ELT (Extract, Load, Transform)
- **Workflow:** 
  1. A **Data Pipeline** extracts raw data from the source and loads it directly into the Lakehouse (Bronze layer).
  2. A **Dataflow Gen2** connects to the raw data inside the Lakehouse, cleans/transforms it, and saves it back to the Lakehouse (Silver/Gold layers).
- **Best for:** Staging raw data first, then serving the cleaned Dataflow output as a curated semantic model for Power BI report developers.

---

### 5. Orchestration, Scaling, & Reuse

- **Trigger Options:** You can run/refresh Dataflows Gen2:
  1. **Manually** on-demand.
  2. On a **Refresh Schedule** (built-in).
  3. As part of a **Data Pipeline orchestration** (using the *Dataflow* activity node).
- **Optional Destination & Discoverability:** Adding a data destination to your Dataflow is **optional**. If you omit it, you can still make the dataflow **discoverable** so that report developers can connect to it directly via the **OneLake Data Hub** in Power BI Desktop.
- **Horizontal Partitioning:** You can create one global "master" dataflow, and then let analysts build specialized, downstream dataflows off of it for specific business needs. This promotes reusable ETL logic and prevents hitting your source database with duplicate queries.

---

### 6. Benefits & Limitations (HIGH YIELD FOR EXAM)

#### Benefits
- **Consistent Dimensions:** Easy to inject standard reference tables (like a standard Date Dimension).
- **Self-Service Subsets:** Allows self-service business users to access a clean subset of data separately without giving them direct access to the main Data Warehouse.
- **Source Protection:** Extract data from slow/busy source databases *once* into a dataflow, then let multiple reports query the dataflow. This dramatically reduces source database load.
- **Simplicity:** Hides complex raw database structures from analysts. They only see the cleaned, simplified tables.
- **Low-Code:** Allows non-programmers to do ingestion/ETL using visual drag-and-drop.

#### Limitations (Must Know)
- ❌ **No Row-Level Security (RLS):** Dataflows Gen2 do **NOT** support row-level security. Security must be managed at the database/semantic model layer instead.
- ❌ **Not a Warehouse Replacement:** They are processing pipelines, not a final database storage solution.

---

## 🖥️ Section 2: Dataflows Gen2 Interface & Engine

### 1. Visual Interface Components (Power Query Online)

- **Queries Pane (Left Side):** Lists your data sources/tables (queries).
  - *Key Feature:* **Disable Load**. If you use a query only as an intermediate step (e.g., to merge with another query) but do not want it written to your final database, right-click and uncheck "Enable Load". This optimizes performance.
- **Diagram View (Top):** Visual flowchart showing data sources and applied steps. You can turn this off to save screen space.
- **Data Preview Pane (Center):** Shows a small subset of rows. You can drag columns or right-click to apply transformations directly on the preview.
- **Query Settings Pane (Right Side):**
  - Contains **Applied Steps** (chronological list of transformations). You can delete or edit steps here using the gear icon.
  - Contains **Data Destination** setup (where to load the clean data).

---

### 2. Under the Hood: M Language & Query Folding
- **M Language:** All visual drag-and-drop transformations in Power Query are automatically written in the **M formula language** behind the scenes. You can view/edit the raw M code by opening the **Advanced Editor**.
- **Query Folding:** When connecting to database sources, Power Query tries to convert your M transformation steps into a single SQL query and run it on the source database. This is called **Query Folding** and it prevents pulling billions of raw rows into Fabric for processing.

---

### 3. Valid Data Destinations (HIGH YIELD)
You can configure a Dataflow Gen2 to write to destinations both inside and outside Fabric.

#### Inside Microsoft Fabric:
1. **Lakehouse**
2. **Data Warehouse**
3. **SQL Database** (Fabric SQL DB)

#### Outside Microsoft Fabric (Azure Services):
1. **Azure SQL Database**
2. **Azure Data Explorer** (KQL / Kusto)
3. **Azure Synapse Analytics**

---

## 🔄 Section 3: Data Pipelines & Orchestration

### 1. What is a Data Pipeline?
A visual tool in Microsoft Fabric used to **orchestrate** and automate data workflows. While Dataflows clean data, Pipelines control the *scheduling, order, and dependencies* of multiple different activities.

### 2. When to use a Pipeline vs. a Dataflow (Key Exam Distinction)

| Use **Dataflow Gen2** for: | Use **Data Pipeline** for: |
|---|---|
| Visual, low-code data transformation (Power Query) | High-speed bulk copying of data (Copy activity) |
| Merging, pivoting, or cleaning tables | Running Spark Notebooks |
| Basic data preparation for business analysts | Running SQL Scripts / Stored Procedures in a database |
| | Scheduling and triggering sequential dependencies |

### 3. Common Pipeline Activities
You must know what components can be placed inside a pipeline:
- **Copy Data:** Fast bulk movement of data from source to destination without transformation.
- **Dataflow:** Runs a specific Dataflow Gen2.
- **Notebook:** Runs a PySpark or Scala Spark notebook in a Fabric workspace.
- **Get Metadata:** Inspects files or databases to check if a file exists, its size, or its schema before running a job.
- **Script:** Runs a SQL script or stored procedure inside a Data Warehouse.

### 4. Orchestration Flow & Automation
- **Conditional Paths (Dependencies):** Activities inside a pipeline are visually linked by success/failure constraints. E.g., *Run Notebook A* ──(On Success)──> *Run Script B*. If Notebook A fails, you can trigger a *Send Email* notification instead.
- **Scheduling:** Pipelines can be scheduled to run automatically based on time intervals, daily routines, or event-based triggers (e.g., when a new file lands in your storage).
