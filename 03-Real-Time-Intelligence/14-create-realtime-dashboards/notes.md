# M14: Create Real-Time Dashboards with Microsoft Fabric

Real-Time Dashboard is a capability in Microsoft Fabric used to create interactive data visualizations connected directly to KQL databases in Eventhouses.

## Learning Objectives
By the end of this module, you will be able to:
- Create a Real-Time Dashboard in Microsoft Fabric.
- Organize and filter dashboard data.
- Apply dashboard management and optimization techniques.

---

## 1. Introduction

* **Core Purpose:** Monitor fast-changing operational data (e.g., bike-share availability, fleet tracking) in real time rather than viewing static historical snapshots.
* **Mechanism:** Connects to Eventhouse KQL databases with auto-refresh capability to surface changing conditions immediately for rapid decision-making.

---

## 2. Get Started with Real-Time Dashboards

Real-Time Dashboards consist of **tiles**, where each tile displays a visualization driven by a KQL query against a KQL database in an Eventhouse.

### Data Source Authorization Modes
When creating a data source connection for a Real-Time Dashboard, you configure how permissions are applied:
1. **Pass-through identity:** Viewers access the data using their *own* user permissions.
2. **Dashboard editor's identity:** All viewers access the data using the *dashboard creator's* permissions.

### Tile Creation & Visualizations
* **Queries:** Each tile executes a KQL query against the underlying data source (e.g., retrieving the latest record ingested within a time range using `arg_max()`).
* **Visuals:** By default, tiles render query results as a **table**. You can edit the tile to change the visual type to bar charts, column charts, line charts, maps, cards, or add markdown text tiles.
* **Layout:** Multiple tiles can be added, resized, and arranged on a dashboard grid.

---

## 3. Organize and Filter Dashboard Data

Real-Time Dashboards provide native capabilities for structural organization, dynamic filtering, and automated refresh rates.

### Pages
* Multi-page containers allow organizing related content into logical groups (e.g., separate pages for distinct subject areas or data sources).

### Parameters (Dynamic Filtering)
* **Definition:** Allow dashboard viewers to filter tile data interactively (e.g., date ranges, category drop-downs).
* **Values:** Can be static text or dynamically populated from KQL query results.
* **KQL Reference Syntax:** Parameter variables are referenced in tile queries with an **underscore prefix** (e.g., `_selected_neighbourhoods`).
* **Pattern for Optional Filtering:**
  ```kql
  bikes
  | where ingestion_time() between (ago(30min) .. now())
      and (isempty(_selected_neighbourhoods) or Neighbourhood in (_selected_neighbourhoods))
  | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
  ```

### Auto Refresh & Minimum Refresh Rate
* Automatically updates tile data without manual page reloads.
* **Editors** set the default refresh rate and can enforce a **minimum refresh rate** to protect system performance and prevent excessive query overhead.
* **Viewers** can adjust their refresh rate during a session (subject to the minimum threshold set by the editor).

---

## 4. Dashboard Management and Optimization

To optimize dashboard maintainability and execution performance, Real-Time Dashboards support **Base Queries**.

### Base Queries
* **Concept:** A single shared query definition created via the *Base queries* menu that retrieves a common baseline dataset used across multiple tiles.
* **Benefits:**
  * Eliminates duplicate KQL logic across tiles.
  * Centralizes underlying data retrieval for easier maintenance.
* **Variable Syntax:** Base queries are assigned a variable name starting with an underscore (e.g., `_base_bike_data`).
* **Usage Pattern:**
  * **Base Query Definition (`_base_bike_data`):**
    ```kql
    bikes
    | where ingestion_time() between (ago(30min) .. now())
    | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
    ```
  * **Tile-Specific Query (referencing base query):**
    ```kql
    _base_bike_data
    | project Neighbourhood, latest_observation, No_Bikes, No_Empty_Docks
    | order by Neighbourhood asc
    ```

---

## 5. Module Summary

* **Real-Time Dashboards:** High-frequency, interactive data visualization tool in Fabric connected directly to Eventhouse KQL databases.
* **Identity Delegation:** Data source authorization supports *Pass-through identity* (user's permissions) or *Dashboard editor's identity* (creator's permissions).
* **Multi-Page Layout & Tiles:** Visual tiles (charts, tables, maps, cards, markdown) organized into distinct pages.
* **Dynamic Parameters:** Interactive filters defined using underscore-prefixed variables (e.g., `_selected_neighbourhoods`) evaluated directly inside KQL queries.
* **Refresh Governance:** Configurable auto-refresh rates with editor-enforced **Minimum Refresh Rates** to prevent excessive database query strain.
* **Base Queries:** Shared baseline query logic (`_baseQueryName`) referenced across multiple tiles to eliminate duplicate KQL code and optimize maintainability.
