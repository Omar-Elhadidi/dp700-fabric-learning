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
