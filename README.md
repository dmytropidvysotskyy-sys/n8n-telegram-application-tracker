# Telegram Bot for Logging Job Applications into Google Sheets

An n8n workflow that lets me log every sent application from Telegram in about 15 seconds instead of 2-3 minutes of manual spreadsheet work.

Personal tool built for my own workflow and used daily. Not a client deployment.

## Problem

A monetization course I am taking requires logging every freelance and job application in a Google Sheet: date, platform, type, requirements, cover letter text, and a screenshot of the response. Filling it by hand after each application was slow and easy to skip.

## Solution

I talk to a Telegram bot in three short steps. Each step is a reply to the previous bot message, so the bot always knows which row it is working on.

1. Send a short message with platform, type, optional date and requirements. The bot appends a new numbered row to the sheet.
2. Reply to the bot confirmation with the cover letter text. The bot writes it into the same row.
3. Reply with a screenshot (photo or file). The bot uploads it to Google Drive and writes the link into the same row.

## Message format

```
<platform> <type> [date] <requirements>
```

Platform codes: `up` (Upwork), `fh` (Freelancehunt), `li` (LinkedIn), `dou` (DOU). Any other value is used as typed.

Type codes: `п` (project application), `в` (vacancy application), `о` (personal offer).

Date is optional: `dd.MM` or `вчора`. If omitted, today's date in the Europe/Lisbon time zone is used.

Example:

```
up п 28.10 n8n workflow for lead qualification, Telegram and CRM
```

## How it works

1. Telegram Trigger receives a message. It only accepts messages from one allowed Telegram user ID.
2. A Switch node routes the message into one of three branches: new application, cover letter reply, screenshot reply.
3. New application branch: Code node parses the message, an IF node validates it, Google Sheets reads the column of row numbers, a Code node calculates the next number, then a new row is appended and the bot confirms.
4. Cover letter branch: Code node reads the row number from the replied bot message, Google Sheets finds the row, the cover letter is written into it, the bot confirms.
5. Screenshot branch: Code node reads the row number and the file ID, the file is downloaded from Telegram, uploaded to a Google Drive folder as `<row number>.<extension>`, and the Drive link is written into the row.
6. Every branch has validation and clear error replies in Telegram, for example when the message has no label or the row does not exist.

## Stack

n8n, Telegram Bot API, Google Sheets, Google Drive, JavaScript (n8n Code nodes)

## Result

About 15 seconds per entry instead of 2-3 minutes by hand, with screenshots stored in Drive and linked from the sheet.

## Setup

1. Import `workflow/application-tracker.json` into n8n.
2. Create a Telegram bot with BotFather and add its token as a Telegram credential.
3. Connect Google Sheets and Google Drive credentials.
4. In the Telegram Trigger node, replace `YOUR_TELEGRAM_USER_ID` with your own Telegram user ID.
5. Replace `YOUR_GOOGLE_SHEET_ID` in every Google Sheets node with your sheet ID.
6. Replace `YOUR_GOOGLE_DRIVE_FOLDER_ID` in the Google Drive node with your folder ID.
7. Make your sheet match the expected layout: header row on row 4, data from row 5, first column named `№`.
8. Activate the workflow.

## Notes

- Column names in the Google Sheets nodes are in Ukrainian because the sheet is in Ukrainian. Rename them to match your own sheet.
- All credential IDs, sheet IDs, folder IDs and user IDs in the exported JSON are placeholders.
