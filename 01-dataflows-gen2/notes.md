# 📥 Module 1: Ingest Data with Dataflows Gen2

This module covers the fundamentals of using Dataflows Gen2 inside Microsoft Fabric for low-code data ingestion and transformation.

---

## 1. What is Ingestion?
Simply put, **ingest** means to take data in or bring data into a system.

### Analogies
*   **Eating:** You eat food ──> your body ingests food. Microsoft Fabric ingests data ──> it brings data into Fabric.
*   **Moving Books:** 
    *   *Garage* = External data source (outside Fabric)
    *   *House* = Microsoft Fabric (inside OneLake)
    *   *Carrying the books inside* = Data ingestion
    *   Once the books are inside your house, you can organize them (transform), put them on shelves (store), or read them (analyze).

---

## 2. Power Query vs. Dataflow Gen2
They are closely related, but distinct:
*   **Power Query** = The visual tool/engine you use to clean and transform data.
*   **Dataflow Gen2** = The Microsoft Fabric service that *uses* Power Query to connect to sources, transform the data, and load it into a destination.

### Analogy: Cooking & Restaurants
*   **Power Query** is the **kitchen** where you prepare and chop the food (clean/transform).
*   **Dataflow Gen2** is the **entire restaurant process**:
    1.  Receive ingredients (Ingest data from source)
    2.  Prepare them in the kitchen (Transform via Power Query Editor)
    3.  Serve the meal (Load the clean data to a Fabric destination like a Lakehouse)

```
Data Source ──> Dataflow Gen2 [ Power Query Editor (Clean/Transform) ] ──> Lakehouse/Warehouse
```

### Comparison Matrix

| Feature / Aspect | Power Query | Dataflow Gen2 |
|---|---|---|
| **Role** | Cleans and transforms data | Connects to sources, runs Power Query, and loads to destination |
| **Type** | Transformation engine | Complete ETL/ELT solution/service |
| **Where it's used** | Excel, Power BI Desktop, Fabric | Microsoft Fabric exclusively |
| **Primary Focus** | Shaping data | Ingesting, transforming, and loading data |

---

## 3. Real-World Context: Global Retail Scenario
*   **The Challenge:** A global retail company has sales and inventory data scattered across various sources in different stores. The business requests a **semantic model** that consolidates all these disparate sources.
*   **The Problem with Manual Work:** Without an automated dataflow, you would have to manually extract and transform data from every source daily, which is slow and error-prone.
*   **The Solution (Dataflow Gen2):** It allows you to prepare and clean the data to ensure consistency, and stage it in your preferred destination (Lakehouse/Warehouse).
*   **Core Architectural Principle (HIGH YIELD):** Always perform transformations **upstream** (closer to the data source) rather than downstream (inside Power BI Desktop/reports). This improves report performance, ensures consistent data logic for all analysts, and enables reusable ETL code.

---

## 4. Dataflows Gen2 Architecture: ETL vs. ELT

You can configure your Fabric pipelines in two ways depending on the order of operations:

### Pattern A: ETL (Extract, Transform, Load)
*   **Workflow:** Dataflow Gen2 connects directly to the source system ──> transforms and cleans the data using Power Query ──> loads the cleaned data into the Lakehouse/Warehouse.
*   **Best for:** Low-code cleaning directly during ingestion.

### Pattern B: ELT (Extract, Load, Transform)
*   **Workflow:** 
    1.  A **Data Pipeline** extracts raw data from the source and loads it directly into the Lakehouse (Bronze layer).
    2.  A **Dataflow Gen2** connects to the raw data inside the Lakehouse, cleans/transforms it, and saves it back to the Lakehouse (Silver/Gold layers).
*   **Best for:** Staging raw data first, then serving the cleaned Dataflow output as a curated semantic model for Power BI report developers.

---

## 5. Orchestration, Scaling, & Reuse

