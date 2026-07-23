# M15: Use Activator in Microsoft Fabric

Fabric Activator is an event detection engine that automatically triggers actions when specific patterns or conditions are detected in streaming or business data sources.

## Learning Objectives
By the end of this module, you will be able to:
- Define data objects and properties in Activator.
- Create rules that evaluate conditions in your data.
- Configure actions that execute when rule conditions are met.

---

## 1. Introduction to Activator

**Activator** is Microsoft Fabric's event detection and rules engine within Real-Time Intelligence.

### How Activator Works
1. **Connect:** Ingests streaming data from real-time sources (Eventstreams, KQL databases, Power BI reports).
2. **Evaluate:** Continuously checks data against user-defined rules and conditions.
3. **Act:** Executes automated actions when conditions are met (e.g., sending Email alerts, posting Teams messages, launching Power Automate flows, or running Fabric Notebooks).

### Real-World Scenarios
* **Cold-Chain / Logistics:** Alerting dispatch when temperature-sensitive medicine transport experiences temperature spikes.
* **Manufacturing:** Notifying maintenance when equipment temperatures cross safe operating limits.
* **Supply Chain:** Triggering inventory reorders when stock drops below thresholds.
* **IT Operations:** Automatically restarting degraded services when performance metrics drop.
* **Healthcare:** Alerting clinical staff when patient monitoring devices detect critical metric changes.

---

## 2. Configure Activator for Your Data

Activator can evaluate conditions against incoming streaming data using two primary models: **Business Objects** and **Direct Alerts**.

### The Business Objects Model
Business objects represent the real-world entities you want to monitor (e.g., packages, IoT devices, customers).

* **Objects (Instances):** Represent individual instances uniquely identified by a key field (e.g., `PackageId`, `DeviceId`).
* **Properties (Attributes):** The specific data attributes monitored for each object instance (e.g., `Temperature`, `HoursInTransit`, `DeliveryState`).
* **Events (Data Flow):** Raw streaming records flowing in from data sources. Activator uses incoming event streams to continuously update property values for each object instance.

### Creating Objects from an Eventstream
1. Add **Activator** as a destination in an Eventstream.
2. Open Activator -> Select **Create new object**.
3. **Choose Unique Identifier:** Select the primary key field (e.g., `PackageId`).
4. **Select Properties:** Choose fields to track over time (e.g., `Temperature`).
5. As new streaming events arrive:
   * Existing object properties are updated to the latest values.
   * New unique identifier values automatically instantiate new Business Objects.

### Alternative Alerting Approaches (Direct Alerts)
Activator's underlying engine also powers direct alerting without explicitly creating business object hierarchies:
* **Dashboard Alerts:** Created directly from Real-Time Dashboard tiles.
* **System Event Alerts:** Monitoring Fabric workspace item changes or OneLake file operations.
* **KQL Queryset Alerts:** Created directly from KQL query results.

---

## 3. Create Rules in Activator

Rules define the specific conditions monitored on object properties and the actions executed when conditions are triggered.

### Anatomy of an Activator Rule Definition
Configured within the Activator **Definition pane** across four primary sections:

#### 1. Monitor (What to Watch)
* **Attribute:** Selects the target property from event data (e.g., `Temperature`).
* **Summarization:** Smooths out signal noise and brief anomalies over time windows:
  * `Average` (averages over window), `Minimum` / `Maximum` (captures extremes), `Count` (detects event frequency/failures), `Total` (summation).
* **Timing Settings:**
  * **Window Size:** Amount of historical data included in the calculation (e.g., last 10 minutes).
  * **Step Size:** How frequently the calculation recalculates (e.g., every 5 minutes).

#### 2. Condition (When to Act)
* **Detection Approaches:**
  * *Threshold Monitoring:* Values cross safe limits (e.g., `Temperature > 68°F`).
  * *Change Detection:* Monitors trends (e.g., `Temperature increases above baseline`).
  * *Range Monitoring:* Tracks entry into or exit from safe operating bands.
  * *Missing Data:* Detects sensor/pipeline failures (e.g., `No events received for > 30 minutes`).
* **Occurrence Behavior:**
  * `Every time` (immediate alerts per occurrence).
  * `When it has been true for [duration]` (prevents false positives by requiring persistent condition state).

#### 3. Property Filter (Focus Scope)
* Restricts rule evaluation to a targeted subset of events rather than all events in the stream.
* You can combine up to **3 property filters** (e.g., `ColdChainType == "medicine"` AND `City == "Seattle"`).

---

## 4. Configure Actions in Activator

Actions complete the Activator pipeline by executing automated notifications or workflows when rule conditions are met.

### ⚡ The 4 Action Types in Activator
1. **Email Actions:**
   * Sends formatted email messages containing context.
   * Best for non-urgent notifications or comprehensive reviews where immediate real-time response is not required.
2. **Teams Actions:**
   * Sends real-time messages directly to Microsoft Teams channels or individual users.
   * Best for urgent operational alerts requiring immediate team coordination.
3. **Power Automate Actions:**
   * Triggers automated multi-step workflows across Microsoft 365, Azure, and external 3rd-party SaaS applications.
   * Best for executing multi-app business logic without manual intervention.
4. **Fabric Item Actions:**
   * Directly launches Fabric **Data Pipelines** or **Spark Notebooks**.
   * Best for triggering automated downstream data processing, ETL re-runs, or advanced machine learning scoring when anomalies occur.

---

## 5. Module Summary

* **Activator:** Real-time event detection and rules engine in Fabric that converts data-in-motion into automated actions.
* **Data Configuration Models:**
  * **Business Objects Model:** Maps event streams into **Objects** (instances with unique keys like `PackageId`) and **Properties** (attributes like `Temperature`).
  * **Direct Alerting:** Alerts created directly from Real-Time Dashboards, System Events (OneLake/Workspace), or KQL Querysets.
* **Rule Definitions (Definition Pane):**
  * **Monitor:** Property attribute + Summarization (`Average`, `Min`/`Max`, `Count`, `Total`) over **Window size** with recalculation **Step size**.
  * **Condition:** Detection logic (*Threshold*, *Change*, *Range*, *Missing Data*) + Occurrence behavior (*Every time* vs. *True for [duration]*).
  * **Property Filter:** Narrows rule scope combining up to 3 filter criteria.
* **Automated Action Types:**
  1. **Email:** Informational/Asynchronous context.
  2. **Teams:** Urgent real-time human alerts.
  3. **Power Automate:** Multi-application cross-platform workflows.
  4. **Fabric Items:** Auto-executes Fabric Data Pipelines or Spark Notebooks.
