# 🚀 AI Invoice Automation using n8n

A production-ready invoice automation workflow built with **n8n** that generates professional PDF invoices, stores records, emails clients, uploads invoices to Google Drive, and sends Slack notifications.

## ✨ Features

* 📥 Receive invoice requests via Webhook
* ✅ Validate incoming invoice data
* 🔢 Automatically generate invoice numbers
* 📅 Generate invoice and due dates
* 🧾 Automatically calculate GST (18%)
* 📄 Generate beautiful HTML invoices
* 📑 Convert HTML to PDF using PDFBolt
* ☁️ Upload invoices to Google Drive
* 📊 Save invoice records to Google Sheets
* 📧 Email PDF invoices to clients
* 💬 Send Slack notifications
* 🌐 Return JSON API response

---

## Workflow Overview

```text
Webhook
   │
   ▼
Validate Request
   │
   ▼
Generate Invoice Number
   │
   ▼
Generate Dates
   │
   ▼
Calculate GST
   │
   ▼
Generate HTML Invoice
   │
   ▼
Convert HTML → PDF
   │
   ▼
──────────────┬──────────────
              │
              ▼
Google Drive Upload
              │
              ▼
Google Sheets Record
              │
              ▼
Email Invoice
              │
              ▼
Slack Notification
              │
              ▼
Webhook Response
```

---

## Technologies Used

* n8n
* PDFBolt
* Google Drive API
* Google Sheets API
* Gmail
* Slack
* HTML & CSS
* Webhooks

---

## API Input Example

```json
{
  "invoice_number": "",
  "client_name": "John Doe",
  "client_email": "john@example.com",
  "service": "AI Automation Development",
  "amount": "25000"
}
```

---

## API Response Example

```json
{
  "success": true,
  "message": "Invoice generated successfully."
}
```

---

## What This Workflow Does

✔ Validates request

✔ Generates invoice number

✔ Calculates GST

✔ Generates professional invoice

✔ Creates PDF

✔ Uploads to Google Drive

✔ Stores metadata

✔ Emails customer

✔ Sends Slack notification

✔ Returns API response

---

## Setup

Configure the following credentials inside n8n before running:

* Gmail OAuth2
* Google Drive
* Google Sheets
* Slack
* PDFBolt

---


## Author

**Piyush Kumar**

Founder — AutoNode

AI Automations & AI Agents for Business

GitHub:
https://github.com/autonode-x

LinkedIn:
https://www.linkedin.com/in/piyush-ai

---

⭐ If you found this project useful, please consider starring the repository.
