# 🧪 Exercise 1: Create and Use Dataflows (Gen2) in Microsoft Fabric

This exercise walks through setting up a Lakehouse, ingesting CSV data via Dataflow Gen2, transforming it using Power Query M code, setting up a database destination, and orchestrating it using a Data Pipeline.

---

## 1. Environment Setup

### Workspace Licensing
*   To use Dataflows Gen2, the workspace **must** be assigned to a capacity license:
    *   **Fabric Trial**
    *   **Premium Capacity**
    *   **Fabric Capacity**

### Lakehouse Creation
*   Create a new Lakehouse in the workspace (Data Engineering workload). This serves as the target storage destination.

---

## 2. Ingestion & Transformation Configuration

### Source Data
*   **Format:** CSV File
*   **URL:** `https://raw.githubusercontent.com/MicrosoftLearning/dp-data/main/orders.csv`
*   **Connection Setting:** Create new connection, set **Authentication Kind** to **Anonymous** (since the GitHub URL is public and does not require credentials).

### Transformation (Custom Column)
*   **Task:** Extract the numeric month from the `OrderDate` column.
*   **Action:** Add Custom Column.
    *   *Column Name:* `MonthNo`
    *   *Data Type:* `Whole Number`
    *   *M Formula (Case-Sensitive):*
        ```powerquery
        Date.Month([OrderDate])
        ```
    *   *Data Type Check:* Ensure `OrderDate` is set to **Date** and `MonthNo` is set to **Whole Number**.

---

## 3. Data Destination Configuration (HIGH YIELD)

### Default Destination Behavior
*   If a Dataflow Gen2 is created by clicking **Get Data > New Dataflow Gen2** *inside* a Lakehouse, that Lakehouse is **automatically attached as the default destination**. You do not need to configure it manually.

### Destination Settings
*   **Destination Type:** Lakehouse (writes to a table named `orders`).
*   **Update Method Options:**
    *   **Replace:** Overwrites existing data in the target table on every run.
    *   **Append:** Inserts new rows at the end of the target table on every run (selected for this lab).

---

## 4. Pipeline Orchestration

To run the Dataflow automatically as part of a scheduled or sequential workflow:
1.  Create a new **Data Pipeline** named `Load data`.
2.  Add a **Dataflow activity** to the canvas.
3.  Select the activity, go to the **Settings** tab, and select your `Dataflow 1` from the dropdown.
4.  Run the pipeline. Once successful, go to your Lakehouse, refresh the tables, and view the loaded `orders` table.

---

## 5. Downstream Analytics (Power BI)
*   Data analysts can bypass the Lakehouse if needed and connect directly to the Dataflow Gen2 transformations using the **Power BI dataflows (Legacy) connector** in Power BI Desktop.
