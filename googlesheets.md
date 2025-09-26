
---

# Converse2k25 Firebase Data Integration

This Google Apps Script project connects a Google Sheet with Firebase Realtime Database to fetch and analyze participant data for various events organized in Converse2k25. It provides summary statistics for event participation and departmental registrations, as well as detailed participant lists filtered by specific events.

---

## Features

* **Fetch Participant Data:** Retrieves participant records from Firebase Realtime Database.
* **Event Participation Summary:** Calculates and displays total participants per event.
* **Department and Year-wise Registration Counts:** Summarizes registrations by department and academic year.
* **Event-wise Participant Details:** Exports detailed participant information for a selected event to a dedicated sheet.
* **Custom Menu Integration:** Adds a menu to Google Sheets UI for easy access to the data loading functions.

---

## Firebase Configuration

The Firebase configuration object includes all necessary keys and URLs to access the Firebase Realtime Database.

```js
const firebaseConfig = {
  apiKey: "<YOUR_API_KEY>",
  authDomain: "<YOUR_AUTH_DOMAIN>",
  databaseURL: "<YOUR_DATABASE_URL>",
  projectId: "<YOUR_PROJECT_ID>",
  storageBucket: "<YOUR_STORAGE_BUCKET>",
  messagingSenderId: "<YOUR_SENDER_ID>",
  appId: "<YOUR_APP_ID>"
};
```

Make sure to replace the placeholder values with your actual Firebase project credentials.

---

## Functions Overview

### 1. `getFirebaseData()`

* Fetches participant data from the `/participants` node in Firebase.
* Returns parsed JSON data.

### 2. `writeToSheet()`

* Clears the active sheet.
* Fetches participant data and calculates total participants per event.
* Counts team members (2 participants per team).
* Writes a summary table showing each event and total participants.
* Also appends a row with the overall total participant count.

### 3. `countRegistrationsByDepartmentAndYear()`

* Clears the active sheet.
* Fetches participant data.
* Calculates counts of registrations grouped by department and academic year.
* Supports multi-participant (team) entries.
* Outputs a table with departments as rows and years as columns, plus total registrations.

### 4. Event-specific participant listing (`writeToSheet()` variant)

* Clears the active sheet.
* Filters participants based on a particular event name (e.g., "BUGBUZZ").
* Lists detailed participant information: Enrollment, Name, Department, Year, Email, Contact Number, College.
* Supports both single and team participants, listing each team member separately.

### 5. `onOpen()`

* Adds a custom menu titled **Firebase** to the Google Sheets UI.
* Provides menu options to load event participation data, department/year registration data, or event-specific participant details.

---

## Usage Instructions

1. **Set Up Firebase:**

   * Create a Firebase project with Realtime Database enabled.
   * Upload your participants' data under the `/participants` node.
   * Update the `firebaseConfig` object in the script with your Firebase project credentials.

2. **Deploy Google Apps Script:**

   * Open your Google Sheet.
   * Open the script editor (`Extensions > Apps Script`).
   * Paste the entire script code.
   * Save the script.

3. **Authorize Script:**

   * Run any function once (`writeToSheet` or `countRegistrationsByDepartmentAndYear`) to trigger the authorization prompt.
   * Grant the necessary permissions for the script to access external URLs and modify the spreadsheet.

4. **Use the Custom Menu:**

   * Reload the Google Sheet.
   * A new menu **Firebase** will appear.
   * Use the menu options to load event participation summaries or department/year registration data.
   * For event-specific sheets, use the relevant menu item (e.g., `Load "BUGBUZZ" Data`).

---

## Notes

* Team participation counts assume a maximum of two members per team.
* Department and year data rely on correct fields in participant records; unknown departments or years are grouped under "Other."
* This script uses `UrlFetchApp` which requires authorization to access external URLs.
* Make sure Firebase Realtime Database rules allow read access or use proper authentication if required.

---

## Troubleshooting

