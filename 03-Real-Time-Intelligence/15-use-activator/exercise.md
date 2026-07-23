# 🧪 Exercise 15: Use Activator in Microsoft Fabric

This hands-on lab covers using Fabric Activator to monitor streaming package delivery events, explore pre-built Activator sample objects and rules, create a new custom Business Object (`Redmond Packages`), apply multi-property filter criteria, and configure an automated email alert rule when medicine package temperatures exceed safety thresholds.

---

## 1. Create Activator & Load Sample Data

1. Navigate to your workspace in Fabric.
2. Select **+ New item** (or **Create**) -> Under *Real-Time Intelligence*, select **Activator**.
3. Rename your Activator item:
   * Click the dropdown next to the default name at the top-left -> Change to **`Contoso Shipping Activator`**.
4. On the Activator home page, select the **Try sample** tile.
   * *(This populates the Activator with sample streaming data and pre-built objects).*

---

## 2. Explore Sample Objects, Properties, & Rules

1. In the left **Explorer** pane, scroll down and select the **Package delivery events** stream.
2. Inspect the **Event details** live stream table (contains `PackageId`, `City`, `ColdChainType`, `SpecialCare`, `Temperature`, etc.).
3. In the Explorer pane, under the `Temperature` property of the `Package` object, select the rule **`Too hot for medicine`**.
4. Review the **Definition** pane:
   * **Monitor:** `Temperature` property.
   * **Condition:** `Temperature > 20°C`.
   * **Property Filter:** `Special care contents == "Medicine"`.
   * **Action:** Teams / Email notification.
5. Verify your recipient details in the Action section -> Click **Send me a test action** to verify notification delivery.

   * 📸 **Screenshot Checkpoint 1 (`15_sample_rule_test.png`):** Capture the Activator interface showing the `Too hot for medicine` rule definition and the successfully triggered test action button/output.

---

## 3. Create a Custom Business Object (`Redmond Packages`)

1. In the Explorer pane, select the **Package delivery events** stream.
2. On the ribbon, select **New object**.
3. In the **Build object** pane on the right, configure:
   * **Object name:** `Redmond Packages`
   * **Unique Identifier:** `PackageId`
   * **Properties:** Select `City`, `ColdChainType`, `SpecialCare`, `Temperature`
4. Click **Create**.
5. Observe the new `Redmond Packages` object added to the Explorer pane.

   * 📸 **Screenshot Checkpoint 2 (`15_build_object.png`):** Capture the Build object configuration pane specifying `Redmond Packages` with `PackageId` as unique identifier and properties selected.

---

## 4. Create & Configure Rule with Property Filters

1. Under `Redmond Packages` object in Explorer, select the **Temperature** property.
2. Click **New rule** on the ribbon.
3. Name the rule: Change default name to **`Medicine temp out of range`** (click pencil icon).
4. Configure rule parameters:
   * **Condition:** `Increases above`
   * **Value:** `20`
   * **Occurrence:** `Every time the condition is met`
5. Expand the **Property filter** section and configure **3 filters**:
   * **Filter 1:** `City` | `Is equal to` | `Redmond`
   * Click **Add filter** -> **Filter 2:** `SpecialCare` | `Is equal to` | `Medicine`
   * Click **Add filter** -> **Filter 3:** `ColdChainType` | `Is equal to` | `Refrigerated`

   * 📸 **Screenshot Checkpoint 3 (`15_rule_filters.png`):** Capture the Activator Definition pane showing the condition (`Increases above 20`) and the 3 property filters configured (`Redmond`, `Medicine`, `Refrigerated`).

---

## 5. Configure Automated Email Action & Start Rule

1. Scroll down to the **Action** section:
   * **Type:** `Email`
   * **To:** Your user account
   * **Subject:** `Redmond Medical Package outside acceptable temperature range`
   * **Headline:** `Temperature too high`
   * **Context:** Check the `Temperature` property checkbox.
2. Click **Save and start** on the toolbar.
3. Verify that the rule status updates to Active / Started.
4. *(Optional)* After observing alerts in your email or rule execution history, click **Stop** on the ribbon to turn off the rule.

   * 📸 **Screenshot Checkpoint 4 (`15_action_email_started.png`):** Capture the Action section showing the configured Email details and the active rule status toolbar.
