# M11: Get started with Real-Time Intelligence in Microsoft Fabric

Real-Time Intelligence in Microsoft Fabric helps you ingest, process, store, visualize, and act on data in motion to get insights from events as they happen.

## Learning Objectives
In this module, you learn how to:
- Describe real-time analytics, streams, and events.
- Understand Real-Time Intelligence components in Fabric.
- Ingest data and transform data in motion.
- Store and query data in KQL databases.
- Visualize streaming data with Real-Time Dashboards.
- Automate actions with Activator.

---

## 1. What is Real-Time Data Analytics?

**Real-time analytics** (or near real-time analytics) is the practice of processing, analyzing, and acting on data as it is generated, typically within seconds to minutes of when events occur, as opposed to analyzing static historical snapshots.

### Core Concepts
*   **Events:** Records of things that happen in a system at a specific moment (e.g., website clicks, patient vital changes, stock price changes, sensor readings).
*   **Streams:** A continuous sequence of events ordered by time. Streams are the delivery mechanism carrying events from their origin to where they are processed.

### Components of a Real-Time Analytics Solution
1.  **Real-time data ingestion:** Collecting data simultaneously from multiple sources (CDC, sensors, APIs, logs).
2.  **Stream processing:** Transforming, filtering, aggregating, and analyzing data *while it flows* with minimal latency.
3.  **Low-latency storage:** Specialized databases optimized for high-velocity writes and fast querying.
4.  **Interactive dashboards:** Visualizations that auto-update as new data arrives.
5.  **Automated decision making:** Event-driven rules and triggers for alerts/actions based on real-time conditions.

Real-Time Intelligence in Microsoft Fabric brings all these capabilities into a single platform using Eventstreams (ingestion), Eventhouses (storage), Real-Time hub (discovery), Real-Time Dashboards (visualization), and Activator (automation).

---

## 2. Real-Time Intelligence in Microsoft Fabric

Real-Time Intelligence is an integrated suite in Fabric that handles streaming data from capture through to automated response.

### Core Components
*   **Eventstreams:** Captures streaming data from various sources. Acts as the ingestion and in-flight processing layer to filter, enrich, transform, and route data.
*   **Eventhouses:** Stores time-series data using KQL (Kusto Query Language) databases. Optimized for fast ingestion of streaming data and deeply integrated with OneLake.
*   **KQL Queryset:** A workspace for running, managing, saving, and sharing queries against KQL databases. Supports both KQL and T-SQL syntax.
*   **Real-Time Dashboard:** Visualizations that connect directly to KQL databases and refresh automatically as new data arrives for interactive exploration.
*   **Activator:** Continuously monitors streaming data against user-defined rules/thresholds to trigger automated actions (e.g., sending notifications, starting Power Automate workflows, or executing Fabric pipelines).
*   **Real-Time Hub:** The central catalog and discovery center for all data-in-motion.
    *   Allows subscription to Fabric workspace events (like file creation in OneLake or job status changes).
    *   Connects to Azure events (like Azure Blob Storage) and external sources (IoT Hub, Service Bus, CDC feeds).

### Common Use Cases
*   **Delivery Tracking:** Monitoring vehicle GPS streams.
*   **Equipment Monitoring:** Tracking machine temperature for predictive maintenance.
*   **Fraud Detection:** Analyzing purchase streams to instantly block suspicious behavior.
*   **System Health:** Tracking application errors to maintain uptime.

---

## 3. Ingest and Transform Real-Time Data

There are two primary approaches for ingesting streaming data into Fabric: using **Eventstreams** or **Direct ingestion** to a KQL database.

### Approach 1: Eventstreams
Eventstreams act like a plumbing system (source -> transformations -> destination) to bring real-time events into Fabric, transform them, and route them.

*   **Data Sources:**
    *   *Microsoft sources:* Azure Event Hubs, Azure IoT Hubs, Azure Service Bus, database CDC feeds.
    *   *Azure & Fabric events:* Azure Blob Storage events, Fabric workspace/job changes, OneLake data changes.
    *   *External sources:* Apache Kafka, Google Cloud Pub/Sub, MQTT (preview).
*   **Transformations:** Processes applied *in-flight* before data reaches its destination (e.g., filter, aggregate, group by, expand, join, SQL code).
*   **Destinations:** Where processed data lands (e.g., KQL database in Eventhouse, Lakehouse, derived stream, Activator, custom endpoint).

