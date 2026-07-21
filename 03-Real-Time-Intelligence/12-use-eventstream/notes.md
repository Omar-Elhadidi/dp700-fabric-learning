# M12: Use Eventstream in Microsoft Fabric

Eventstream in Real-Time Intelligence enables you to capture, transform, and route real-time events without needing to write code.

## Key Benefits
* **No-Code Data Pipelines:** Drag-and-drop canvas for creating event processing logic.
* **Multiple Source Connectors:** Ingest data from Azure Event Hubs, Azure IoT Hub, Azure Storage, Apache Kafka, CDC feeds, etc.
* **Flexible Routing:** Direct processed streaming data to multiple destinations for storage, analytics, or automated action.

---

## 1. Introduction

* **Core Purpose:** Eventstreams capture streaming data from various sources, apply optional in-flight transformations, and route data to designated targets.
* **Ease of Use:** Visual drag-and-drop designer eliminates the need to write custom stream-processing code.

---

## 2. Components of Eventstream

An Eventstream works as a visual pipeline dragging-and-dropping nodes to ingest, process, and route data without managing infrastructure.

### The Eventstream Canvas
* Visual editor showing data flowing through nodes in real time.
* Allows previewing data at each stage of the pipeline.

### Core Component Breakdown
1. **Sources:** Where event data originates (Microsoft, Azure, or non-Microsoft/third-party platforms).
2. **Transformations:** In-flight processing operators applied before data reaches storage:
   * *Operators:* `Filter`, `Manage fields`, `Aggregate`, `Group by`, `Expand`, `Join`, and custom `SQL code`.
3. **Destinations:** Where transformed event data is delivered:
   * *Targets:* KQL Database tables in Eventhouse, Lakehouse tables, Custom Endpoints, Derived Streams (for sub-processing), or Fabric Activator (for automated alerts).

---

## 3. Eventstream Transformations

Transformations allow in-flight cleaning, enriching, reshaping, and summarizing of real-time data before it hits destinations.

### Key Transformation Scenarios
* **Data Quality:** Filtering out corrupted/incomplete events early.
* **Content-Based Routing:** Splitting data paths based on payload values.
* **Data Enrichment:** Adding calculated fields, renaming columns, converting types.
* **Aggregation/Summarization:** Calculating metrics over temporal windows.
* **Format Standardization:** Normalizing schemas across diverse stream sources.

### No-Code Transformation Operators
1. **Filter:** Keeps events meeting specific logical conditions (e.g., `temperature > 80°`, `status = 'error'`).
2. **Manage Fields:** Adds, removes, renames, or changes data types of incoming columns.
3. **Aggregate:** Calculates running metrics (`Sum`, `Min`, `Max`, `Avg`) as events arrive over periods of time.
4. **Group By:** Computes aggregations across time windows. Supports:
   * *Tumbling Windows:* Fixed, non-overlapping time intervals (e.g., every 5 minutes).
   * *Sliding Windows:* Overlapping time intervals.
5. **Union:** Combines events from multiple stream nodes into a single table (requires matching field names & data types; non-matching fields dropped).
6. **Join:** Combines two streams based on matching join keys.
7. **Expand:** Flattens array fields into separate individual rows.

### Multi-Step Pipeline Example
* `Filter` (remove sensor errors) -> `Manage Fields` (add priority score) -> `Group By` (calculate hourly averages by location) -> `Route` (alerts to Activator, summaries to Lakehouse).

---

## 4. Module Summary

*   **Microsoft Fabric Eventstream** enables real-time, no-code data ingestion, transformation, and multi-destination routing.
*   **Key Source Types:** Azure Event Hubs, IoT Hubs, Kafka, CDC feeds, Azure/Fabric system events.
*   **Key Destination Types:** KQL Databases (Eventhouse), Lakehouse Delta tables, Activator rules, Custom endpoints, Derived sub-streams.
*   **Transformation Operators:** Filter, Manage Fields, Aggregate, Group By (Tumbling/Sliding windows), Union, Join, Expand.
