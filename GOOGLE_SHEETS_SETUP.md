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
   - I: Dietary
   - J: Message

## Step 2: Create Google Apps Script

1. In your Google Sheet, go to **Extensions → Apps Script**
2. Delete the default code and paste the code below:

```javascript
function doPost(e) {
  try {
    // Log what we received for debugging
    console.log('Received data:', e);
    console.log('Parameters:', e.parameter);
    
    // Get form data from URL-encoded parameters
    const params = e.parameter;
    
    // Get the active spreadsheet
    const spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
    const sheet = spreadsheet.getActiveSheet();
    
    // Add headers if first row
    if (sheet.getLastRow() === 0) {
      sheet.appendRow(['Timestamp', 'Attendance', 'Guest Name', 'Child Name', 'Email', 'Phone', 'Adults', 'Kids', 'Dietary', 'Message']);
    }
    
    // Append the data
    const rowData = [
      params.timestamp || new Date().toISOString(),
      params.attendance || '',
      params.guestName || '',
      params.childName || '',
      params.email || '',
      params.phone || '',
      params.adults || 0,
      params.kids || 0,
      params.dietary || '',
      params.message || ''
    ];
    
    console.log('Saving row:', rowData);
    sheet.appendRow(rowData);
    console.log('Row saved successfully');
    
    // Send email notification
    sendEmailNotification(params);
    
    // Return success
    var output = ContentService.createTextOutput(JSON.stringify({
      'success': true,
      'message': 'RSVP saved!'
    }));
    output.setMimeType(ContentService.MimeType.JSON);
    return output;
    
  } catch (error) {
    console.error('Error:', error);
    var output = ContentService.createTextOutput(JSON.stringify({
      'success': false,
      'error': error.toString()
    }));
    output.setMimeType(ContentService.MimeType.JSON);
    return output;
  }
}

function doGet(e) {
  var output = ContentService.createTextOutput(JSON.stringify({
    'success': true,
    'message': 'Working!'
  }));
  output.setMimeType(ContentService.MimeType.JSON);
  return output;
}

// Send email notification for new RSVP
function sendEmailNotification(params) {
  try {
    // YOUR EMAIL ADDRESS - Change this if needed
    const yourEmail = Session.getActiveUser().getEmail();
    
    if (!yourEmail) {
      console.log('Could not get user email - skipping notification');
      return;
    }
    
    const subject = params.attendance === 'Yes' 
      ? '🎉 New RSVP: ' + params.guestName + ' is coming!'
      : '💔 New RSVP: ' + params.guestName + ' cannot attend';
    
    let body = 'NEW RSVP SUBMITTED\n\n';
    body += 'Attendance: ' + (params.attendance || 'N/A') + '\n';
    body += 'Guest: ' + (params.guestName || 'N/A') + '\n';
    body += 'Child: ' + (params.childName || 'N/A') + '\n';
    body += 'Email: ' + (params.email || 'N/A') + '\n';
    body += 'Phone: ' + (params.phone || 'N/A') + '\n';
    body += 'Adults: ' + (params.adults || 0) + '\n';
    body += 'Kids: ' + (params.kids || 0) + '\n';
    body += 'Dietary: ' + (params.dietary || 'None') + '\n';
    body += 'Message: ' + (params.message || 'None') + '\n\n';
    body += 'Submitted at: ' + new Date().toLocaleString();
    
    // Send the email
    GmailApp.sendEmail(yourEmail, subject, body);
    console.log('Email notification sent to: ' + yourEmail);
    
  } catch (error) {
    console.error('Failed to send email notification:', error);
    // Don't fail the form submission if email fails
  }
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

## Email Notifications (Built-in)

**Good news:** Email notifications are already included in the script above! The `sendEmailNotification` function will automatically send you an email every time someone RSVPs.

### How It Works:
- The script automatically detects your Gmail address
- Sends an email with all RSVP details
- Works immediately - no additional setup needed

### To Enable:
1. The code above already includes email notifications (line 64 calls `sendEmailNotification`)
2. Just make sure to **create a new deployment** after any code changes
3. When you first deploy, Google will ask for permission to send emails - click **Allow**

### Email Format:
You'll receive emails like:
```
Subject: 🎉 New RSVP: John Smith is coming!

NEW RSVP SUBMITTED

Attendance: Yes
Guest: John Smith
Child: Emily Smith
Email: john@email.com
Phone: (555) 123-4567
Adults: 2
Kids: 1
Dietary: None
Message: Can't wait!

Submitted at: 4/9/2026, 11:30:00 AM
```

### Note About Google Sheets Notifications:
The built-in **Tools → Notification rules** feature in Google Sheets only works with Google Forms, not with Apps Script. That's why we added email notifications directly in the script code instead.

### Troubleshooting:
**Not receiving emails?**
- Check your spam/junk folder
- Make sure you authorized the script to send emails when deploying
- Check the Apps Script execution logs (View → Executions)
- The script sends to the account that owns the Google Sheet
- The script logs all submissions with timestamps

## Benefits vs Netlify Forms

✅ **Free** - No usage limits
✅ **Real-time** - Data appears instantly
✅ **Customizable** - Easy to add formulas, formatting
✅ **Shareable** - Easy to share RSVP list with family
✅ **Notifications** - Can set up Google Sheets notifications for new entries
