# Data

google sheet 


function doGet() {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheets()[0] || ss.insertSheet();
  const data = sheet.getDataRange().getValues();
  return ContentService.createTextOutput(JSON.stringify(data)).setMimeType(ContentService.MimeType.JSON);
}

function doPost(e) {
  const ss = SpreadsheetApp.getActiveSpreadsheet();
  const sheet = ss.getSheets()[0];
  const payload = JSON.parse(e.postData.contents);

  if (payload.action === "UPLOAD_FILE") {
    sheet.appendRow(["---", "LEDGER START:", payload.ledgerId, "---", "---", "", "", ""]);
    sheet.getRange(sheet.getLastRow(), 1, 1, 8).setBackground("#cbd5e1").setFontWeight("bold");

    const rows = payload.rows.map(r => [
      payload.uploadDate, 
      new Date().toLocaleTimeString(), 
      payload.fileName,
      r.join(" | "), 
      "", 
      "ACTIVE", 
      payload.fileId, 
      payload.ledgerId
    ]);
    sheet.getRange(sheet.getLastRow() + 1, 1, rows.length, 8).setValues(rows);
  }

  const data = sheet.getDataRange().getValues();
  if (payload.action === "CHANGE_STATUS") {
    for (let i = 1; i < data.length; i++) {
      if (data[i][6] === payload.fileId) sheet.getRange(i + 1, 6).setValue(payload.status);
    }
  }
  if (payload.action === "PERMANENT_DELETE") {
    for (let i = data.length - 1; i >= 1; i--) {
      if (data[i][6] === payload.fileId) sheet.deleteRow(i + 1);
    }
  }
  if (payload.action === "UPDATE_COMMENT") {
    for (let i = 1; i < data.length; i++) {
      if (data[i][3] === payload.rowData) sheet.getRange(i + 1, 5).setValue(payload.comment);
    }
  }
  return ContentService.createTextOutput("Success");
}
