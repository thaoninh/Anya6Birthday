# Google Sheets Integration Setup

This guide will help you set up Google Sheets to collect RSVP form submissions instead of Netlify Forms.

## Step 1: Create a Google Sheet

1. Go to [Google Sheets](https://sheets.new) and create a new spreadsheet
2. Name it "Anya's Birthday RSVP"
3. Add these headers in row 1 (columns A-I):
   - A: Timestamp
   - B: Attendance
   - C: Guest Name
   - D: Child Name
   - E: Email
   - F: Phone
   - G: Adults
   - H: Kids
   - I: Message

## Step 2: Create Google Apps Script

1. In your Google Sheet, go to **Extensions → Apps Script**
2. Delete the default code and paste the code below:

```javascript
function doPost(e) {
  // Handle CORS preflight
  if (e.parameter.method === 'OPTIONS') {
    return ContentService.createTextOutput(JSON.stringify({'success': true}))
      .setMimeType(ContentService.MimeType.JSON)
      .setHeaders({
        'Access-Control-Allow-Origin': '*',
        'Access-Control-Allow-Methods': 'POST',
        'Access-Control-Allow-Headers': 'Content-Type'
      });
  }
  
  try {
    // Parse the JSON data
    const data = JSON.parse(e.postData.contents);
    
    // Get the active spreadsheet
    const sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
    
    // Append the data to the sheet
    sheet.appendRow([
      data.timestamp,
      data.attendance,
      data.guestName,
      data.childName,
      data.email,
      data.phone,
      data.adults,
      data.kids,
      data.message
    ]);
    
    // Return success response with CORS headers
    return ContentService.createTextOutput(JSON.stringify({
      'success': true,
      'message': 'RSVP saved successfully!'
    })).setMimeType(ContentService.MimeType.JSON)
      .setHeaders({
        'Access-Control-Allow-Origin': '*'
      });
      
  } catch (error) {
    // Return error response with CORS headers
    return ContentService.createTextOutput(JSON.stringify({
      'success': false,
      'error': error.toString()
    })).setMimeType(ContentService.MimeType.JSON)
      .setHeaders({
        'Access-Control-Allow-Origin': '*'
      });
  }
}

// Handle GET requests (for testing)
function doGet(e) {
  return ContentService.createTextOutput(JSON.stringify({
    'success': true,
    'message': 'RSVP form endpoint is working!'
  })).setMimeType(ContentService.MimeType.JSON)
    .setHeaders({
      'Access-Control-Allow-Origin': '*'
    });
}
```

3. Save the project (click the disk icon or Ctrl+S)
4. Name it "Anya Birthday RSVP Script"

## Step 3: Deploy as Web App

1. Click **Deploy → New deployment**
2. Click the gear icon (⚙️) and select **Web app**
3. Configure:
   - **Description**: RSVP Form Handler
   - **Execute as**: Me (your Google account)
   - **Who has access**: Anyone
4. Click **Deploy**
5. Authorize the script when prompted
6. Copy the **Web App URL** (looks like: `https://script.google.com/macros/s/XXXXXXXX/exec`)

## Step 4: Update HTML File

1. Open `index.html`
2. Find the line: `const GOOGLE_SCRIPT_URL = 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec';`
3. Replace `YOUR_SCRIPT_ID` with your actual script ID from the Web App URL

Example:
```javascript
const GOOGLE_SCRIPT_URL = 'https://script.google.com/macros/s/AKfycbzxxxxxxxx/exec';
```

## Step 5: Test

1. Open your birthday invitation page in a browser
2. Submit a test RSVP
3. Check your Google Sheet - the data should appear within a few seconds!

## Troubleshooting

**Form not submitting?**
- Check the browser console for errors (F12 → Console)
- Verify the Web App URL is correct
- Make sure the Google Sheet has the correct headers

**CORS errors?**
- Ensure you deployed with "Who has access: Anyone"
- The script includes CORS headers, but you may need to re-deploy

**Data not appearing?**
- Check if the sheet is the active sheet when the script runs
- Ensure column headers match the data being sent

## Security Notes

- The Google Apps Script is publicly accessible (required for form submissions)
- Only you can edit the script and view the spreadsheet
- Consider adding form validation in the Apps Script if needed
- The script logs all submissions with timestamps

## Benefits vs Netlify Forms

✅ **Free** - No usage limits
✅ **Real-time** - Data appears instantly
✅ **Customizable** - Easy to add formulas, formatting
✅ **Shareable** - Easy to share RSVP list with family
✅ **Notifications** - Can set up Google Sheets notifications for new entries
