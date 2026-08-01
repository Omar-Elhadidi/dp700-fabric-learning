# M19: Monitor a Microsoft Fabric Data Warehouse

Learn how to monitor a Microsoft Fabric Data Warehouse to manage compute capacity costs, diagnose query performance issues, track usage trends, and analyze historical execution data.

## Learning Objectives
In this module, you learn how to:
- Monitor Fabric **Capacity Metrics** to understand CPU consumption and capacity smoothing/throttling.
- Use **Dynamic Management Views (DMVs)** for real-time session and execution monitoring.
- Query historical execution metrics via the **`queryinsights`** system schema.
- Utilize the **Monitoring Hub** to track pipeline and dataflow execution statuses.

---

## 1. Introduction

* **Core Purpose:** Monitoring a Fabric Data Warehouse is essential for controlling capacity unit (CU) consumption costs, diagnosing long-running queries, and uncovering data usage patterns.
* **Key Monitoring Angles:**
  1. **Capacity & Cost Monitoring:** Tracking Capacity Unit (CU) usage via the Fabric Capacity Metrics App.
  2. **Real-Time Query Diagnostics:** Using SQL DMVs (`sys.dm_exec_requests`, `sys.dm_exec_sessions`).
  3. **Historical Performance Analytics:** Querying the `queryinsights` schema (retains 30 days of query history).

---

## 2. Monitor Capacity Metrics

Fabric billing is based on **Capacity Units (CUs)**. Capacity is a shared pool of compute resources determined by your license (F-SKUs or P-SKUs).

### 💡 Warehouse CU Consumption Drivers
In Data Warehouse workloads, CUs are consumed by:
* Data **Read** operations (T-SQL queries, Power BI Direct Lake reports).
* Data **Write** operations (`INSERT`, `UPDATE`, `DELETE`, `MERGE`, `COPY INTO`, `CTAS`).
* OneLake storage reads/writes and transaction log file updates.

### 📱 Microsoft Fabric Capacity Metrics App (EXAM HIGH YIELD)
An administrative Power BI app deployed in the Fabric environment to monitor CU consumption and system health.

* **Key Functions:**
  * **Filter Workloads:** Isolate utilization specifically for **Warehouse** items.
  * **Track Trends:** Analyze peak CU consumption vs. background smoothing averages.
  * **Detect Throttling:** Identifies if workloads are being throttled due to exceeding purchased capacity thresholds (over-utilization).
  * **Cost Optimization:** Helps admins decide whether to scale up/down F-SKUs, pause capacity, or optimize long-running queries.

---

## 3. Monitor Current Activity (Real-Time DMVs)

Use **Dynamic Management Views (DMVs)** to query real-time connection status, active sessions, and currently executing T-SQL queries.

### 🔍 Core Monitoring DMVs (EXAM HIGH YIELD)
* **`sys.dm_exec_connections`:** Returns details about client network connections (e.g., `client_net_address`).
* **`sys.dm_exec_sessions`:** Returns details about authenticated user sessions (e.g., `login_name`, `session_id`).
* **`sys.dm_exec_requests`:** Returns details about currently active SQL requests (e.g., `command`, `status`, `start_time`, `total_elapsed_time`).

### 🛠️ Identifying Running Long Queries Pattern
Join these three DMVs on `session_id` to inspect running queries in the current database ordered by duration:

```sql
SELECT 
    s.session_id,
    s.login_name,
    c.client_net_address,
    r.command,
    r.start_time,
    r.total_elapsed_time
FROM sys.dm_exec_connections AS c
INNER JOIN sys.dm_exec_sessions AS s 
    ON c.session_id = s.session_id
INNER JOIN sys.dm_exec_requests AS r 
    ON r.session_id = s.session_id
WHERE r.status = 'running' 
  AND r.database_id = DB_ID()
ORDER BY r.total_elapsed_time DESC;
```

---

## 4. Monitor Queries (`queryinsights` System Schema)

The **`queryinsights`** schema retains **30 days** of historical, aggregated query performance data.

### 📜 The 3 Query Insights System Views (HIGH YIELD)
1. **`queryinsights.exec_requests_history`:** Details of every completed SQL request.
   ```sql
   -- Find queries executed in the past hour
   SELECT start_time, login_name, command 
   FROM queryinsights.exec_requests_history 
   WHERE start_time >= DATEADD(MINUTE, -60, GETUTCDATE());
   ```
2. **`queryinsights.long_running_queries`:** Aggregates queries ranked by execution time.
   ```sql
   -- Identify recurring long-running queries
   SELECT last_run_command, number_of_runs, median_total_elapsed_time_ms, last_run_start_time 
   FROM queryinsights.long_running_queries 
   WHERE number_of_runs > 1 
   ORDER BY median_total_elapsed_time_ms DESC;
   ```
3. **`queryinsights.frequently_run_queries`:** Details on query invocation frequency and failure rates.
   ```sql
   -- Find most frequent queries and check failure counts
   SELECT last_run_command, number_of_runs, number_of_successful_runs, number_of_failed_runs 
   FROM queryinsights.frequently_run_queries 
   ORDER BY number_of_runs DESC;
   ```

### ⚡ Critical Query Insights Behaviors (EXAM MUST-KNOW)
* **15-Minute Latency:** Query insights are not real-time — completed queries may take **up to 15 minutes** to reflect in these views. (Use DMVs for real-time live monitoring!).
* **Auto-Parameterization:** Queries with differing literal values in `WHERE` clauses (e.g., `WHERE date = '2023-01-01'` vs `WHERE date = '2024-01-01'`) are automatically parameterized and aggregated as the **same command** in `long_running_queries` and `frequently_run_queries`.

---

## 5. Module Summary

* **Capacity Monitoring:** Use the admin-installed **Fabric Capacity Metrics App** to track CU utilization trends, analyze read/write/storage costs, and detect **throttling** (over-capacity usage).
* **Real-Time DMVs:** Live activity monitoring via T-SQL:
  * `sys.dm_exec_connections` (network/IPs)
  * `sys.dm_exec_sessions` (logins/session IDs)
  * `sys.dm_exec_requests` (active SQL statements, execution status, elapsed time)
* **Query Insights (`queryinsights` Schema - 30-Day Retention):** Historical performance views with auto-parameterization:
  * `exec_requests_history` (raw execution log)
  * `long_running_queries` (queries ranked by median duration)
  * `frequently_run_queries` (queries ranked by execution count and failure rates)
  * *Note:* Query Insights has up to a **15-minute latency**.
