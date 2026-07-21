# 🧪 Exercise 12: Ingest Real-Time Data with Eventstream in Microsoft Fabric

This hands-on lab covers ingesting streaming bicycle rental data using Microsoft Fabric Eventstream, routing raw data directly to an Eventhouse KQL database, applying a 5-second tumbling window `Group By` transformation *in-stream*, and querying both raw and transformed data using KQL.

---

## 1. Setup Eventhouse & KQL Database

1. Navigate to your workspace in Fabric.
2. Select **+ New item** -> Choose **Eventhouse**. Name it (e.g., `BicycleEventhouse`).
3. Note the automatically created default KQL database with the same name.

---

## 2. Create Eventstream & Add Sample Data Source

1. In the KQL database main page, select **Get data** -> **Eventstream** -> **New eventstream**.
2. Name the Eventstream: `Bicycle-data`.
3. In the Eventstream canvas, select **Use sample data**.
4. Configure the source:
   * **Source name:** `Bicycles`
   * **Sample data:** Select **Bicycles** sample data.

---

## 3. Configure Direct Eventhouse Destination (Raw Data)

1. Select **Transform events or add destination** -> Choose **Eventhouse**.
2. Configure settings:
   * **Data ingestion mode:** `Event processing before ingestion`
   * **Destination name:** `bikes-table`
   * **Workspace:** Your workspace
   * **Eventhouse:** `BicycleEventhouse`
   * **KQL database:** Your database
   * **KQL Destination table:** Select **Create new** -> Name it `bikes` -> Click **Done**.
   * **Input data format:** `JSON`
3. Click **Save** -> Click **Publish** on the toolbar.
4. Wait ~1 minute -> Select `bikes-table` node on canvas -> Verify rows in **Data preview** pane.

   * 📸 **Screenshot Checkpoint 1 (`12_raw_bikes_destination.png`):** Capture the Eventstream canvas showing `Bicycles` source -> `Bicycle-data-stream` -> `bikes-table` Eventhouse destination node with live data preview.

---

## 4. Query Raw Streaming Data

1. Navigate to your KQL Database -> Under *Tables*, select `bikes`.
2. Run standard 24-hour ingestion query:
   ```kql
   // See the most recent data - records ingested in the last 24 hours.
   bikes
   | where ingestion_time() between (now(-1d) .. now())
   ```
3. Verify that raw bicycle records (`Street`, `No_Bikes`, etc.) are actively landing in the database.

   * 📸 **Screenshot Checkpoint 2 (`12_raw_kql_query.png`):** Capture the KQL query editor executing the `bikes` table ingestion time query and displaying raw stream rows.

---

## 5. Add In-Stream Tumbling Window Transformation (`Group By`)

1. Open `Bicycle-data` Eventstream -> Click **Edit** on the toolbar.
2. Under **Transform events**, select **Group by**.
3. Connect the output of `Bicycle-data` node to the input of `Group by` node.
4. Edit `Group by` node settings:
   * **Operation name:** `GroupByStreet`
   * **Aggregate type:** `Sum`
   * **Field:** `No_Bikes` -> Click **Add** (Creates function `SUM of No_Bikes`)
   * **Group aggregations by:** `Street`
   * **Time window:** `Tumbling`
   * **Duration:** `5 seconds`
   * **Offset:** `0 seconds`
5. Connect `GroupByStreet` node to a new **Eventhouse** destination:
   * **Data ingestion mode:** `Event processing before ingestion`
   * **Destination name:** `bikes-by-street-table`
   * **Destination table:** Create new table named `bikes-by-street`
   * **Input data format:** `JSON`
6. Click **Save** -> Click **Publish**.
7. Select `bikes-by-street-table` node -> Inspect **Data preview** pane (observe `Street`, `SUM_no_Bikes`, and `Window_End_Time` columns).

   * 📸 **Screenshot Checkpoint 3 (`12_transformed_eventstream.png`):** Capture the full Eventstream canvas showing the `GroupByStreet` transformation node connected to `bikes-by-street-table` destination node.

---

## 6. Query Transformed Streaming Data

1. Navigate to your KQL Database -> Refresh tables -> Select `bikes-by-street`.
2. Execute KQL query aggregating 5-second window bike counts:
   ```kql
   ['bikes-by-street']
   | summarize TotalBikes = sum(tolong(SUM_No_Bikes)) by Window_End_Time, Street
   | sort by Window_End_Time desc, Street asc
   ```
3. Observe live grouped results breaking down total bike availability per street for each 5-second tumbling window.

   * 📸 **Screenshot Checkpoint 4 (`12_transformed_kql_query.png`):** Capture the KQL query editor executing the `bikes-by-street` 5-second window summary query with output results.
