# M24: Monitor Activities in Microsoft Fabric

Learn how to monitor, diagnose, and troubleshoot data engineering activities (Dataflows Gen2, Data Pipelines, Spark Notebooks) using the Fabric Monitoring Hub and automate responses to job failures using Activator.

## Learning Objectives
In this module, you learn how to:
- Use the **Fabric Monitoring Hub** to track activity statuses, investigate job failures, and view run history across workspaces.
- Configure **Activator** to automatically trigger alerts (Email, Teams, Power Automate, Notebooks) on Fabric job events (e.g., pipeline failure, notebook error).
- Analyze execution details and logs for dataflows, pipelines, and Spark jobs.

---

## 1. Introduction

* **Core Challenge:** Detecting and resolving silent failures or performance degradations across scheduled pipelines, Dataflows Gen2, and Spark notebooks before stale data impacts business decisions.
* **Key Monitoring Components:**
  1. **Fabric Monitoring Hub:** Centralized single-pane-of-glass interface for tracking activity across all Fabric items and workspaces.
  2. **Activator Event Detection:** Automated rule engine triggering alerts (Teams/Email) or corrective actions when job failures/events occur.

---

## 2. Understand Monitoring

Monitoring tracks Fabric activity execution states, duration, and failures across chained data workflows (**Dataflow ➔ Lakehouse ➔ Spark Notebook ➔ Semantic Model ➔ Power BI Report**).

### 🔍 Activity Types & Monitoring Nuances (EXAM HIGH YIELD)
| Fabric Activity Type | Key Focus & Unique Failure Modes | High-Yield Exam Tip |
| :--- | :--- | :--- |
| **Data Pipelines** | Overall pipeline state & individual activity steps. | ⚠️ A pipeline can show **`Succeeded`** overall even if a nested optional activity fails! Must review individual activity run logs. |
| **Dataflow Gen2** | Table load steps, start/end time, duration. | Drill into activity details to locate specific table schema/type mismatch errors. |
| **Semantic Model Refreshes** | Refresh status & **retry counts**. | Watch for increasing retry counts—signals transient source connection issues before total failure. |
| **Spark Jobs / Notebooks** | Spark application logs, task duration, resource usage. | Unexpectedly long runs usually indicate **data skew** or unpartitioned joins. |
| **Eventstreams** | **Continuous** real-time data streaming (not batch). | Monitor throughput, ingestion latency, and streaming error rates. |

### 📊 Establishing a Monitoring Baseline
* **Define "Normal":** Establish expected run duration (e.g., Dataflow takes 3–5 min normally; a 20 min run indicates a bottleneck).
* **Automate Alerts:** Do not rely on manual portal checks—configure **Activator** rules for automated notifications.

---

## 3. Use the Microsoft Fabric Monitoring Hub

The **Monitoring Hub** (accessed via **`Monitor`** on the left navigation pane) aggregates activity logs from across the workspace into a single-pane-of-glass dashboard.

### 📋 Key Features & Retention Rules (EXAM HIGH YIELD)
* **Activity Statuses:** `Succeeded`, `Failed`, `In progress`, `Cancelled`.
* **View Details (`i` icon):** Inspects start/end time, duration, processed row counts, Spark app IDs, and error tracebacks.
* **30-Day Historical Runs:** Hover item → `...` → **`Historical runs`** displays up to **30 days** of historical execution data to analyze duration creep or failure patterns.
* **Schedule Failures Page:** Configures workspace-wide email alerts for scheduled job failures from one central screen (Requires **Contributor role** or higher).

### ⚡ Workspace Monitoring vs. Monitoring Hub (CRITICAL EXAM COMPARISON)
| Feature | Monitoring Hub | Workspace Monitoring (Eventhouse) |
| :--- | :--- | :--- |
| **Interface** | Visual UI dashboard in portal. | KQL Database / Eventhouse created in workspace. |
| **Query Capability** | Filtering & visual inspection only. | Full programmatic querying using **KQL** or **T-SQL**. |
| **Use Case** | Quick real-time check of recent activity & failures. | Advanced log analytics, cross-item correlation, & custom Real-Time Dashboards. |
| **Retention & Access**| 30 days visual history. | 30 days raw diagnostic log retention (Requires **Contributor role**). |
| **Configuration** | Built-in by default. | Must be explicitly enabled in **Workspace Settings ➔ Monitoring**. |

---

## 4. Respond to Fabric Events with Activator

While the Monitoring Hub shows past runs, **Activator** connects to live **Job Events** (via Real-Time Hub) to automate alerts and trigger downstream corrective or orchestration actions.

### 📡 Fabric Job Event Sources (Real-Time Hub)
Fabric items emit platform events: `Job created`, `Job failed` (`Microsoft.Fabric.JobEvents.ItemJobFailed`), `Job succeeded`, and `Job status changed`.

### ⚡ Activator Action Types
1. **Email:** Sends notification email with deep links to the Monitoring Hub.
2. **Teams:** Posts notifications to individuals, group chats, or Teams channels.
3. **Run a Fabric Activity (KEY DIFFERENTIATOR):** Automatically executes a pipeline, Spark notebook, dataflow, or function.
   * *Examples:* Run a data cleanup notebook after a failed partial load, or trigger a semantic model refresh when an upstream pipeline succeeds.

### 🆚 Activator vs. Schedule Failures Page (HIGH YIELD EXAM MATRIX)

| Requirement / Scenario | Use Schedule Failures Page | Use Activator |
| :--- | :---: | :---: |
| Centralized email-only alerts for scheduled failures | ✅ | |
| Post a failure message to a **Microsoft Teams channel** | | ✅ |
| React to job **success** (e.g., notify analysts on refresh success) | | ✅ |
| Automatically **trigger a follow-up pipeline/notebook** on event | | ✅ |
| Run a data cleanup job after a partial load failure | | ✅ |

---

## 5. Module Summary

* **Monitoring Hub:** Centralized visual UI dashboard for tracking real-time status across pipelines, dataflows, notebooks, and semantic models. Retains **30 days** of historical run data.
* **Workspace Monitoring:** Enable in Workspace Settings ➔ creates an **Eventhouse (KQL Database)** storing raw diagnostic logs for querying via **KQL or T-SQL** and building custom Real-Time Dashboards (requires Contributor role).
* **Schedule Failures Page:** Simple, central workspace-level **email-only** alerts for scheduled job failures.
* **Activator Automation:** Connects to live **Job Events** in Real-Time Hub (`ItemJobFailed`, `ItemJobSucceeded`). Supports **Teams notifications**, success alerts, and **automatically executing downstream activities** (running cleanup notebooks or pipelines).
