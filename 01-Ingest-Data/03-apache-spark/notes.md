# 📚 Module 3: Use Apache Spark in Microsoft Fabric

---

## 1. Introduction to Apache Spark in Microsoft Fabric

*   **What is Apache Spark?** An open-source, parallel processing framework designed for large-scale data processing and big data analytics.
*   **Where is it used?** Spark runs across multiple platforms, including Azure HDInsight, Azure Synapse Analytics, and Microsoft Fabric.
*   **Fabric Integration Benefit:** Because Spark is built natively into Fabric, it shares the same workspace environment as other services (like Lakehouses and Data Pipelines), making it much easier to integrate Spark-based data processing into your overall data engineering workflows.

---

## 2. Prepare to Use Apache Spark (Architecture & Configuration)

Spark uses a distributed "divide and conquer" architecture to execute tasks across multiple computers in a cluster.

### 🖥️ Cluster Node Architecture
A Spark cluster (known as a **Spark pool** in Fabric) coordinates work across two types of compute nodes:
*   **Head Node (Driver Node):** Coordinates the distributed execution of processes through a driver program. It acts as the brain.
*   **Worker Nodes (Executors):** The muscle. These nodes run executor processes that perform the actual data processing tasks (reading, transforming, writing).
*   **Data Access:** The cluster accesses data residing in OneLake-based storage (such as a Fabric Lakehouse).

### 🗣️ Supported Programming Languages
*   **Languages:** PySpark (Python-specific variant), Spark SQL, Scala (Java-based scripting), Spark R, and Java.
*   **Best Practice:** In daily production, most data engineering workloads are accomplished using a combination of **PySpark** and **Spark SQL**.

### ⚡ Spark Pools in Microsoft Fabric
*   **Starter Pools (Default):** Pre-warmed nodes created by Fabric at the workspace level. They start almost instantly (< 5 seconds) and can be scaled up/down according to workloads.
*   **Custom Pools:** User-defined configurations. You can customize:
    *   *Node Family:* Virtual machine type. Typically, **Memory-Optimized** VM nodes perform best for Spark workloads.
    *   *Autoscale:* Enables dynamic cluster scaling by specifying a minimum and maximum node size.
    *   *Dynamic Allocation:* Dynamically allocates executor processes on worker nodes based on data volumes.
*   **Capacity Override:** Fabric administrators can completely disable custom pools at the capacity settings level.

### 🌐 Runtimes and Custom Environments
*   **Spark Runtime:** A pre-packaged combination of core software versions (Apache Spark, Delta Lake, Python, Java, Scala).
*   **Environments:** Custom environment configurations in workspaces allow you to:
    *   Select specific default Spark Runtimes.
    *   Install public libraries from the **Python Package Index (PyPI)** or upload custom `.whl` files.
    *   Override default Spark configuration properties.
    *   Attach custom pools and upload resource files.

### ⚙️ High-Yield Advanced Optimizations

#### 1. Native Execution Engine (Vectorized Processing)
A vectorized engine that runs Spark queries directly on the lakehouse infrastructure, vastly improving performance when reading large Parquet/Delta tables.
*   **Properties to Enable:**
    *   `spark.native.enabled: true`
    *   `spark.shuffle.manager: org.apache.spark.shuffle.sort.ColumnarShuffleManager`
*   **Notebook Level Inline Enable:**
    ```json
    %%configure 
    { 
       "conf": {
           "spark.native.enabled": "true", 
           "spark.shuffle.manager": "org.apache.spark.shuffle.sort.ColumnarShuffleManager" 
       } 
    }
    ```

#### 2. High Concurrency Mode
Enables multiple users or pipelines to **share the same Spark session** across different concurrent notebooks/jobs. 
*   **Benefit:** Prevents resource wastage and reduces startup delays.
*   **Isolation:** Ensures variables or session contexts do not bleed across notebooks.

#### 3. Automatic MLFlow Logging
Fabric automatically logs machine learning model training parameters, metrics, and models using **MLFlow** without requiring explicit logging code. This can be toggled on/off in Workspace Settings.

---

## 3. Running Spark Code (Notebooks vs. Spark Job Definitions)

Microsoft Fabric offers two key workloads to edit, manage, and execute Spark code depending on whether the task is interactive or automated:

