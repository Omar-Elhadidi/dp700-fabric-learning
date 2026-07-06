# 🧪 Exercise 2: Ingest Data with a Pipeline in Microsoft Fabric

This exercise walks through building a complete, automated ELT pipeline in Fabric that cleans old staging files, copies raw CSV files from an HTTP source, and executes a parameterized PySpark Spark notebook to clean and write the data into a Delta table.

---

## 1. Core Architecture Pattern (ELT)

The pipeline orchestrates three sequential steps:
```
[Delete old CSV files] ──(On Completion)──> [Copy fresh raw CSV] ──(On Completion)──> [Load & Transform Notebook]
```

### 💡 High-Yield Exam Note: "On Completion" vs. "On Success"
This pipeline uses **On Completion** (Blue Arrow) dependencies rather than **On Success** (Green Arrow). This ensures that if the *Delete* task has nothing to delete (which can trigger a warning/non-success state), the pipeline does not stop and still proceeds to copy the fresh data.

---

## 2. Activity 1: Delete Data Activity
*   **Purpose:** Clear old data files before the copy operation to avoid mixing stales.
*   **Properties:**
    *   *Connection:* Target Lakehouse
    *   *File Path Type:* Wildcard file path (`Files/new_data/*.csv`)
    *   *Recursively:* Selected
    *   *Logging settings:* Unselected (improves performance for metadata-only deletes)

---

## 3. Activity 2: Copy Data Activity (HTTP Source to OneLake Files)

### Copy Job vs. Copy Data Activity (HIGH YIELD)
*   **Copy Job:** A standalone, simplified data movement assistant. It does **not** run inside a pipeline and cannot be orchestrated with other activities.
*   **Copy Data Activity:** An activity block configured **inside** a pipeline, allowing it to be linked with notebooks, script triggers, and conditional logic.

### Source Settings
*   **Connector Type:** HTTP
*   **Source URL:** `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/sales.csv`
*   **Authentication Kind:** Anonymous
*   **File Format Settings:**
    *   *File Format:* DelimitedText
    *   *Column Delimiter:* Comma (`,`)
    *   *Row Delimiter:* Line feed (`\n`)
    *   *First row as header:* Selected

### Destination Settings
*   **Target Connection:** Lakehouse
*   **Root Folder:** Files
*   **File Path:** Directory: `new_data` / File Name: `sales.csv`

---

## 4. Activity 3: Parameterized Spark Notebook (Transformation)

### Spark Notebook Configuration
1.  Create a Spark notebook in the workspace.
2.  **Cell 1 (Parameters Cell):** Define your default variable:
    ```python
    table_name = "dbo.sales"
    ```
    *   *Action:* In the cell's `...` menu, toggle **Parameter cell**. This registers it so the variable can be overridden dynamically by the calling Data Pipeline.
3.  **Cell 2 (PySpark Code):**
    ```python
    from pyspark.sql.functions import *

    # 1. Read raw CSV from Lakehouse Files directory
    df = spark.read.format("csv").option("header","true").load("Files/new_data/*.csv")

    # 2. Add month and year columns
    df = df.withColumn("Year", year(col("OrderDate"))).withColumn("Month", month(col("OrderDate")))

    # 3. Split CustomerName into FirstName and LastName
    df = df.withColumn("FirstName", split(col("CustomerName"), " ").getItem(0)).withColumn("LastName", split(col("CustomerName"), " ").getItem(1))

    # 4. Filter and reorder columns
    df = df["SalesOrderNumber", "SalesOrderLineNumber", "OrderDate", "Year", "Month", "FirstName", "LastName", "EmailAddress", "Item", "Quantity", "UnitPrice", "TaxAmount"]

    # 5. Load/write data as a Delta table (Append mode)
    df.write.format("delta").mode("append").saveAsTable(table_name)
    ```

### Pipeline Integration & Parameter Overriding
*   Add a **Notebook activity** to the pipeline canvas.
*   *Settings Tab:* Point it to your Spark notebook.
*   *Base Parameters (HIGH YIELD):* Add a parameter to override the default:
    *   **Name:** `table_name`
    *   **Type:** `String`
    *   **Value:** `dbo.new_sales`
*   *Result:* When the pipeline runs, the notebook executes but writes to `dbo.new_sales` instead of the default `dbo.sales`.