* **Empty Sheets:** Verify the Firebase URL and data structure. Make sure `/participants` contains participant data.
* **Authorization Errors:** Ensure script permissions are granted.
* **Incorrect Counts:** Verify participant records have consistent `eventName`, `fullName`, `department`, and `year` fields.

---

## License

This project is provided as-is under the MIT License. Feel free to modify and use it according to your needs.


---

### ✅ STEP 1: Final Google Apps Script Code

> 📋 Click the **"Copy Code"** button to easily copy and paste into your Apps Script Editor.

<div style="margin-bottom: 1em;"><strong>📜 Final Script:</strong></div>

```javascript
// ✅ Firebase configuration
const firebaseConfig = {
  apiKey: "<YOUR_API_KEY>",
  authDomain: "<YOUR_AUTH_DOMAIN>",
  databaseURL: "<YOUR_DATABASE_URL>",
  projectId: "<YOUR_PROJECT_ID>",
  storageBucket: "<YOUR_STORAGE_BUCKET>",
  messagingSenderId: "<YOUR_SENDER_ID>",
  appId: "<YOUR_APP_ID>"
};


// ✅ Fetch data from /participants
function getFirebaseData() {
  const url = `${firebaseConfig.databaseURL}/participants.json`;
  const response = UrlFetchApp.fetch(url);
  return JSON.parse(response.getContentText());
}

// ✅ Display total participants per event
function writeToSheet() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = getFirebaseData();
  sheet.clear();

  const eventNameMap = {
    aiMemes: "AI Memes",
    itaiQuiz: "IT & AI Quiz",
    bugBuzz: "BUG BUZZ",
    codathon: "CODATHON",
    cyberSpy: "Cyber Siege",
    logoHunt: "LOGO HUNT",
    pyIt: "PY-IT",
    promptHunt: "Prompt Hunt",
    webWave: "WEB WAVE"
  };

  const eventParticipantCount = {};
  let totalParticipants = 0;

  // Initialize counters
  for (const event in eventNameMap) {
    eventParticipantCount[event] = 0;
  }

  // Loop through each participant record
  for (const id in data) {
    const record = data[id];
    const event = record.eventName;

    if (eventParticipantCount.hasOwnProperty(event)) {
      const isTeam = record.fullName2 ? true : false;
      const count = isTeam ? 2 : 1;

      eventParticipantCount[event] += count;
      totalParticipants += count;
    }
  }

  // Output to sheet
  sheet.appendRow(["Event", "Total Participants"]);
  for (const event in eventParticipantCount) {
    const prettyName = eventNameMap[event] || event;
    sheet.appendRow([prettyName, eventParticipantCount[event]]);
  }

  sheet.appendRow(["---------------------", "---------------------"]);
  sheet.appendRow(["Total Participants", totalParticipants]);
}

// ✅ Display department & year registration counts
function countRegistrationsByDepartmentAndYear() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = getFirebaseData();
  sheet.clear();

  const headers = ["Department", "1st", "2nd", "3rd", "4th", "Total Registrations"];
  sheet.appendRow(headers);

  const departmentYearCount = {
    "AI&DS": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "Ch_En": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "Ci_En": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "CO": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "EE": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "ECE": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "IT": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "IC": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "ME": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "TT": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 },
    "Other": { "1st": 0, "2nd": 0, "3rd": 0, "4th": 0, "Total": 0 }
  };

  let totalCount = 0;

  for (const id in data) {
    const record = data[id];

    // Check both participants
    const participants = [];

    if (record.fullName1 || record.enrollment1) {
      participants.push({
        department: record.department1 || "Other",
        year: record.year1
      });
    }

    if (record.fullName2 || record.enrollment2) {
      participants.push({
        department: record.department2 || "Other",
        year: record.year2
      });
    }

    // If single participant
    if (participants.length === 0) {
      participants.push({
        department: record.department || "Other",
        year: record.year
      });
    }

    // Add to count
    for (const p of participants) {
      const dept = departmentYearCount.hasOwnProperty(p.department) ? p.department : "Other";
      const year = p.year === "1" ? "1st" :
                   p.year === "2" ? "2nd" :
                   p.year === "3" ? "3rd" :
                   p.year === "4" ? "4th" : "Other";

      if (departmentYearCount[dept].hasOwnProperty(year)) {
        departmentYearCount[dept][year]++;
        departmentYearCount[dept]["Total"]++;
        totalCount++;
      }
    }
  }

  // Output results
  for (const dept in departmentYearCount) {
    const row = [
      dept,
      departmentYearCount[dept]["1st"],
      departmentYearCount[dept]["2nd"],
      departmentYearCount[dept]["3rd"],
      departmentYearCount[dept]["4th"],
      departmentYearCount[dept]["Total"]
    ];
    sheet.appendRow(row);
  }

  sheet.appendRow(["", "", "", "", "", "----------------------------"]);
  sheet.appendRow(["", "", "", "", "", totalCount]);
}

// ✅ Add menu
function onOpen() {
  const ui = SpreadsheetApp.getUi();
  ui.createMenu("Firebase")
    .addItem("Load Event Participation", "writeToSheet")
    .addItem("Load Department and Year Data", "countRegistrationsByDepartmentAndYear")
    .addToUi();
}



```