### 📓 1. Spark Notebooks
*   **Purpose:** Best for interactive data exploration, visualization, collaboration, and rapid prototyping.
*   **Key Characteristics:**
    *   Uses **cells** containing either Markdown (text/images) or Executable Code.
    *   Provides **immediate feedback** by rendering results and charts directly below code cells.
    *   Allows code in multiple languages (PySpark, SQL, Scala, R) within the same notebook using language magic commands (e.g., `%%sql`).

### 📦 2. Spark Job Definitions (SJD)
*   **Purpose:** Best for automating non-interactive production scripts as part of scheduled, recurring, or triggered ETL processes.
*   **Key Characteristics:**
    *   Runs compiled or plain script files (e.g., Python `.py` scripts, Java/Scala `.jar` files).
    *   Requires a **main script file** that contains the core program logic.
    *   Allows adding **reference files** (such as dependency libraries or helper utility scripts).
    *   Requires a default **Lakehouse reference** so the script knows which OneLake target to run against.
    *   Runs headless (no UI/cell feedback, output is logged in job run history).

---

## 4. Working with Spark DataFrames

While Spark natively uses Resilient Distributed Datasets (RDDs), data engineers work primarily with **DataFrames** (part of the Spark SQL library). Spark DataFrames behave similarly to Pandas but are fully optimized for distributed cluster processing.

### 📥 Loading Data & Schema Design

#### 1. Infer Schema (Default)
Spark scans the file to automatically guess data types.
```python
%%pyspark
df = spark.read.load('Files/data/products.csv', format='csv', header=True)
```
*   *Note:* Default cell language can be overridden using cell magics (e.g. `%%pyspark` for Python, `%%spark` for Scala).

#### 2. Explicit Schema (HIGH YIELD)
Defining the schema programmatically using Spark SQL types.
```python
from pyspark.sql.types import *

productSchema = StructType([
    StructField("ProductID", IntegerType(), nullable=False),
    StructField("ProductName", StringType(), nullable=True),
    StructField("Category", StringType(), nullable=True),
    StructField("ListPrice", FloatType(), nullable=True)
])

df = spark.read.load('Files/data/product-data.csv', format='csv', schema=productSchema, header=False)
```
*   > [!IMPORTANT]
    > **Performance Benefit:** Defining an explicit schema **significantly improves read performance** because Spark does not need to scan the entire data file to infer types.

### 🔄 DataFrame API Operations
DataFrame methods return a new DataFrame object, allowing operations to be **chained** together:

```python
# Chaining Selection, Filtering (Logical OR "|", Logical AND "&")
bikes_df = df.select("ProductName", "Category", "ListPrice") \
             .where((df["Category"] == "Mountain Bikes") | (df["Category"] == "Road Bikes"))

# Grouping and Aggregating
counts_df = df.select("ProductID", "Category").groupBy("Category").count()
```
*   *Alternative Select Syntax:* You can use a list bracket to select columns: `df["ProductID", "ListPrice"]`.

### 💾 Saving Data & Partitioning (Optimization)

#### 1. Saving to Parquet
Parquet is a highly compressed, columnar storage format preferred for downstream analytical ingestion.
```python
bikes_df.write.mode("overwrite").parquet('Files/product_data/bikes.parquet')
```

#### 2. Partitioning Output Files (`partitionBy`)
Partitioning splits your dataset physically on disk into a hierarchical folder structure based on the values of one or more columns.
```python
bikes_df.write.partitionBy("Category").mode("overwrite").parquet("Files/bike_data")
```
*   **Physical Folder Structure:** 
    ```text
    Files/bike_data/
    ├── Category=Mountain Bikes/
    │   └── part-00000.snappy.parquet
    └── Category=Road Bikes/
        └── part-00000.snappy.parquet
    ```
*   > [!TIP]
    > **Performance Optimization:** Partitioning eliminates unnecessary disk I/O. When a user runs a query with a filter (e.g., `WHERE Category = 'Road Bikes'`), Spark reads **only** the relevant folder on disk, completely skipping the rest.

#### 3. Reading Partitioned Data
*   **Reading a specific partition:**
    ```python
    road_bikes_df = spark.read.parquet('Files/bike_data/Category=Road Bikes')
    ```
*   > [!WARNING]
    > **Column Omission:** When you load data from a specific partition folder directly, the partitioning column (`Category` in this case) is **omitted** from the resulting DataFrame schema, as its value is constant based on the folder path.

