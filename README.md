# 🍽️ WhatsApp Restaurant Reservation Agent (n8n)

A fully automated, AI-powered restaurant reservation system built with n8n. Customers interact via WhatsApp — the AI handles the entire conversation, manages bookings, and syncs everything to Google Calendar and Google Sheets. No human needed.

---

## 🎥 Demo

> Customer sends a WhatsApp message (or voice note) → AI understands the request → checks availability → confirms the booking → logs to CRM → syncs Google Calendar automatically.

---

## ✨ Features

- 💬 **Conversational AI** — Handles natural language booking requests via WhatsApp
- 🎙️ **Voice message support** — Customers can send voice notes; Whisper transcribes them and a dedicated data node routes the text correctly to the AI agent
- 📅 **Full reservation management** — Create, update, and cancel bookings
- 🗓️ **Automated Google Calendar sync** — Dedicated scheduled flows create/update/delete calendar events independently from the AI agent
- 📊 **Google Sheets as CRM** — All reservations logged and managed in a structured sheet
- 📧 **Optional email confirmation** — Gmail confirmation sent to customer after booking (optional, toggle on/off)
- 🧠 **Conversation memory** — AI remembers context within the same session
- 📚 **Knowledge base** — AI can answer questions about the restaurant (hours, menu, policies)

---

## 🛠️ Tech Stack

| Service | Purpose |
|---|---|
| **n8n** (self-hosted) | Workflow automation engine |
| **WhatsApp Business API** | Customer messaging trigger & responses |
| **OpenAI GPT-4.1** | AI agent for conversation handling |
| **OpenAI Whisper** | Voice message transcription |
| **Google Sheets** | Reservation database / CRM |
| **Google Calendar** | Automated calendar event management |
| **Gmail** | Optional booking confirmation emails |
| **Calendarific API** | Holiday & festival date awareness |

---

## 🏗️ Workflow Architecture

The workflow is split into 3 logical sections:

### 1. Main Conversation Flow
- WhatsApp Trigger → detects text or voice message
- Voice branch: downloads media → transcribes via Whisper → extracts text via dedicated "1b- Voice Data" node → feeds to agent
- AI Agent with tools: Get Tables, Get Reservations, Create / Update / Cancel Reservations, Calculator, Knowledge Base, Date Context
- Sends reply back via WhatsApp

### 2. Calendar Sync — Create Events (Scheduled)
- Runs on a schedule → fetches new unsynced reservations from Sheets → creates Google Calendar events → marks as synced

### 3. Calendar Sync — Delete/Modify Events (Scheduled)
- Runs on a schedule → checks for cancelled or modified reservations → deletes or updates the corresponding calendar event

---

## 🚀 Setup Instructions

### Prerequisites
- n8n instance (self-hosted recommended — [Hetzner VPS](https://hetzner.com) works great)
- WhatsApp Business API account (Meta Developer Portal)
- OpenAI API key
- Google Cloud project with Sheets + Calendar + Gmail APIs enabled
- Calendarific API key (free tier available)

### Step 1 — Import the workflow
1. Download `Restaurant_Reservation_sanitized.json`
2. In n8n, go to **Workflows → Import from File**
3. Select the JSON file

### Step 2 — Set up credentials
You'll need to create the following credentials in n8n:

| Credential | Used by |
|---|---|
| WhatsApp OAuth account | WhatsApp Trigger, Send message, Download media |
| OpenAI API | ChatGPT-4.1, Transcribe a recording |
| Google Sheets OAuth2 | All Google Sheets nodes |
| Google Calendar OAuth2 | All Calendar nodes |
| Gmail OAuth2 | Send a message (email confirmation) |

### Step 3 — Configure Google Sheets
Create a Google Sheet with the following columns in Sheet 1:

```
Reservation ID | Name | Phone | Date | Time Slot | Table | Party Size | Status | Mail | Calendar Synced
```

Copy your Sheet ID from the URL and replace `YOUR_GOOGLE_SHEET_ID` in all relevant nodes.

### Step 4 — Configure your Knowledge Base
Replace `YOUR_KNOWLEDGE_BASE_URL` in the `Knowledge_base` node with a URL pointing to a plain text file containing your restaurant info (name, address, hours, menu highlights, policies, etc).

A GitHub Gist works perfectly for this — create a public gist, click **Raw**, and use that URL.

### Step 5 — Configure WhatsApp
1. Set your WhatsApp Phone Number ID in the **Send message** node
2. Point your WhatsApp webhook to your n8n instance URL

### Step 6 — Configure Calendarific (Optional)
Replace `YOUR_CALENDARIFIC_API_KEY` in the `Learn_Dates_Ogren` node URL with your API key. Update the `country` parameter to your country code (e.g. `TR` for Turkey, `PL` for Poland).

### Step 7 — Activate
1. Activate the main workflow
2. Activate both scheduled Calendar sync workflows
3. Test by sending a WhatsApp message to your number

---

## 📋 Google Sheets Structure

The workbook has **3 sheets**:

---

### Sheet 1: `Table Info`
Defines the restaurant's tables. The AI agent reads this to check availability and match customer requests.

| Column | Description |
|---|---|
| `Table ID` | Unique table identifier (e.g. T1, T2) |
| `Capacity (People)` | Supported party size range (e.g. `1-4`, `6-12`) |
| `Smoking` | `Yes` / `No` |
| `Available` | `Yes` / `No` |
| `Location` | `Indoor` / `Terrace` |
| `Near Window` | `Yes` / `No` / `Optional` |
| `High Chair` | `Yes` / `No` |
| `Notes` | e.g. `Can be merged or rearranged` |

---

### Sheet 2: `Table Reservation`
The main CRM. Every booking is logged here. The AI agent reads and writes to this sheet.

| Column | Description |
|---|---|
| `Reservation ID` | Unique ID generated by AI (e.g. `RES-001`) |
| `Date` | Booking date (YYYY-MM-DD) |
| `Time Slot` | e.g. `18:00-20:00` |
| `Table` | Table ID (e.g. `T1`) |
| `Party size` | Number of guests |
| `Name` | Customer name |
| `Phone` | WhatsApp number |
| `Mail` | Customer email (for confirmation emails) |
| `Request` | Special requests (e.g. `near window`, `smoking available`) |
| `Status` | `Confirmed` / `Modified` / `Cancelled` / `Cancellation Confirmed` |
| `Source` | `Whatsapp` / `Chat` |
| `Booked At` | Timestamp of when the booking was made |
| `Calendar_ID` | Google Calendar event ID (populated by the calendar sync flow) |

---

### Sheet 3: `Table Availability`
Used for availability lookups. Columns: `Slot ID`, `Date`, `Time_Slot`, `Table_ID`, `Party_Size`, `Status`, `Reservation ID`

> This sheet can be populated manually or extended to support time-slot-based availability checking.

---

## 🤝 Contributing

Pull requests welcome! If you improve the workflow, please sanitize credentials before sharing.

---

## 📬 Built by

**Efe Güngör** — Automation & Integration Specialist  
[LinkedIn](https://www.linkedin.com/in/efegungor1/) | [GitHub](https://github.com/VodkaChocolate)

If this helped you, a ⭐ on the repo is appreciated!
