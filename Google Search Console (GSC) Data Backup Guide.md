# Google Search Console (GSC) Data Backup Guide

## Stage 1: Create the Destination

First, we need a Google Sheet to store the data and an Apps Script environment to run our code.

1. Open your browser and visit [sheets.new](https://sheets.new) to create a new Google Sheet.
2. Give your spreadsheet a name, such as `GSC Data Backup`.
3. In the top menu, go to **Extensions** > **Apps Script**.
4. This will open a new code editor window—this is your "control center" for the backup script.

## Stage 2: Obtain Google Cloud Credentials

Since we are pulling data directly from Google’s backend, we need to use a Google Cloud Platform (GCP) project as a bridge.

**1. Create/Find a GCP Project**

- Open the [Google Cloud Console](https://console.cloud.google.com/) in a new tab.
- If you don't have a project, click the project dropdown in the top-left corner and select **New Project**. Name it something like `GSC-Backup-Tool`.
- Once created, stay on the project dashboard.

**2. Copy the Project Number**

- Look for the **Project Info** card on the dashboard.
- Find the **Project Number**.
- ⚠️ **Note**: You need the **all-numeric "Number,"** NOT the "Project ID" (which contains letters).
- **Copy this number.**

**3. Enable the Search Console API**

- In the left-hand menu, go to **APIs & Services** > **Library**.
- Search for: `Google Search Console API`.
- Click on it and hit the blue **Enable** button.
- *(If you skip this, the script will fail with an "API not enabled" error.)*

## Stage 3: Link the Script to the Cloud

Return to your **Apps Script editor** window.

**1. Bind the Project**

- Click the **Project Settings** (gear icon ⚙️) in the left sidebar.
- Scroll down to the **Google Cloud Platform (GCP) Project** section.
- Click **Change project**.
- Paste your **Project Number (numeric)** and click "Set project."
- **OAuth Configuration**: If a red warning appears asking for authorization:
  1. Click **Configure Consent Screen**.
  2. Set User Type to **External**.
  3. Fill in the app name and your support email.
  4. In the **Test Users** section, click **Add Users** and enter your own Google email address. (This is crucial for the script to run).

**2. Modify the Manifest (appsscript.json)**

- Still in **Project Settings**, check the box: **“Show 'appsscript.json' manifest file in editor”**.
- Switch back to the **Editor** (brackets icon `< >`).
- Select `appsscript.json` from the file list.
- **Completely replace** its contents with the code below and click **Save**:

JSON

```
{
  "timeZone": "Asia/Shanghai",
  "dependencies": {
  },
  "exceptionLogging": "STACKDRIVER",
  "oauthScopes": [
    "https://www.googleapis.com/auth/script.external_request",
    "https://www.googleapis.com/auth/spreadsheets", 
    "https://www.googleapis.com/auth/webmasters.readonly"
  ],
  "runtimeVersion": "V8"
}
```

## Stage 4: Deploy the Core Code

1. Select `Code.gs` in the left sidebar.
2. **Clear** the default `function myFunction() {...}`.
3. Paste the following code:

JavaScript

```
/**
 * GSC Data Saver - v0.1 (Internal MVP)
 * Fetches data via REST API to bypass standard Apps Script service limitations.
 */

function getGSCDataViaRest() {
  // ================= CONFIGURATION (Edit Here) =================
  
  // Your GSC Property URL.
  // For Domain Properties: use 'sc-domain:example.com'
  // For URL Prefix Properties: use 'https://example.com/'
  var siteUrl = 'sc-domain:yourdomain.com'; 
  
  // =============================================================

  var sheetName = 'GSC_Backup_Rest';
  var date = new Date();
  date.setDate(date.getDate() - 3); // Fetch data from 3 days ago to account for GSC lag
  var dateString = Utilities.formatDate(date, Session.getScriptTimeZone(), 'yyyy-MM-dd');
  
  Logger.log('Fetching data for date: ' + dateString);

  var apiUrl = 'https://www.googleapis.com/webmasters/v3/sites/' + encodeURIComponent(siteUrl) + '/searchAnalytics/query';
  
  var payload = {
    "startDate": dateString,
    "endDate": dateString,
    "dimensions": ["date", "query", "page", "device"], 
    "rowLimit": 25000
  };

  try {
    var token = ScriptApp.getOAuthToken();
    var options = {
      "method": "post",
      "contentType": "application/json",
      "headers": {"Authorization": "Bearer " + token},
      "payload": JSON.stringify(payload),
      "muteHttpExceptions": true
    };

    var response = UrlFetchApp.fetch(apiUrl, options);
    var json = JSON.parse(response.getContentText());

    if (response.getResponseCode() !== 200) {
      Logger.log('API Error: ' + JSON.stringify(json));
      return;
    }

    if (json.rows && json.rows.length > 0) {
      saveToSheet(json.rows, sheetName);
      Logger.log('Successfully saved ' + json.rows.length + ' rows.');
    } else {
      Logger.log('No data found for this date.');
    }
  } catch (e) {
    Logger.log('Execution Error: ' + e.toString());
  }
}

function saveToSheet(rows, sheetName) {
  var ss = SpreadsheetApp.getActiveSpreadsheet();
  var sheet = ss.getSheetByName(sheetName);
  if (!sheet) {
    sheet = ss.insertSheet(sheetName);
    sheet.appendRow(['Date', 'Query', 'Page', 'Device', 'Clicks', 'Impressions', 'CTR', 'Position']);
    sheet.setFrozenRows(1);
  }
  var output = rows.map(function(row) {
    return [row.keys[0], row.keys[1], row.keys[2], row.keys[3], row.clicks, row.impressions, row.ctr, row.position];
  });
  sheet.getRange(sheet.getLastRow() + 1, 1, output.length, output[0].length).setValues(output);
}
```

1. **Important**: Update line 12 (`siteUrl`) with your actual GSC property URL.
2. Click the **Save** (diskette 💾) icon.

## Stage 5: First Run & Automation

**1. Initial Authorization**

- In the toolbar, ensure `getGSCDataViaRest` is selected in the function dropdown.
- Click **Run**.
- An "Authorization required" popup will appear. Click **Review Permissions**.
- Select your Google account.
- **The "Safety" Warning**: You will likely see "Google hasn't verified this app." This is normal since you just wrote it.
  - Click **Advanced**.
  - Click **Go to [Project Name] (unsafe)** at the bottom.
  - Check all permission boxes and click **Allow**.

**2. Verify the Results**

- Check your Google Sheet. You should see a new tab named `GSC_Backup_Rest` populated with your data.

**3. Set Up Automation**

- In the Apps Script editor, click the **Triggers** (alarm clock icon ⏰) in the left menu.
- Click **+ Add Trigger** in the bottom right.
- Configure as follows:
  - **Function to run**: `getGSCDataViaRest`
  - **Event source**: Time-driven
  - **Type of time-based trigger**: Day timer
  - **Time of day**: 1am to 2am (Recommended)
- Click **Save**.

**🎉 Congratulations!** Your Google Sheet will now automatically back up your GSC data every night. Even years from now, you’ll have a complete record of every search term that led users to your site.

------

### Skip the Manual Setup?

Setting up scripts can be tedious. I am currently developing the **GSC Data Saver Browser Extension**:

- ✅ **Zero Config**: No APIs or code required. Install and go.
- ✅ **Multi-site Management**: Automatically back up all properties under your account.
- ✅ **Visual Dashboards**: Built-in Looker Studio templates for instant reporting.

[👉 Get GSC Saver on Chrome Web Store](https://chromewebstore.google.com/detail/jafnmmnidbecklolnjlkakhppjmoolfn?utm_source=item-share-cb)

------

**About the Author** This guide was created by **Mice**, an independent developer focused on solving real-world problems with code. If you run into errors or have ideas for SEO data analysis, feel free to reach out:

- **Twitter/X**: @Mice242697
- **Email**: maxinhao1997@gmail.com