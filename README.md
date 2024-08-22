# Google Ads Report Automation

This Google Ads Script automatically generates a report based on specified campaign performance metrics and writes the results to a Google Sheet. The report includes data on clicks, conversions, impressions, and other key metrics for a selected date range.

## Features

- **Automated Report Generation**: Fetches Google Ads campaign performance data for the previous day.
- **Customizable Queries**: Uses two separate queries to gather data on conversions and general campaign performance.
- **Data Aggregation**: Combines data from both queries and aggregates it by date and campaign name.
- **Google Sheets Integration**: Outputs the aggregated data into a specified Google Sheet.

## Setup Instructions

1. **Google Sheet Setup**: Create a Google Sheet where the report will be written. Note the Spreadsheet ID and sheet name.
2. **Script Deployment**:
   - Open Google Ads Script Editor.
   - Copy and paste the provided code into the editor.
   - Replace `spreadsheetId` with the ID of your Google Sheet.
3. **Running the Script**: Execute the `main()` function manually or set up a trigger to run it daily.

## Code

```javascript
function main() {
  var spreadsheetId = 'YOUR_SPREADSHEET_ID';
  var currentDate = new Date();
  var yesterday = new Date(currentDate.getTime() - 24 * 60 * 60 * 1000);
  var startDate = Utilities.formatDate(yesterday, 'GMT', 'yyyyMMdd');
  var endDate = startDate;

  var reportQuery1 = 'SELECT Date, CampaignName, ConversionTypeName ' +
    'FROM CAMPAIGN_PERFORMANCE_REPORT ' +
    'WHERE ConversionTypeName IN [\'Google Ads Conversion - Get Estimate\', \'Whatsapp Button Click\', \'Google Ads Conversion - Begin Booking\', \'Google Ads Conversion - Submit Booking (Thank you Page)\'] ' +
    'DURING ' + startDate + ',' + endDate;

  var reportQuery2 = 'SELECT Date, CampaignName, Clicks, Conversions, Impressions, Ctr, AverageCpc, Cost ' +
    'FROM CAMPAIGN_PERFORMANCE_REPORT ' +
    'DURING ' + startDate + ',' + endDate;

  var report1 = AdsApp.report(reportQuery1);
  var report2 = AdsApp.report(reportQuery2);

  var sheet = getSheet(spreadsheetId, 'Main');

  writeReportToSheet(report1, report2, sheet);
}

function getSheet(spreadsheetId, sheetName) {
  var spreadsheet = SpreadsheetApp.openById(spreadsheetId);
  var sheet = spreadsheet.getSheetByName(sheetName);
  if (!sheet) {
    sheet = spreadsheet.insertSheet(sheetName);
  }
  return sheet;
}

function writeReportToSheet(report1, report2, sheet) {
  var headerRow = [
    'Date',
    'Campaign Name',
    'Get Estimate',
    'Whatsapp Button Click',
    'Begin Booking',
    'Submit Booking',
    'Clicks',
    'Conversions',
    'Impressions',
    'CTR',
    'Avg CPC',
    'Cost'
  ];

  var counts = {};

  var rows1 = report1.rows();
  while (rows1.hasNext()) {
    var row = rows1.next();
    var date = row['Date'];
    var campaignName = row['CampaignName'];
    var conversionType = row['ConversionTypeName'];

    var key = date + '|' + campaignName;
    if (!counts[key]) {
      counts[key] = {
        'Date': date,
        'CampaignName': campaignName,
        'Google Ads Conversion - Get Estimate': 0,
        'Whatsapp Button Click': 0,
        'Google Ads Conversion - Begin Booking': 0,
        'Google Ads Conversion - Submit Booking (Thank you Page)': 0,
        'Clicks': 0,
        'Conversions': 0,
        'Impressions': 0,
        'Ctr': 0,
        'AverageCpc': 0,
        'Cost': 0
      };
    }
    counts[key][conversionType]++;
  }

  var rows2 = report2.rows();
  while (rows2.hasNext()) {
    var row = rows2.next();
    var date = row['Date'];
    var campaignName = row['CampaignName'];

    var key = date + '|' + campaignName;
    if (!counts[key]) {
      counts[key] = {
        'Date': date,
        'CampaignName': campaignName,
        'Google Ads Conversion - Get Estimate': 0,
        'Whatsapp Button Click': 0,
        'Google Ads Conversion - Begin Booking': 0,
        'Google Ads Conversion - Submit Booking (Thank you Page)': 0,
        'Clicks': parseInt(row['Clicks']),
        'Conversions': parseFloat(row['Conversions']),
        'Impressions': parseInt(row['Impressions']),
        'Ctr': parseFloat(row['Ctr']),
        'AverageCpc': parseFloat(row['AverageCpc']),
        'Cost': parseFloat(row['Cost'])
      };
    } else {
      counts[key]['Clicks'] += parseInt(row['Clicks']);
      counts[key]['Conversions'] += parseFloat(row['Conversions']);
      counts[key]['Impressions'] += parseInt(row['Impressions']);
      counts[key]['Ctr'] += parseFloat(row['Ctr']);
      counts[key]['AverageCpc'] += parseFloat(row['AverageCpc']);
      counts[key]['Cost'] += parseFloat(row['Cost']);
    }
  }

  var countsArray = Object.keys(counts).map(key => {
    return counts[key];
  });

  countsArray.sort((a, b) => {
    return a.Date.localeCompare(b.Date) || a.CampaignName.localeCompare(b.CampaignName);
  });

  sheet.clear();
  sheet.appendRow(headerRow);

  for (var i = 0; i < countsArray.length; i++) {
    var data = countsArray[i];
    var dataRow = [
      data['Date'],
      data['CampaignName'],
      data['Google Ads Conversion - Get Estimate'],
      data['Whatsapp Button Click'],
      data['Google Ads Conversion - Begin Booking'],
      data['Google Ads Conversion - Submit Booking (Thank you Page)'],
      data['Clicks'],
      data['Conversions'],
      data['Impressions'],
      data['Ctr'],
      data['AverageCpc'],
      data['Cost']
    ];
    sheet.appendRow(dataRow);
  }
}
