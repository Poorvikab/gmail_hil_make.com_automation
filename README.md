# 📬 Human-in-the-Loop Gmail Automation
### Gmail × Gemini AI × Google Sheets — Built on Make.com

An automation that reads incoming emails, drafts AI-powered replies using Google Gemini, and lets **you** approve or reject each reply before it's ever sent. No emails go out without your sign-off.

---
## Demo Video


## 🧠 How It Works

```
SCENARIO 1: Capture & Draft
Gmail (Watch Emails) → Gemini AI (Draft Reply) → Google Sheets (Log Row)

SCENARIO 2: Approve & Send (runs independently)
Google Sheets (Search Rows) → Router → [YES] Gmail (Send) + Sheets (Approved)
                                      → [NO]  Sheets (Rejected)
```

You manually write **Yes** or **No** in the Google Sheet's Approval column. Scenario 2 picks that up and acts accordingly.

---

## ✨ Features

- 📥 Auto-monitors Gmail inbox for unread emails
- 🤖 Drafts personalized replies using **Gemini 2.5 Flash**
- 📊 Logs every email + AI draft to Google Sheets
- ✅ Human approval gate — nothing sends without your go-ahead
- 🔄 Two decoupled scenarios for clean separation of concerns
- 📝 Edit the AI draft directly in the sheet before approving

---

## 🛠️ Tech Stack

| Tool | Purpose |
|---|---|
| [Make.com](https://make.com) | Automation platform |
| Gmail | Trigger + send replies |
| Google Gemini AI (2.5 Flash) | Draft generation |
| Google Sheets | Logging + approval interface |

---

## 📋 Prerequisites

- A Make.com account (free tier works)
- Gmail account connected to Make
- Google Gemini AI account connected to Make
- Google Sheets access via Make

---

## 🚀 Setup Guide

### Step 1 — Create the Google Sheet

Create a new Google Sheet with these exact column headers:

| A | B | C | D | E | F |
|---|---|---|---|---|---|
| Subject | Sender Mail | Thread ID | Gmail Draft | Approval | Status |

> ⚠️ Leave **Approval** and **Status** columns blank — these are filled manually/automatically.

---

### Scenario 1: Gmail → Gemini → Sheets

**Step 2 — Gmail: Watch Emails**
- Module: `Gmail → Watch Emails`
- Criteria: `Only unread messages`
- Mark as read when fetched: `Yes`
- Trigger: `From now on`

**Step 3 — Gemini AI: Generate Response**
- Module: `Google Gemini AI → Generate a response`
- Model: `Gemini 2.5 Flash`
- Role: `User`
- Message:
```
Here's a lead inquiry:
{{1.fullTextBody}}

Here's the lead's name:
{{1.fromName}}

Write a polite and engaging reply considering the leads inquiry and leads name.

Some context:
My name is:
[YOUR_NAME]
```
- System Instruction: `You are a professional email assistant. Craft concise, personalized replies for the queries.`

**Step 4 — Google Sheets: Add a Row**
- Module: `Google Sheets → Add a Row`
- Map fields:
  - Subject (A) → `{{1.subject}}`
  - Sender Mail (B) → `{{1.fromEmail}}`
  - Thread ID (C) → `{{1.threadId}}`
  - Gmail Draft (D) → `{{2.result}}`

**Step 5 — Run & Test**
1. Send yourself a test email
2. Click **Run Once**
3. Check your Google Sheet — a new row should appear
4. Manually write `Yes` or `No` in the **Approval** column

---

### Scenario 2: Sheets → Router → Send/Reject

**Step 6 — Create a NEW Scenario**
> Temporarily connect to Scenario 1's last block; you'll unlink later.

**Step 7 — Google Sheets: Search Rows**
- Module: `Google Sheets → Search Rows`
- Same spreadsheet & sheet
- Filter (OR condition):
  - `Approval = Yes`
  - `Approval = No`

**Step 8 — Add Router**
- Creates two routes: one for Yes, one for No

**Step 9 — Route 1 Filter (YES)**
- Label: `Approved`
- Condition: `Approval (E) → Text Equal to → Yes`

**Step 10 — Gmail: Reply to Email** *(YES route)*
- Module: `Gmail → Reply to an Email`
- Thread ID → from Google Sheet
- Body → Gmail Draft column (edit the draft in the sheet if needed before approving)

**Step 11 — Google Sheets: Update Row** *(YES route)*
- Row number → from Search Rows
- Status → `Approved`

**Step 12 — Route 2 Filter (NO)**
- Label: `Rejected`
- Condition: `Approval (E) → Text Equal to → No`

**Step 13 — Google Sheets: Update Row** *(NO route)*
- Row number → from Search Rows
- Status → `Rejected`
- *(No email is sent)*

**Step 14 — Unlink Scenarios**
- Right-click the connection between Scenario 1 and 2 → **Unlink**
- Scenario 2 now runs independently

**Step 15 — Move Timer Trigger**
- Move the schedule trigger to Scenario 2's **Search Rows** module
- Click **Run Once** to process all pending approvals

---

## 📁 Files in This Repo

```
├── README.md
└── blueprint_clean.json        # Make.com blueprint (sanitized — no credentials)
```

### Importing the Blueprint

1. Download `blueprint_clean.json`
2. In Make.com → Create new scenario → **Import Blueprint**
3. Re-connect your own accounts (Gmail, Gemini, Google Sheets)
4. Update the spreadsheet ID to your own sheet
5. Replace `YOUR_NAME` in the Gemini prompt

---

## ⚙️ Configuration Reference

After importing the blueprint, replace these placeholders:

| Placeholder | What to put |
|---|---|
| `YOUR_CONNECTION_ID` | Your Make.com connection ID (auto-assigned on reconnect) |
| `YOUR_EMAIL@gmail.com` | Your Gmail address |
| `YOUR_SPREADSHEET_ID` | Your Google Sheet ID (from the URL) |
| `YOUR_NAME` | Your name for the Gemini prompt |
| `YOUR_MAKE_ZONE` | Your Make.com region (e.g. `eu1.make.com`) |

---

## 📌 Notes

- You can **edit the AI-generated draft** directly in the Google Sheet before writing `Yes`
- Scenario 2 only processes rows where Approval is `Yes` or `No` — blank rows are skipped
- Run Scenario 2 manually or set a schedule (e.g. every 15 minutes)
- One router can have only one fallback route — configure carefully

---

## 📄 License

MIT — free to use, fork, and adapt.
