# Student Intake → Google Sheets → Welcome Email (n8n)

Collect student details via an n8n Form, append them to a Google Sheet, and send a welcome email with Gmail.  
This repo includes a redacted workflow JSON that’s safe to share (IDs are placeholders).

---

## Overview

**Flow**
1. **Form Trigger** (`n8n-nodes-base.formTrigger`) — collects **Full Name**, **Email**, **City**  
2. **Google Sheets** (`append`) — appends submission to a sheet  
3. **Gmail** — sends a simple “Thanks for enrolling!” message

**Why this workflow?**
- Zero-code intake form
- Instant logging to Sheets for ops/analytics
- Immediate confirmation email to students

---

## Prerequisites

- An **n8n** instance (self-hosted or n8n Cloud)
- A Google account with:
  - Access to a target **Google Sheet**
- A Gmail account (or Google Workspace) to send emails
- Network access to your n8n webhook URL (if you’ll share the form publicly)

---

## Files

- `workflow.json` — the workflow with **placeholders** instead of real IDs

> Replace placeholders inside n8n after importing:
- `YOUR_GOOGLE_SHEET_ID`
- `YOUR_SHEET_TAB_NAME_OR_gid=XXXXXX`
- `YOUR_GOOGLE_SHEETS_CREDENTIAL_REF/NAME`
- `YOUR_GMAIL_CREDENTIAL_REF/NAME`
- `REGENERATE_ME_WEBHOOK_ID*`
- `YOUR_WORKFLOW_ID`, `YOUR_INSTANCE_ID` (auto-managed by n8n; not required to change)

---

## Setup (5–10 min)

1. **Create the Google Sheet**
   - Name a tab (e.g., `Students`) with headers in Row 1:
     ```
     Full Name | Email | City
     ```
   - Get the Sheet ID from the URL:
     `https://docs.google.com/spreadsheets/d/<YOUR_GOOGLE_SHEET_ID>/edit`

2. **Import the Workflow**
   - In n8n → **Import from file** → select `workflow.json`.

3. **Connect Credentials**
   - **Google Sheets OAuth2**: create/select your credential and grant access to the sheet.
   - **Gmail OAuth2**: create/select your credential for sending emails.

4. **Fill Placeholders**
   - In **Google Sheets node**:
     - `documentId.value` → `YOUR_GOOGLE_SHEET_ID`
     - `sheetName.value` → `Students` **or** `gid=0`
   - In **Gmail node** and **Form Trigger**:
     - Leave `webhookId` blank or **click “Regenerate”** in n8n.

5. **Activate & Test**
   - **Activate** the workflow.
   - Open the **Form Trigger** URL (shown in the node) → submit a test.
   - Confirm:
     - New row appears in Google Sheet
     - Test email arrives in inbox

---

## Security & Hardening

- **Do not publish** real webhook URLs/IDs; use placeholders (as in this repo).
- Restrict Sheet sharing (avoid “anyone with the link”).
- Add **CAPTCHA** or rate-limiting (reverse proxy or API gateway) if the form is public.
- Validate inputs (length, email format). Minimal validation is already in node config; add more if needed.
- Rotate/regenerate webhook IDs if a link may have leaked.

---

## Customization

- **Fields**: Add/rename fields in Form Trigger → update:
  - Sheet headers (must match keys)
  - `columns.value` mappings in Google Sheets node
  - Email template (subject/body)
- **Email Template**: You can switch Gmail node to **HTML** and use a nicer template.
- **Spam Control**: Add a small **Function** node before Sheets to drop duplicates or throttle.

---

## Data Mapping (current)

| Form Field  | JSON Key       | Sheet Column | Email Usage                         |
|-------------|----------------|--------------|-------------------------------------|
| Full Name   | `Full Name`    | Full Name    | Subject: `Hello {{ Full Name }}`    |
| Email       | `Email`        | Email        | Recipient: `$json.Email`            |
| City        | `City`         | City         | (not used in email by default)      |