### Approach 2: Direct Ingestion to KQL Database
Data can bypass an Eventstream and be ingested directly into a KQL database from sources like local files, Azure storage, Amazon S3, Event Hubs, or OneLake via the "Get data" option.

*   **Update Policies (Transformation):** When using direct ingestion, transformation occurs *after* the data lands. Update policies are automation mechanisms triggered when new data is written to a source table. They run a query on the ingested data and save the transformed result into a destination table.

---

## 4. Store and Query Real-Time Data

KQL databases inside an Eventhouse are the primary storage for real-time data flowing from Eventstreams. 

### Eventhouse Architecture
*   **KQL Databases:** Real-time optimized stores hosting tables, stored functions, materialized views, shortcuts, and data streams. Optimized for time-series data; they index incoming data by ingestion time and partition it for query performance.
*   **KQL Querysets:** A collection/workspace of queries. Supports both KQL (Kusto Query Language) and a subset of T-SQL.

### Kusto Query Language (KQL) Basics
KQL is the native language for analyzing large volumes of structured, semi-structured, and unstructured data in Fabric, Azure Data Explorer, and Microsoft Sentinel.

**Syntax Example:**
```kql
stock
| where ["time"] > ago(5m)
| summarize avgPrice = avg(todouble(bidPrice)) by symbol
| project symbol, avgPrice
```

### Advanced Data Processing Commands
Beyond basic queries, you can automate processing in a KQL database using:
1.  **Update policies:** Auto-transform incoming data and save to another table.
2.  **Materialized views:** Precalculate and store summary results for fast querying of massive datasets.
3.  **Stored functions:** Reusable query logic saved across multiple queries.

### Querying Alternatives
*   **T-SQL:** Eventhouses support a subset of T-SQL (e.g., `SELECT TOP 10 * FROM stock;`) for those familiar with SQL syntax.
*   **Copilot:** Copilot for Real-Time Intelligence can auto-generate KQL queries based on natural language prompts.

---

## 5. Visualize Real-Time Data

Fabric provides two primary ways to visualize your streaming data directly from a KQL database:

1.  **Real-Time Dashboards:**
    *   Dashboards consist of **tiles**, where each tile is driven by an underlying KQL query.
    *   Dashboards can be created from scratch in a workspace or built directly from a KQL queryset in an Eventhouse.
    *   By default, query results display as a table, but they can be customized into various charts/graphs.
    *   Tiles are highly interactive—users can drill in, filter, aggregate, and alter visualization types on the fly.
2.  **Power BI Reports:**
    *   You can also connect a Power BI dataset directly to your KQL database to create rich, real-time Power BI reports leveraging DirectQuery to reflect the latest streaming data.

---

## 6. Automate Actions with Activator

**Activator** is a technology in Fabric that enables automated processing of events that trigger actions. For example, sending an email when an Eventstream value deviates from a range, or running a Spark notebook when a dashboard is updated.

### Core Concepts of Activator
1.  **Events:** Each record in a stream representing something that occurred at a specific point in time.
2.  **Objects:** The business entity represented by the event data (e.g., a sales order, a delivery truck, a sensor).
3.  **Properties:** Fields in the event mapped to aspects of the business object's state (e.g., `temperature` field representing the sensor's current heat, or `total_amount` for an order).
4.  **Rules:** The core logic setting conditions under which an action is triggered, based on the property values of objects (e.g., `IF sensor.temperature > 100 THEN send_maintenance_email()`).

### Common Use Cases
*   **Logistics:** Alerting when shipments aren't updated within a time frame or monitoring delivery delays.
*   **Retail/Inventory:** Alerting store managers to move food if a freezer fails, or initiating marketing if sales drop.
*   **IT/Operations:** Flagging app/website UX issues in real-time or responding to pipeline failures immediately.
*   **Finance:** Sending alerts when a customer balance crosses a specific threshold.

---

## 7. Module Summary

*   **Real-Time Intelligence (RTI)** in Microsoft Fabric empowers end-to-end management of data-in-motion: ingestion, transformation, storage, visualization, and automated action.
*   **Real-Time Hub:** Central discovery catalog for workspace events, Azure events, and external streaming sources.
*   **Eventstreams:** In-flight ingestion and stream-processing engine (filter, aggregate, transform, route).
*   **Eventhouses & KQL Databases:** Scalable, low-latency time-series storage querying with KQL/T-SQL.
*   **Real-Time Dashboards & Power BI:** High-frequency, interactive visualization tools.
*   **Activator:** Event-driven rule monitoring engine acting on Object-Property condition thresholds.
