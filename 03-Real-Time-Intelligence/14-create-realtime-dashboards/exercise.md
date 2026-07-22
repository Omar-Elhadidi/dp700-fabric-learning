# 🧪 Exercise 14: Get Started with Real-Time Dashboards in Microsoft Fabric

This hands-on lab covers creating a Real-Time Dashboard connected to a KQL Eventhouse database, building interactive Stacked Bar Chart and Map tiles, optimizing with a Base Query, adding dynamic Multi-select Parameters, managing Multi-page layouts, and configuring Auto Refresh.

---

## 1. Prerequisites & Environment Setup

1. Create a workspace with Fabric capacity enabled.
2. Create an **Eventhouse** (e.g., `BicycleEventhouse`).
3. In your KQL database, select **Get data** -> **Eventstream** -> **New eventstream**:
   * **Name:** `Bicycle-data`
   * **Source:** Sample data -> `Bicycles`
   * **Destination:** Eventhouse KQL database -> Table `bikes` (JSON format) -> **Publish**.
4. Verify streaming data is flowing into the `bikes` table.

---

## 2. Create Real-Time Dashboard & Add Initial Tiles

1. Select **Create** from the left menu bar -> Under *Real-Time Intelligence*, select **Real-Time Dashboard**.
2. Name it `bikes-dashboard`.
3. In the toolbar, select **New data source** -> Choose **Eventhouse / KQL Database**:
   * **Display name:** `Bike Rental Data`
   * **Database:** Default database in your Eventhouse
   * **Passthrough identity:** Selected -> Click **Add**.
4. Select **Add tile** -> Ensure `Bike Rental Data` source is selected -> Enter KQL query:
   ```kql
   bikes
   | where ingestion_time() between (ago(30min) .. now())
   | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
   | project Neighbourhood, latest_observation, No_Bikes, No_Empty_Docks
   | order by Neighbourhood asc
   ```
5. Apply changes -> Edit tile (pencil icon) -> Visual formatting settings:
   * **Tile name:** `Bikes and Docks`
   * **Visual type:** `Bar chart`
   * **Visual format:** `Stacked bar chart`
   * **Y columns:** `No_Bikes`, `No_Empty_Docks`
   * **X column:** `Neighbourhood`
   * **Series columns:** `infer` | **Legend location:** `Bottom` -> Click **Apply changes**.
6. Select **New tile** -> Enter KQL query for map:
   ```kql
   bikes
   | where ingestion_time() between (ago(30min) .. now())
   | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
   | project Neighbourhood, latest_observation, Latitude, Longitude, No_Bikes
   | order by Neighbourhood asc
   ```
7. Edit tile -> Visual formatting settings:
   * **Tile name:** `Bike Locations`
   * **Visual type:** `Map`
   * **Define location by:** `Latitude and longitude` (`Latitude`, `Longitude`)
   * **Label column:** `Neighbourhood`
   * **Size:** Show (`Size column:` `No_Bikes`) -> Click **Apply changes**.

   * 📸 **Screenshot Checkpoint 1 (`14_dashboard_tiles.png`):** Capture the dashboard canvas showing both the `Bikes and Docks` stacked bar chart tile and the `Bike Locations` map tile side-by-side.

---

## 3. Consolidate Logic with a Base Query

1. On the dashboard toolbar, select **Base queries** -> Click **+ Add**.
2. Configure Base Query:
   * **Variable name:** `base_bike_data`
   * **Data source:** `Bike Rental Data`
   * **Query:**
     ```kql
     bikes
     | where ingestion_time() between (ago(30min) .. now())
     | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
     ```
3. Run query -> Click **Done** -> Close Base queries pane.
4. Refactor **Bikes and Docks** bar chart tile query to:
   ```kql
   base_bike_data
   | project Neighbourhood, latest_observation, No_Bikes, No_Empty_Docks
   | order by Neighbourhood asc
   ```
5. Refactor **Bike Locations** map tile query to:
   ```kql
   base_bike_data
   | project Neighbourhood, latest_observation, No_Bikes, Latitude, Longitude
   | order by Neighbourhood asc
   ```

   * 📸 **Screenshot Checkpoint 2 (`14_base_query.png`):** Capture the Base queries editor window displaying the `base_bike_data` variable definition and query execution.

---

## 4. Add Dynamic Multi-Select Parameter

1. On the toolbar, open **Manage** tab -> Select **Parameters**.
2. Delete any default parameters (e.g. default Time range).
3. Click **+ Add** and configure:
   * **Label:** `Neighbourhood`
   * **Parameter type:** `Multiple selection`
   * **Variable name:** `selected_neighbourhoods`
   * **Data type:** `string` | **Show on pages:** `Select all`
   * **Source:** `Query` (`Bike Rental Data`)
   * **Edit query:**
     ```kql
     bikes
     | distinct Neighbourhood
     | order by Neighbourhood asc
     ```
   * **Value column:** `Neighbourhood` | **Label column:** `Match value selection`
   * **Add "Select all" value:** Selected | **"Select all" sends empty string:** Selected
   * **Default value:** `Select all` -> Click **Done**.
4. Update `base_bike_data` Base Query to incorporate the parameter filter:
   ```kql
   bikes
   | where ingestion_time() between (ago(30min) .. now())
     and (isempty(['selected_neighbourhoods']) or Neighbourhood in (['selected_neighbourhoods']))
   | summarize latest_observation = arg_max(ingestion_time(), *) by Neighbourhood
   ```

   * 📸 **Screenshot Checkpoint 3 (`14_dashboard_parameter.png`):** Capture the dashboard with specific neighborhoods filtered via the `Neighbourhood` top parameter dropdown.

---

## 5. Add Pages & Configure Auto Refresh

1. Expand the **Pages** pane on the left side -> Click **+ Add page** -> Name it `Page 2`.
2. Select `Page 2` -> Click **+ Add tile** -> Enter query:
   ```kql
   base_bike_data
   | project Neighbourhood, latest_observation
   | order by latest_observation desc
   ```
3. Apply changes -> Resize tile to fill page height.
4. On toolbar, open **Manage** tab -> Select **Auto refresh**:
   * **Enabled:** Selected
   * **Minimum time interval:** `Allow all refresh intervals`
   * **Default refresh rate:** `30 minutes` -> Click **Apply**.
5. Save the dashboard.

   * 📸 **Screenshot Checkpoint 4 (`14_multipage_dashboard.png`):** Capture `Page 2` of the dashboard showing the left Pages navigation pane and the page 2 tile visual.
