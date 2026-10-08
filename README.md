# Salesforce Integration Guide

Filter and route customer service tickets using IndieML via Salesforce Flow's native HTTP Callouts.

## Prerequisite: Request API Key
Before starting, you need an API key to authenticate the connection.
Click [here](https://indieml.app/#api-key) to get your free IndieML API key. Copy the key and keep it handy for the configuration step.

---

## Option 1: The Quick Install (Unmanaged Package)
You can deploy the complete Flow, External Service, and JSON schemas directly into your Salesforce environment using our unmanaged package.

### 1. Install the Package
* Click [here](https://login.salesforce.com/packaging/installPackage.apexp?p0=04tg8000000QIu9&isdtp=p1) for the package link.
* Log into your Salesforce environment.
* Select **Install for Admins Only** and click **Install**.

### 2. Update the API Key
* Go to the **Setup** page.
* Navigate to **Quick Find** and search for **Named Credentials**.
* Click on the **IndieML** credential.
* Scroll down to the **Principals** section, click the arrow below **Actions**, and select **Edit**.
* **Parameter 1 > Value:** *Paste your API key here.*
* Click **Save**.

Your integration is now active. You can go to **Setup** > **Flows** to view and customize the IndieML Flow Blueprint.

---

## Option 2: Build from Scratch
If you prefer to build the architecture yourself or want to understand exactly how the integration routes data, follow this step-by-step blueprint. You can set up the entire pipeline using only Flow Builder (no Apex code required).

### Phase 1: The Secure Vault (Named Credentials)

**1. Create the External Credential**
* Go to the **Setup** page.
* Navigate to **Quick Find** and search for **Named Credentials**.
* Click the **External Credentials** tab and click **New**.
* **Label / Name:** `IndieML`
* **Authentication Protocol:** `Custom`
* Click **Save**.

**2. Set the Principal**
* From the `IndieML` page, scroll to **Principals** and click **New**.
* **Parameter Name:** `ApiKey`
* **Sequence Number:** `1`
* Under **Authentication Parameters**, click **Add**.
    * **Name:** `ApiKey`
    * **Value:** *Paste your API key here.*
* Click **Save**.

**3. Set the Custom Header**
* From the `IndieML` page, scroll to **Custom Headers** and click **New**.
* **Name:** `X-API-Key`
* **Value:** `{!$Credential.IndieML.ApiKey}`
* **Sequence Number:** `1`
* Click **Save**.

**4. Create the Named Credential**
* Go back to the **Named Credentials** page, click the **Named Credentials** tab, and click **New**.
* **Label:** `IndieML Endpoint`
* **Name:** `IndieMLEndpoint`
* **URL:** `https://indieml.app/v1/score/substance`
* **Enabled for Callouts:** Checked (ON)
* **Authentication:**
    * **External Credential:** `IndieML`
* **Callout Options:**
    * **Generate Authorization Header:** Unchecked (OFF)
    * **Allow Formulas in HTTP Header:** Checked (ON)
    * **Allow Formulas in HTTP Body:** Unchecked (OFF)
* Click **Save**.

### Phase 2: Admin Permissions
* Navigate to **Quick Find** and search for **Profiles**.
* Select **System Administrator**.
* Click **Enabled External Credential Principal Access** and click **Edit**.
* Select `IndieML-ApiKey` from the Available list and click **Add** to transfer it to the Enabled list.
* Click **Save**.

### Phase 3: The "Hello World" Sandbox Flow

**1. Create the Callout Action**
* Navigate to **Quick Find**, search for **Flows**, and click **New Flow**.
* Select **Autolaunched Automations**.
* Select **Autolaunched Flow (No Trigger)**.
* On the blank canvas, click the **+** icon and select **Action**.
* At the bottom of the right panel, click **Create HTTP Callout**.
* **Name the Callout:** `GetSubstanceScore`
* **Named Credential:** Select `IndieML Endpoint` from the dropdown, then click **Next**.
* **Configure Invocable Action:**
    * **Label:** `Score Text`
    * **Method:** `POST`
    * Click **Next**.

**2. Define the JSON Schemas**
* Under **Sample JSON Request**, paste:

```json
{
  "input_text": "This is a test run."
}

* Click **Review**, then click **Next**.
* Under **Select Sample Response Method**, select **Use Example Response** and click **Next**.
* Under **Sample JSON Response**, paste:

```json
{
  "result": {
    "text": "This is a test run.",
    "substance_score": 0.18
  }
}
```

* Click **Review**, then click **Save**.

### 3. Configure the Flow Variables
You will now be back on the Main Flow Builder screen.
* **Label:** `Call IndieML API`
* **API Name:** (Let this auto-populate to `Call_Indie_ML_API`)
* Under **Set Request Body** > **Value**, click and select **New Resource**.
* **API Name:** `requestPayload` (Data Type and Apex Class will auto-populate). Click **Done**.
* Click the **+** icon on the canvas *below* **Autolaunched Flow**.
* Select **Logic > Assignment**.
* **Label:** `Set Text To Score`
* **API Name:** (Let this auto-populate to `Set_Text_To_Score`)
* **Variable:** Select your new `requestPayload` resource.
    * The menu will reset. Click `requestPayload` a second time to display the `input_text` option.
    * Select `input_text` from the newly expanded list.
    * **Verify:** The box should now show `requestPayload > input_text`.
* **Operator:** `Equals`
* **Value:** Click **New Resource** at the very bottom of the dropdown list.
    * **Resource Type:** `Constant`
    * **API Name:** `textToScore`
    * **Data Type:** `Text`
    * **Value:** `The quick red fox jumps over the lazy dog.`
* Click **Done**.

### Phase 4: Run the Debugger
* Click **Save** in the top right corner.
* **Flow Label:** `IndieML Substance Flow`
* **Flow API Name:** (Let this auto-populate to `IndieML_Substance_Flow`)
* Click **Save**.
* Click **Debug**, then click **Run** at the bottom of the left side popout.
* Look at the **Debug Details** in the left side popout.
* Click **Details** under **GetSubstanceScore.Score Text (External Services): Call IndieML API**.
* Under `2XX`, you should see a substance score of `0.19`.

---

## Addendum 1: Adapting the flow to a Live Pipeline
* Create a **Record-Triggered Flow** instead of an Autolaunched Flow (e.g., triggered when a `Case` is created).
* In your Assignment block, map the `requestPayload > input_text` variable directly to `{!$Record.Description}` to ingest live customer tickets.
* Add a **Decision** block after the HTTP Callout to evaluate the score (e.g., `Outputs from Call_Indie_ML_API > 2XX > result > substance_score` is **Less Than** `0.30`).
* Use standard Salesforce actions to auto-tag, reprioritize, or deflect the case based on the branch outcome.

---

## Addendum 2: Modifying the Package for any External API
Because this unmanaged package deploys native, standard Salesforce metadata, you can use it as a foundational boilerplate to integrate any third-party REST API into Flow without writing Apex.

### 1. Install the Package
* Click [here](https://login.salesforce.com/packaging/installPackage.apexp?p0=04tg8000000QIu9&isdtp=p1) for the package link.
* Log into your Salesforce environment.
* Select **Install for Admins Only** and click **Install**.

### 2. Update the API URL
* Go to the **Setup** page.
* Navigate to **Quick Find** and search for **Named Credentials**.
* Click the **Named Credentials** tab.
* Click the dropdown arrow next to **IndieML Endpoint** and select **Edit**.
* Enter the new API base URL and click **Save**.

### 3. Update the API Key
* Stay in **Named Credentials**, but click the **External Credentials** tab.
* Click on the **IndieML** credential.
* Scroll down to the **Principals** section, click the arrow below **Actions**, and select **Edit**.
* Under **Parameter 1**, paste your new API key into the **Value** field.
* Click **Save**.

### 4. Update the Request & Response Schemas
* Navigate to **Quick Find** and search for **External Services**.
* Under **External Service Name** find the generated service, click the arrow below **Actions**, and select **Edit**.
* Replace the sample JSON request and response payloads with the schema expected by your new API. Salesforce will automatically regenerate the input and output variables.
* Click **Save**.

### 5. Remap the Flow Action
* Open the routing flow in **Flow Builder**.
* Double-click the **HTTP Callout** action element on the canvas.
* Map your Salesforce record values to the newly generated request fields.
* Update any downstream **Decision** elements to reference the new output variables returned by your API.