---

## 5. Working with Spark SQL

Spark SQL enables querying data cataloged in the Spark Metastore using standard relational SQL expressions.

### 🗄️ Relational Database Objects in the Spark Catalog
The Spark catalog acts as a schema registry. You can register dataframes as relational objects:

#### 1. Temporary Views
*   **Command:** `df.createOrReplaceTempView("products_view")`
*   **Lifecycle:** **Session-scoped**. Automatically destroyed when the notebook session closes, the Spark pool is shut down, or the session timeouts.

#### 2. Managed Tables (HIGH YIELD)
*   **Command:** `df.write.format("delta").saveAsTable("products")`
*   **Format:** Preferred format is **Delta** (Delta Lake format), which brings ACID transactions, time-travel versioning, and streaming compatibility.
*   **Storage Location:** Written directly inside the **`Tables/`** directory of the Lakehouse.
*   *Behavior on Delete:* Deleting a managed table **deletes both the catalog metadata AND the underlying Delta files** on OneLake.

#### 3. External Tables (HIGH YIELD)
*   **Command:** Created using `spark.catalog.createExternalTable()`.
*   **Storage Location:** Points to a folder path in external storage (typically the **`Files/`** folder of your Lakehouse).
*   *Behavior on Delete:* Deleting an external table **ONLY deletes the metadata definition** from the Spark catalog; the raw data files remain untouched in OneLake.

---

## 6. Querying Data via Spark SQL API vs. Cell Magics

You can query tables in the catalog in two different ways depending on your use case:

### 🐍 1. Programmatic API (Returning a DataFrame)
Use this within PySpark code blocks to select data, apply SQL filters, and output another DataFrame object for further processing.
```python
bikes_df = spark.sql("SELECT ProductID, ProductName, ListPrice FROM products WHERE Category IN ('Mountain Bikes', 'Road Bikes')")
display(bikes_df)
```

### 📊 2. Cell Magic (Pure SQL Query)
Use this in a notebook for quick, visual analysis of your catalog tables.
```sql
%%sql

SELECT Category, COUNT(ProductID) AS ProductCount
FROM products
GROUP BY Category
ORDER BY ProductCount DESC
```
*   The results are automatically rendered below the cell as a formatted table or interactive chart.

---

## 7. Data Visualization inside Spark Notebooks

Visualizing data is crucial for exploratory data analysis (EDA). Fabric notebooks provide both codeless and code-based charting methods:

### 🎛️ 1. Built-in Notebook UI Charts
*   **How it works:** When displaying a DataFrame or running a SQL query, the output window displays a **Table** tab and a **Chart** tab below the code cell.
*   **Usage:** You can visually configure charts (bar, line, pie, scatter) using a simple configuration sidebar directly in the UI without writing code.
*   **Limitation:** Good for quick visual summaries but lacks complex customization options.

### 🐍 2. Code-Based Visualization (Python Graphics Libraries)
For fine-grained layout control, you can use standard Python plotting libraries like **Matplotlib** or **Seaborn**.

#### 💡 The Pandas Conversion Constraint (HIGH YIELD)
Spark DataFrames are distributed datasets across worker nodes, which graphics libraries cannot plot directly. 
*   **Rule:** You **must** convert the Spark DataFrame to a localized Python **Pandas DataFrame** using the **`.toPandas()`** method before plotting it.
*   *Warning:* Converting very large Spark DataFrames to Pandas can crash the driver (head node) because it pulls all distributed data into the single driver node's local memory. Aggregating/summarizing the data *before* calling `.toPandas()` is standard practice.

#### Example Plotting Code:
```python
from matplotlib import pyplot as plt

# 1. Query catalog data and convert to Pandas DataFrame
data_pdf = spark.sql("SELECT Category, COUNT(ProductID) AS ProductCount FROM products GROUP BY Category").toPandas()

# 2. Clear plot area and set figure dimensions
plt.clf()
fig = plt.figure(figsize=(12, 8))

# 3. Build bar plot using Pandas columns
plt.bar(x=data_pdf['Category'], height=data_pdf['ProductCount'], color='orange')

# 4. Customize labels & grid properties
plt.title('Product Counts by Category')
plt.xlabel('Category')
plt.ylabel('Products')
plt.xticks(rotation=70)

# 5. Render plot inline
plt.show()
```
