# 🧪 Exercise 3: Analyze Data with Apache Spark in Microsoft Fabric

This hands-on exercise focuses on using an Apache Spark Notebook inside Microsoft Fabric to ingest raw files, perform exploratory data analysis using the PySpark DataFrame API, query data using Spark SQL, visualize results, and save the final cleaned data as a managed Delta Table.

---

## 1. Setting Up the Lab Environment
1.  Open your Microsoft Fabric workspace.
2.  Select your existing **Lakehouse** (or create a new one called `Lakehouse_1` if needed).
3.  Upload the raw sales data:
    *   Under the **Files** section, create a folder named `data`.
    *   Upload the raw `products.csv` (or standard product/sales csv provided in the lab).
    *   *(Note: You can fetch it directly using code in cell 1 as shown below).*

---

## 2. Reading Data into a Spark DataFrame
Create a new **Notebook** in your workspace, link it to your Lakehouse, and run the following PySpark code to read the raw CSV:

```python
# Read products CSV into a DataFrame
df = spark.read.format("csv") \
    .option("header", "true") \
    .option("inferSchema", "true") \
    .load("Files/data/products.csv")

# Display the first few rows
display(df)
```

---

## 3. Exploratory Data Analysis & Transformations
Using the PySpark DataFrame API, perform basic operations to clean and aggregate the products data.

### Filter and Select
```python
# Filter products that are active or within a specific price range
df_filtered = df.filter(df['ListPrice'] > 100).select('ProductNumber', 'Name', 'ListPrice')
display(df_filtered)
```

### Aggregate Data
```python
from pyspark.sql.functions import count, avg

# Group by category and find average price and total count
df_summary = df.groupBy("ProductCategory") \
    .agg(count("ProductNumber").alias("TotalProducts"), 
         avg("ListPrice").alias("AveragePrice"))
display(df_summary)
```

---

## 4. Querying Data with Spark SQL
You can register your DataFrames as temporary views and query them using standard SQL syntax.

```python
# Register DataFrame as a Temporary View
df.createOrReplaceTempView("products_view")
```

Now, create a new cell, switch the language to **Spark SQL** (using the `%%sql` magic command), and run:

```sql
%%sql
SELECT ProductCategory, count(ProductNumber) AS ProductCount, max(ListPrice) AS MaxPrice
FROM products_view
GROUP BY ProductCategory
ORDER BY ProductCount DESC
```

---

## 5. Visualizing Data in Spark Notebooks
Use the built-in Charting tool or code-based libraries to visualize your findings.

### Code-Based Visualization (Matplotlib & Seaborn)
```python
import matplotlib.pyplot as plt
import seaborn as sns

# Convert the summary DataFrame to Pandas for charting
pdf = df_summary.toPandas()

# Create a bar chart showing average price by category
plt.figure(figsize=(10, 6))
sns.barplot(data=pdf, x="ProductCategory", y="AveragePrice")
plt.xticks(rotation=45)
plt.title("Average Product Price by Category")
plt.show()
```

---

## 6. Saving Data as a Managed Delta Table
Save the cleaned and processed dataset as a managed Delta table in the Lakehouse's `Tables` directory.

```python
# Write DataFrame as a managed Delta table
df.write.format("delta").mode("overwrite").saveAsTable("dbo.products")
```

---

## 7. Verification Screenshots

### Spark Notebook Execution (PySpark DataFrame Output)
![Spark Notebook PySpark cell execution](screenshots/03_spark_dataframe.png)

### Spark SQL Query Output
![Spark SQL Query execution cell](screenshots/03_spark_sql.png)

### Populated Delta Table in Lakehouse Tables
![Delta table registered under Lakehouse Tables](screenshots/03_lakehouse_table.png)