---

### ✅ STEP 2: Final Google Apps Script Code for the Events wish sheet
> 📋 Click the **"Copy Code"** button to easily copy and paste into your Apps Script Editor.

<div style="margin-bottom: 1em;"><strong>📜 Final Script:</strong></div>

```javascript
const firebaseConfig = {
  apiKey: "<YOUR_API_KEY>",
  authDomain: "<YOUR_AUTH_DOMAIN>",
  databaseURL: "<YOUR_DATABASE_URL>",
  projectId: "<YOUR_PROJECT_ID>",
  storageBucket: "<YOUR_STORAGE_BUCKET>",
  messagingSenderId: "<YOUR_SENDER_ID>",
  appId: "<YOUR_APP_ID>"
};

// 🔁 Fetch data from Firebase /participants
function getFirebaseData() {
  const url = `${firebaseConfig.databaseURL}/participants.json`;
  const response = UrlFetchApp.fetch(url);
  return JSON.parse(response.getContentText());
}

// ✍️ Write all participants of "aiMemes" to the sheet
function writeToSheet() {
  const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  const data = getFirebaseData();
  sheet.clear();

  const headers = [
    "Sr. No", "Enrollment", "Name", "Department", "Year", "Email", 
    "Contact Number", "College", "Signature"
  ];
  sheet.appendRow(headers);

  let count = 1;

  for (const key in data) {
    if (!data.hasOwnProperty(key)) continue;

    const entry = data[key];
    const event = entry.eventName;

    // 🧠 Filter only aiMemes event
    if (event && event.toLowerCase() === "bugbuzz") {
      // 🧍 Single participant
      if (entry.fullName) {
        const row = [
          count++,
          entry.enrollment || "",
          entry.fullName || "",
          entry.department || "",
          entry.year || "",
          entry.email || "",
          entry.mobileNumber || "",
          entry.college || "",
          ""
        ];
        sheet.appendRow(row);
      }

      // 👥 Team participants
      else if (entry.fullName1 && entry.fullName2) {
        const row1 = [
          count++,
          entry.enrollment1 || "",
          entry.fullName1 || "",
          entry.department1 || "",
          entry.year1 || "",
          entry.email1 || "",
          entry.mobileNumber1 || "",
          entry.college1 || "",
          ""
        ];
        const row2 = [
          count++,
          entry.enrollment2 || "",
          entry.fullName2 || "",
          entry.department2 || "",
          entry.year2 || "",
          entry.email2 || "",
          entry.mobileNumber2 || "",
          entry.college2 || "",
          ""
        ];
        sheet.appendRow(row1);
        sheet.appendRow(row2);
      }
    }
  }
}

// 📋 Add custom menu
function onOpen() {
  const ui = SpreadsheetApp.getUi();
  ui.createMenu('Firebase')
    .addItem('Load "BUGBUZZ" Data', 'writeToSheet')
    .addToUi();
}
```