*   **Trigger Options:** You can run/refresh Dataflows Gen2:
    1.  **Manually** on-demand.
    2.  On a **Refresh Schedule** (built-in).
    3.  As part of a **Data Pipeline orchestration** (using the *Dataflow* activity node).
*   **Optional Destination & Discoverability:** Adding a data destination to your Dataflow is **optional**. If you omit it, you can still make the dataflow **discoverable** so that report developers can connect to it directly via the **OneLake Data Hub** in Power BI Desktop.
*   **Horizontal Partitioning:** You can create one global "master" dataflow, and then let analysts build specialized, downstream dataflows off of it for specific business needs. This promotes reusable ETL logic and prevents hitting your source database with duplicate queries.

---

## 6. Benefits & Limitations (HIGH YIELD FOR EXAM)

### Benefits
*   **Consistent Dimensions:** Easy to inject standard reference tables (like a standard Date Dimension).
*   **Self-Service Subsets:** Allows self-service business users to access a clean subset of data separately without giving them direct access to the main Data Warehouse.
*   **Source Protection:** Extract data from slow/busy source databases *once* into a dataflow, then let multiple reports query the dataflow. This dramatically reduces source database load.
*   **Simplicity:** Hides complex raw database structures from analysts. They only see the cleaned, simplified tables.
*   **Low-Code:** Allows non-programmers to do ingestion/ETL using visual drag-and-drop.

### Limitations (Must Know)
*   ❌ **No Row-Level Security (RLS):** Dataflows Gen2 do **NOT** support row-level security. Security must be managed at the database/semantic model layer instead.
*   ❌ **Not a Warehouse Replacement:** They are processing pipelines, not a final database storage solution.
*   ❌ **Capacity Workspace Required:** You must run Dataflows Gen2 inside a workspace that is assigned to a **Fabric Capacity** (or Trial capacity), not a shared personal workspace.

---

## 7. Power Query Online Interface

### Visual UI Components
*   **Queries Pane (Left Side):** Lists your data sources/tables (queries).
    *   *Disable Load:* If a query is used only as an intermediate step (e.g., to merge with another query) but you do not want it written to your database, uncheck "Enable Load" to optimize performance.
    *   *Duplicate vs. Reference (HIGH YIELD):* 
        *   **Duplicate:** Creates a complete copy of the query with all its M transformation steps. Changes to the original do not affect the duplicate.
        *   **Reference:** Creates a link to the original query. The new query starts with the output of the original query. If the original query changes, the referenced query automatically updates (useful for building dimension tables from a single source).
*   **Diagram View (Top):** Visual flowchart showing data sources and applied steps. You can turn this off to save screen space.
*   **Data Preview Pane (Center):** Shows a small subset of rows. You can drag columns or right-click to apply transformations directly on the preview.
*   **Query Settings Pane (Right Side):**
    *   Contains **Applied Steps** (chronological list of transformations). You can delete or edit steps here using the gear icon.
    *   Contains **Data Destination** setup (where to load the clean data).

### Under the Hood: M Language & Query Folding
*   **M Language:** All visual drag-and-drop transformations in Power Query are automatically written in the **M formula language** behind the scenes. You can view/edit the raw M code by opening the **Advanced Editor**.
*   **Query Folding:** When connecting to database sources, Power Query tries to convert your M transformation steps into a single SQL query and run it on the source database. This is called **Query Folding** and it prevents pulling billions of raw rows into Fabric for processing.

### Valid Data Destinations (HIGH YIELD)
You can configure a Dataflow Gen2 to write to destinations both inside and outside Fabric.

#### Inside Microsoft Fabric:
1.  **Lakehouse**
2.  **Data Warehouse**
3.  **SQL Database** (Fabric SQL DB)

#### Outside Microsoft Fabric (Azure Services):
1.  **Azure SQL Database**
2.  **Azure Data Explorer** (KQL / Kusto)
3.  **Azure Synapse Analytics**
