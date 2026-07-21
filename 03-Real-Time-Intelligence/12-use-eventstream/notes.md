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

## 3. Eventstream Sources and Destinations

Eventstream supports connecting to a wide array of sources and fanning out to multiple destinations simultaneously without performance collision.

### 📥 Supported Sources
* **Microsoft Cloud Sources:** Azure Event Hubs, Azure IoT Hubs, Azure Service Bus, Database CDC feeds.
* **Azure & Fabric System Events:** Azure Blob Storage events, Fabric Workspace changes, OneLake storage events, Fabric Job events.
* **External/Third-Party Sources:** Apache Kafka, Google Cloud Pub/Sub, MQTT.

### 📤 Supported Destinations
1. **Eventhouse (KQL Database):** Direct real-time event ingestion into KQL tables for fast time-series analytics.
2. **Lakehouse:** Transforms real-time events, converts them into **Delta Lake format**, and stores them into Lakehouse Delta tables.
3. **Derived Stream:** 
   * Enables **content-based routing** by creating transformed sub-streams.
   * Allows routing specific subsets of the main stream to different destinations (e.g., routing filtered anomaly events to Activator while routing full raw data to an Eventhouse).
4. **Fabric Activator:** Connects directly to real-time events to evaluate rules and trigger automated actions (emails, Power Automate workflows, notebooks).
5. **Custom Endpoint:** Routes real-time event data to external third-party systems or custom applications outside of Microsoft Fabric.

*Note: A single Eventstream can send data to **multiple destinations concurrently** without conflict.*

---

## 4. Eventstream Transformations

Transformations allow cleaning, enriching, and reshaping streaming data *in-flight* prior to landing at destinations.

### 🛠️ No-Code Transformation Operators
1. **Filter:** Keeps events matching specific conditions (e.g., `temperature > 80`, `status == "error"`). Drops non-matching events.
2. **Manage Fields:** Adds calculated fields, removes unneeded columns, renames fields, or casts data types to meet target table schemas.
3. **Aggregate:** Calculates running aggregations (`SUM`, `MIN`, `MAX`, `AVG`) every time a new event occurs over time intervals.
4. **Group By:** Calculates windowed aggregations across groups. Supports:
   * **Tumbling Windows:** Fixed-size, non-overlapping time intervals (e.g., 5-minute fixed chunks).
   * **Sliding Windows:** Fixed-size, overlapping time intervals (e.g., 5-minute averages evaluated every minute).
5. **Union:** Combines events from 2+ stream nodes into one output stream based on matching schema fields (non-matching fields are dropped).
6. **Join:** Combines two streams based on matching key conditions over time windows.
7. **Expand:** Flattens array fields, creating a separate output row for each item in an array.

---

## 5. Module Summary

* **Eventstream Engine:** A code-free, drag-and-drop visual pipeline for streaming data ingestion, in-flight processing, and routing.
* **Sources:** Broad support for Azure Event Hubs, IoT Hubs, Service Bus, Database CDC, Blob events, Fabric system events, Kafka, Google Pub/Sub, and MQTT.
* **Transformations:** In-stream operators (`Filter`, `Manage Fields`, `Aggregate`, `Group By`, `Union`, `Join`, `Expand`) prepare data before landing.
* **Destinations:** Concurrently route to Eventhouses (KQL), Lakehouses (Delta format), Derived Streams (content-based routing), Fabric Activators (alert rules), or Custom Endpoints.
