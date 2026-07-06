# 🔄 Module 2: Orchestrate Processes and Data Movement with Microsoft Fabric

This module covers Data Factory pipelines inside Microsoft Fabric, focusing on orchestration, the Copy Data activity, templates, and monitoring.

---

## 1. Introduction & Architecture

*   **Core Purpose:** Automate and orchestrate Extract, Transform, and Load (ETL/ELT) processes to bring transactional data into analytical stores (Lakehouse, Data Warehouse, SQL Database).
*   **ADF Connection (HIGH YIELD):** Fabric Data Pipelines use the exact same architecture as **Azure Data Factory (ADF)**. The concepts of activities, orchestration flow, and control logic are identical.
*   **Run Methods:**
    *   *Interactive:* Run manually/on-demand directly in the Fabric UI for testing.
    *   *Automated:* Scheduled or event-triggered runs.

---

## 2. Core Pipeline Concepts

### A. Activities (The Executable Tasks)
Activities are connected in a sequence. The outcome of one activity directs the flow to the next.

| Activity Category | What it does | Examples |
|---|---|---|
| **Data Transformation / Movement** | Moves or processes data | - **Copy Data:** Bulk transfers data from source to destination.<br>- **Dataflow:** Runs a Dataflow Gen2 to transform data.<br>- **Notebook:** Executes a Spark notebook.<br>- **Stored Procedure:** Runs SQL code on a database.<br>- **Delete Data:** Clears existing data. |
| **Control Flow** | Manages pipeline execution logic | - Loops (e.g., `ForEach`, `Until`) <br>- Branching (e.g., `If Condition`, `Switch`) <br>- Variable/Parameter manipulation |

#### 💡 High-Yield Exam Tip:
*   *Activity Outcomes:* You link activities together using conditional paths: **Success** (Green), **Failure** (Red), or **Completion** (Blue).
*   *Classification:* A **Notebook** activity is classified as a *Data Transformation* activity, not control flow, even though you write code in it.

### B. Parameters (Reusability)
*   **Definition:** Variables that you pass into the pipeline at runtime.
*   **Why use them?** To avoid hardcoding values (e.g., folder paths, file dates, or source database credentials), allowing one pipeline to be reused for different datasets.

### C. Pipeline Runs
*   **Definition:** A single execution of a pipeline.
*   **Monitoring:** Every run gets a **unique Run ID**. You use this ID in the monitoring logs to investigate success/failure rates and track down errors.

---

## 3. The Copy Data Activity

### A. Core Concept
The **Copy Data** activity is used to transfer raw data directly from a source to a destination.
*   **No Transformation:** It does a direct copy. It does **not** clean, filter, or restructure data during the copy operation.
*   **Integration:** It is commonly combined with other activities to create an automation sequence. E.g.:
    1.  **Delete Data activity** (clears old staging files) ──>
    2.  **Copy Data activity** (fetches new raw file) ──>
    3.  **Notebook activity** (runs Spark code to transform the file and write to a table).

### B. The Copy Data Assistant (Wizard)
*   When adding this activity, Fabric opens a step-by-step graphical wizard to define the source connections and targets (Lakehouse, Warehouse, SQL Database, etc.).

---

### 🚨 HIGH-YIELD EXAM DECISION RULE: Copy Data vs. Dataflow Gen2

You will face scenario questions asking which tool to select. Use this decision matrix:

| Use **Copy Data Activity** if: | Use **Dataflow Gen2** if: |
|---|---|
| You want to copy data **exactly as it is** (raw format). | You need to apply transformations **during ingestion** (ETL). |
| You need maximum speed for copying large bulk datasets. | You need to merge data from multiple sources visually. |
| You plan to transform the data *later* using Spark Notebooks or SQL Stored Procedures (ELT). | You want a low-code/no-code visual interface (Power Query Online) to clean columns, filter rows, or calculate values. |

---

## 4. Pipeline Templates

*   **Definition:** Predefined, ready-to-use pipeline configurations built by Microsoft for common data movement scenarios (e.g., incremental copy, file staging, bulk loading).
*   **Access Point:** Select the **Templates** tile ("Choose a task to start") inside a new, empty pipeline canvas.
*   **Key Advantage:** Avoids building pipelines from scratch. You select a template that matches your workflow and then customize the connections or variables to fit your environment.

---

## 5. Run, Validate, and Monitor Pipelines

*   **Validate:** A crucial step before running. You select the **Validate** option on the canvas ribbon to verify that the pipeline's properties, parameters, and activity configurations are correct and free of syntax/logic errors.
*   **Run Methods:**
    *   *Interactive (Manual):* Run on-demand directly using the **Run** button for development testing.
    *   *Scheduled:* Automate runs using time-based or event-based schedules.
*   **Monitoring Run History:** You can track the success, failure, duration, and inputs/outputs of pipeline executions in two ways:
    1.  **Directly on the canvas:** In the output/monitoring pane at the bottom of the pipeline editor.
    2.  **Workspace item level:** Viewing the run history for that specific pipeline file from the main workspace folder.
    3.  **Monitoring Hub:** The centralized dashboard for tracking all jobs across the entire Fabric capacity.

