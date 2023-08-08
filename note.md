<!-- google sheet script that working for "EnergyFuelConsumptionData" and also working for button and sending emails, moving data to master sheet -->

var sheetName = 'EnergyFuelConsumptionData';
var scriptProp = PropertiesService.getScriptProperties();
var recipientEmail = 'hafizzamanfarid@gmail.com'; // Specify the email address to which the notification should be sent

function intialSetupSampling() {
var activeSpreadsheet = SpreadsheetApp.getActiveSpreadsheet();
scriptProp.setProperty('key', activeSpreadsheet.getId());
}

function doPost(e) {
var lock = LockService.getScriptLock();
lock.tryLock(10000);

try {
var doc = SpreadsheetApp.openById(scriptProp.getProperty('key'));
var sheet = doc.getSheetByName(sheetName);

    var headers = sheet.getRange(1, 1, 1, sheet.getLastColumn()).getValues()[0];
    var nextRow = sheet.getLastRow() + 1;

    var newRow = headers.map(function(header) {
      return header === 'timestamp' ? new Date() : e.parameter[header];
    });

    sheet.getRange(nextRow, 1, 1, newRow.length).setValues([newRow]);






    MailApp.sendEmail(recipientEmail, subject, message);

    return ContentService
      .createTextOutput(JSON.stringify({ 'result': 'success', 'row': nextRow }))
      .setMimeType(ContentService.MimeType.JSON);

} catch (e) {
return ContentService
.createTextOutput(JSON.stringify({ 'result': 'error', 'error': e }))
.setMimeType(ContentService.MimeType.JSON);
} finally {
lock.releaseLock();
}
}

function sendEmailAndMoveRow() {
var ss = SpreadsheetApp.getActiveSpreadsheet();
var recipientAddress = ss.getSheetByName('recipientAddress');
var notificationMessage = ss.getSheetByName('notificationMessage');
var sourceSheet = ss.getSheetByName('EnergyFuelConsumptionData')
// && ss.getSheetByName('EnergyFuelConsumptionData');
var backupSheet = ss.getSheetByName('Backup');

var subject = notificationMessage.getRange(2, 1).getValue();
var message = notificationMessage.getRange(2, 2).getValue();
var n = recipientAddress.getLastRow();

for (var i = 2; i <= n; i++) {
var emailAddress = recipientAddress.getRange(i, 1).getValue();
MailApp.sendEmail(emailAddress, subject, message);
}

var lastRow = sourceSheet.getLastRow();
var lastRowValues = sourceSheet.getRange(lastRow, 1, 1, sourceSheet.getLastColumn()).getValues();
backupSheet.appendRow(lastRowValues[0]);
sourceSheet.deleteRow(lastRow);
}

//Send email notification
// var subject = 'New row added in Google Sheets (Sales Sampling)';
// var message = 'A new row has been added in the sheet: ' + sheetName + '\n\n';
// message += 'Row: ' + nextRow + '\n';
// message += 'Column values: ' + newRow.join(', ');

// send email when a button is clicked
// function sendEmail() {

// var ss = SpreadsheetApp.getActiveSpreadsheet()

// var recipientAddress=ss.getSheetByName('recipientAddress');

// var notificationMessage=ss.getSheetByName('notificationMessage');

// var subject = notificationMessage.getRange(2,1).getValue();;

// var message = notificationMessage.getRange(2,2).getValue();

// var n=recipientAddress.getLastRow();

// for (var i = 2; i < n+1 ; i++ ) {

// var emailAddress = recipientAddress.getRange(i,1).getValue();

// MailApp.sendEmail(emailAddress, subject, message);

// }

// }

<!-- end of script -->

<!-- other code -->
