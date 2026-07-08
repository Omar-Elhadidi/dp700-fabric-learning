# 📚 Module 3: Use Apache Spark in Microsoft Fabric

This module covers the core concepts, configurations, and programmatic patterns of Apache Spark inside Microsoft Fabric. Spark is the primary engine used for big data processing, data engineering (cleansing and shaping), and data science workloads inside a Fabric Lakehouse.

---

## 1. Spark Configuration in Fabric Workspaces (HIGH YIELD)

Fabric manages Spark compute through **Spark Pools**. There are two main types of pools you must understand for the exam:

### 🚀 Starter Pools vs. 🛠️ Custom Pools

| Feature | Starter Pools (Default) | Custom Pools |
|---|---|---|
| **Provisioning Speed** | **Fast (< 5 seconds)**. Nodes are pre-warmed. | **Slow (2 - 5 minutes)**. Clusters spin up from scratch. |
| **Node Sizes** | Small, Medium, Large (preconfigured). | Customizable (specify exact CPU, Memory, Node Types). |
| **Autoscale** | Automatically scales up/down within defaults. | Configurable min/max node limits. |
| **Use Case** | Quick ad-hoc analysis, development, fast-running pipeline activities. | Production workloads, predictable large-scale ETL, memory-intensive jobs. |
| **Billing** | Charges based on active usage duration. | Charges based on configuration run limits. |

### ⚙️ Workspace-Level Spark Settings
As a Fabric administrator or developer, you can configure Spark settings at the **Workspace Settings** level:
1.  **Spark Compute:** Assign default starter or custom pools to the workspace.
2.  **Automatic Tuning (Autotuning):** Automatically adjusts Spark configurations (like memory allocations and executor counts) based on query history and execution metrics.
3.  **Environment Configurations:** Install custom Python packages (via PyPI/Conda libraries) and set custom Spark properties (like `spark.sql.shuffle.partitions`) that apply to all notebooks in the workspace.

---

## 2. Notebooks vs. Spark Job Definitions (SJD)

There are two primary methods to execute Spark code in Fabric:

### 📓 1. Spark Notebooks
*   **Interface:** Interactive, multi-language (PySpark, Spark SQL, Scala, Spark R) notebook interface.
*   **Scenarios:** 
    *   Ad-hoc data exploration, visualization, and debugging.
    *   Prototyping data transformations.
    *   Iterative model development.
*   **Orchestration:** Can be run interactively or scheduled within a Data Pipeline via the *Notebook Activity*.

### 📦 2. Spark Job Definitions (SJD)
*   **Interface:** Non-interactive submission of compiled scripts (Python `.py` files, Java/Scala `.jar` packages).
*   **Scenarios:**
    *   Production ETL scripts that don't need visualization.
    *   Batch jobs scheduled to run at regular intervals.
    *   Migrating existing Spark code from Databricks or Azure Synapse.
*   **Performance:** Slightly faster startup time compared to notebooks because it bypasses interactive UI shell setup overhead.

---

## 3. Spark Dataframe Operations

The Dataframe API allows you to manipulate structured data programmatically using Python (PySpark).

### 📥 Reading Data
```python
# Read from a CSV file in Lakehouse Files
df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("Files/raw_data/sales.csv")

# Read from a Delta Table in Lakehouse Tables
df_tables = spark.read.table("dbo.sales_table")
```

### 🔄 Transforming Data
```python
from pyspark.sql.functions import col, year, month, split

# Adding new columns, casting types, splitting strings, and filtering rows
transformed_df = df.withColumn("Year", year(col("OrderDate"))) \
                    .withColumn("Month", month(col("OrderDate"))) \
                    .withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)) \
                    .filter(col("Quantity") > 5)
```

### 💾 Writing Data (Saving to Delta Tables)
```python
# Write as a Managed Delta Table (automatically registers in Lakehouse catalog)
transformed_df.write.format("delta") \
    .mode("overwrite") \
    .saveAsTable("dbo.processed_sales")
```

---

## 4. Spark SQL (Tables vs. Views)

Spark SQL allows developers to query dataframes and tables using standard ANSI SQL.

### 📊 Managed Tables vs. External Tables
*   **Managed Tables:** Registered in the Lakehouse catalog. Both the data files (stored in `Tables/` directory) and metadata are managed by Fabric. Deleting the table **deletes both** metadata and the underlying Delta files.
*   **External Tables:** Point to a folder location outside of the default `Tables/` path. Deleting the table **only deletes the metadata**; your data files remain intact in storage.

### 👁️ Temporary Views vs. Global Temporary Views
*   **Temporary View (Session-Scoped):**
    *   Created using `df.createOrReplaceTempView("my_temp_view")`.
    *   Only accessible within the active notebook session/Spark Context.
    *   Automatically dropped when the notebook is closed or the Spark session idle-timeout kicks in.
*   **Global Temporary View (Application-Scoped):**
    *   Created using `df.createOrReplaceGlobalTempView("my_global_view")`.
    *   Shared across different notebook sessions running on the *same Spark application/cluster*.
    *   Must be queried with the prefix path `global_temp.my_global_view`.

---

## 5. Visualizing Data in Spark Notebooks
Fabric Spark Notebooks contain built-in charting engines. You can easily visualize dataframes without writing visualization code by using:
1.  **The Chart View:** Converting a Spark Dataframe display output directly into a bar, line, scatter, or pie chart by clicking the **Chart** tab below a cell output.
2.  **Native Code Visualization:** Using popular python charting libraries:
    ```python
    import matplotlib.pyplot as plt
    import seaborn as sns

    # Convert Spark DataFrame to Pandas for local visualization (small datasets only)
    pandas_df = transformed_df.select("Year", "Quantity").toPandas()
    
    sns.lineplot(data=pandas_df, x="Year", y="Quantity")
    plt.show()
    ```
