---
title: "Automating Invoicing with BitSwan: Fakturoid, CouchDB, Bonuses, Email, and Google Drive"
date: 2025-08-11T09:00:00+01:00
author: "patrik_kiseda"
description: "An end-to-end invoicing automation that combines a dynamic web form, bonus distribution, CouchDB persistence, Fakturoid invoicing, Google Drive archiving, and email notifications — all orchestrated with BitSwan's auto_pipeline."
tags: ["Tutorial", "Automation", "BitSwan", "BSPump", "Invoicing", "CouchDB", "Fakturoid"]
categories: ["Bitswan Workspace"]
draft: true
---

---

## Why we built this

Traditional invoice tools are great at producing invoices, but they get limiting when you need custom workflows: storing rich local data, sending branded emails, restoring client data from history, or distributing bonuses across recipients. We needed something flexible, auditable, and easily extensible — without reinventing every integration.

This article introduces a BitSwan-based automation that:

- Creates a polished web form powered by BSPump
- Stores all submissions in CouchDB
- Generates official invoices via Fakturoid
- Distributes bonuses to multiple recipients per invoice
- Saves PDFs to Google Drive
- Sends customizable notification emails with attachments

All of it is orchestrated through BitSwan’s auto_pipeline, so the whole workflow is a single, readable pipeline.

---

## What we’re building

### High-level flow

1. User fills a web form (with company search and bonus management)
2. Pipeline validates and normalizes data
3. Fakturoid API creates an official invoice (returns number, variable symbol, links)
4. Data is stored in CouchDB (including enriched metadata)
5. PDF is downloaded and archived to Google Drive
6. Email notification is prepared and optionally sent to the client

---

## Why BitSwan auto_pipeline

BitSwan’s `auto_pipeline` lets you define the entire flow — web UI, processing, and output — in a single notebook or Python module. That brings:

- Easy onboarding: one file to read and run
- Composable sources/sinks: web form as a source, web response as a sink
- Explicit processing steps: each cell (or code block) becomes a sequential stage
- Environment-friendly: secrets loading, mock mode, and portable configs
- Rapid iteration: tweak fields, routes, or integrations without scaffolding a web app

A minimal pipeline entrypoint looks like this:

```python
auto_pipeline(
    source=create_web_source,
    sink=lambda app, pipeline: WebSink(app, pipeline)
)
```

Where `create_web_source` builds the form (including custom fields and preview) and registers any REST endpoints used by the page’s JavaScript.

---

## Web form and preview

- Company search field (AJAX) pre-fills known clients from CouchDB history
- Dynamic bonus field allows N bonuses with M recipients per bonus
- Live email preview panel renders subject/body from templates with placeholders like `[COMPANY_NAME]`, `[INVOICE_NUMBER]`, `[AMOUNT]`, etc.

Key idea: keep the preview DOM present but initially hidden; make it visible and keep it in sync as the user types.

---

## Bonus Management

The core goal is flexibility: unlimited bonuses, each with 1..N recipients, without complex backend forms or migrations. The UI isn’t meant to be pixel-perfect yet — the key is that adding, removing, and editing is fast and the data model is solid.

- **How it works (conceptually)**
  - **UI**: A bonus section where you can add bonuses and recipients dynamically.
  - **JavaScript**: Manages the DOM and keeps a single source of truth in a hidden field `bonuses_json_data`.
  - **Serialization**: The hidden field stores a JSON string with the full structure, submitted with the form.
  - **Pipeline**: On submit, the pipeline parses this JSON. If the hidden JSON is missing, it falls back to reading individual form fields (`bonus_..._recipient_...`).

- **Why this is powerful**
  - No schema migration for each UI tweak; add recipients or whole bonuses without backend changes.
  - The pipeline always receives a consistent nested object it can validate and persist.

### Data shape stored in the hidden field

```json
{
  "B1": {
    "recipients": [
      { "recipient_name": "Alice", "amount": 0.15, "payment_date": "2025-01-25" },
      { "recipient_name": "Bob",   "amount": 0.10, "payment_date": "2025-01-26" }
    ]
  },
  "B2": {
    "recipients": [
      { "recipient_name": "Carol", "amount": 0.05, "payment_date": "2025-01-27" }
    ]
  }
}
```

### The custom field and its script

- We render a `DynamicBonusField("bonuses")` in the form. It:
  - Inserts a container where bonuses/recipients are composed.
  - Adds a hidden input `bonuses_json_data`.
  - Loads `scripts/bonus_management.js` with functions like `addBonus()`, `addRecipient()`, `removeBonus()`, `removeRecipient()`, and `updateBonusesJSON()`.
- The script updates the hidden field on every change, so the server gets the latest structure without manual wiring.

### Server-side parsing (simplified)

```python
# Prefer nested JSON if provided, else parse legacy flat fields
bonuses = {}
bonuses_json = form_data.get('bonuses_json_data', '{}')
if bonuses_json and bonuses_json != '{}':
    bonuses = json.loads(bonuses_json)
else:
    # Fallback: scan names like bonus_1_name, bonus_1_recipient_name_2, ...
    # Build bonuses[bonus_name] = { "recipients": [...] }
    pass
```

### Developer ergonomics

- UI changes are front-end only; the hidden JSON contract stays the same.
- Multiple recipients or new fields can be added without touching the pipeline logic.
- The pipeline validates and persists the nested data to CouchDB alongside the invoice, so it’s queryable and auditable.

---

## CouchDB Integration

Outline database connection strategy (local vs container), idempotent DB creation, and what we persist (raw form, enrichment, status fields, timestamps).

To be completed.

---

## Fakturoid Integration

Summarize auth with client credentials, subject resolution/creation, and invoice creation. Note: we attach `subject_id` to the invoice and read back `number`, `variable_symbol`, status, and public links.

To be completed.

---

## Google Drive Archiving

Explain retry logic for PDF availability, saving to disk, then uploading to Drive with service account credentials and storing file/link metadata for later use.

To be completed.

---

## Email Notifications

Discuss template loading (HTML file), placeholder substitution, attachment of PDF, and SMTP configuration via secrets. Include live in-form preview for the user before sending.

To be completed.

---

## Running locally

```bash
bitswan-notebook examples/automation-server-1-site/InvoicePipeline/main.ipynb
```

Open the form at `http://localhost:8080`, fill it, and watch the pipeline logs for each stage (Fakturoid creation, CouchDB saving, PDF download, Drive upload, email preparation).

---

## Conclusion

To be completed.

---

## Next steps

- Add screenshots and short clips of the form, preview, and end-to-end run
- Publish a sample dataset and sanitized secrets template for quick-start
- Package a docker-compose for local CouchDB and an example SMTP relay


