---
title: "Invoice Automation: An Example of a Larger Automation Utilizing BitSwan's auto_pipeline"
date: 2025-08-15T12:05:14+01:00
author: "patrik_kiseda"
description: "A practical example of building a larger automation with BitSwan’s auto_pipeline, using an invoice workflow that integrates a protected web form, CouchDB, Fakturoid(An invoicing service), Google Drive, and email — and reusable patterns you can apply to bigger apps."
tags: ["Tutorial", "Automation", "BitSwan", "BSPump", "Invoicing", "CouchDB", "Fakturoid"]
categories: ["Automations"]
draft: false
---

---

## Why we built this

If you rely on standard invoice tools, they’re fine for creating basic invoices, but quickly hit a wall when you need to do more. What if you want to store extra details for each client, send personalized emails, or automatically split bonuses among your team? Suddenly, you’re juggling spreadsheets, copy‑pasting data, or building awkward workarounds.

In a busy office, these repetitive tasks pile up fast. The more your business grows, the more time you spend on manual steps—time that could be better spent elsewhere. We needed a solution that could automate these everyday workflows, adapt to our needs, and connect with the tools we already use, all without starting from scratch every time. That’s why we built something flexible, auditable, and easy to extend—so we can focus on the work that matters, not the busywork.

At our company, we needed a simple, lightweight, and quickly deployable way to automate our invoicing and bonus workflows—without building a full custom app. This article walks through how we solved that need using BitSwan to build an automation that fits our needs.

All of it is orchestrated through BitSwan’s auto_pipeline, so the whole workflow is a single, readable pipeline.

---

## What we’re building

### High-level flow

1. User fills a web form 
2. Pipeline validates and normalizes data
3. Fakturoid API creates an official invoice (returns invoice number, variable symbol, invoice PDF)
4. Data is stored in CouchDB (including enriched metadata from Fakturoid)
5. PDF is downloaded and archived to Google Drive
6. Email notification is prepared and optionally sent to the client

---

## Why BitSwan auto_pipeline

BitSwan’s `auto_pipeline` lets you define the entire flow—web UI, processing, and output—in a single notebook or Python module. That brings:

- Easy onboarding: one file to read and run
- Composable sources/sinks: web form as a source, web response as a sink
- Explicit processing steps: each cell (or code block) becomes a sequential stage
- Environment-friendly: secrets loading, mock mode, and portable configs
- Rapid iteration: tweak fields, routes, or integrations without scaffolding a web app

A minimal pipeline entrypoint looks like this:

```python
auto_pipeline(
    source=lambda app, pipeline: WebFormSource(
        app, pipeline, route="/", fields=[
            TextField("user_name", display="User Name"),
            ...
    ),
    sink=lambda app, pipeline: WebSink(app, pipeline)
)
```

Where `WebFormSource` builds the form and registers any REST endpoints used by the page’s JavaScript.

![Form with company data](/images/invoiceautomation/form.png)  
_An example of a form based on WebFormSource used in our Automation._

## Web forms:  `WebFormSource` vs `ProtectedWebFormSource` and an example of custom `create_web_source`

- **WebFormSource**
  - Plain form source. It renders a page at a route and accepts POSTed form data.
  - No access control by default. Anyone who can reach the route can view and submit the form.
  - Good for local prototyping, NOT for production exposure.

- **ProtectedWebFormSource**
  - Same features as `WebFormSource`, but requests are gated by a shared secret.
  - Access paths supported by BitSwan:
    - Query param: `?secret=YOUR_SECRET`
    - Header: `Authorization: Bearer YOUR_SECRET`
  - Our AJAX endpoints also accept the secret via header `X-Form-Secret: ...` or JSON body field `{ "secret": "..." }`. If the page URL contains `?secret=...`, the Referer can be used to pass it along automatically.
  - Recommended for any environment beyond localhost.

- **create_web_source (custom helper)**
  - `create_web_source(app, pipeline)` wraps form setup for a cleaner `auto_pipeline` call.
    - Sets up `WebServerConnection`, registers AJAX endpoints, and returns the form source (`ProtectedWebFormSource` for production, `WebFormSource` for local use).
  - The form features:
    - Company search (AJAX, pre‑fills from CouchDB)
    - Dynamic bonus fields (multiple bonuses/recipients)
    - Live email preview (updates as you type, uses template placeholders)

![Company search dropdown](/images/invoiceautomation/searchbar.png)  
_Company search field with AJAX suggestions; choosing a company auto‑fills the form._

![AJAX results](/images/invoiceautomation/preFilledForm.png)  
_JSON returned by the search endpoint fills in the form from the last available data._

### Production warning

- Do not deploy `WebFormSource` directly on public networks. Use `ProtectedWebFormSource` (or `JWTWebFormSource` for expiring tokens and prefilled hidden fields) to protect the UI and backend endpoints.

## Secrets handling (AOC and local)

- BitSwan loads secrets from groups declared in `pipelines.conf` under `[secrets]`:
  - Example: `groups=fakturoid couchdb google-drive smtp webform`
  - Each group corresponds to a KEY=VALUE file in the secrets directory (e.g., `secrets/couchdb`, `secrets/webform`).

- **AOC/Container**
  - Place secret files under `/home/coder/workspace/secrets/`.
  - The runtime loads them into environment variables automatically on startup.

- **Local development**
  - Put files under `<pipelineFolder>/secrets/`.
  - For local development in notebooks, you can use `python-dotenv` to load environment variables from a secrets folder (typically a `secrets` directory next to your notebook).

- Verify secrets (optional)
  - Check presence of secrets via `os.environ.get('VAR')` in the notebook.
  - In our notebook we log whether critical variables like `FAKTUROID_CLIENT_ID`, `COUCHDB_PASSWORD`, and `WEBFORM_SECRET` are set.

When deploying, it is preferable to keep secrets in files or environment variables rather than hard‑coding, and avoid printing secret values in logs.

Minimal pipelines.conf:

```ini
[secrets]
groups = fakturoid couchdb google-drive smtp webform
```

---

### Example: Secrets for the protected web form

- The form secret in our automation is `WEBFORM_SECRET`.
- Create a file named `webform` in your secrets directory (e.g., `secrets/webform` or `/home/coder/workspace/secrets/webform`) and set:
  - `WEBFORM_SECRET=<your_secret>`

In our pipeline, `create_web_source` configures:

```python
protected_config = {"secret": WEBFORM_SECRET or ""}
web_source = ProtectedWebFormSource(app, pipeline, route="/", connection=web_connection, config=protected_config, ...)
```

This ensures both the form and the related AJAX routes are consistently gated.

---


## Bonus Management
Short version: add any number of bonuses, each with 1..N recipients, without changing backend schemas.

- **Why this works with a basic form**: We keep one hidden field `bonuses_json_data` that stores the entire bonus structure as JSON. You can freely tweak the UI (add/remove recipients or fields) and the form still submits a single value the pipeline parses. No migrations; no new form fields required.

- **How JavaScript is wired in**:
  - The custom `DynamicBonusField("bonuses")` renders buttons, a container, a hidden input, and includes `<script src="scripts/bonus_management.js"></script>`.
  - Button clicks call functions like `addBonus()`, `addRecipient()`, `removeRecipient()`; each change calls `updateBonusesJSON()` to keep the hidden JSON in sync.
  - The company search field follows the same pattern: it includes `<script src="scripts/company_search.js"></script>`, and the search input uses `onkeyup="searchCompanies(this.value)"` to call AJAX endpoints and then fill the form.

![Bonus Management Interface](/images/invoiceautomation/bonusManagement.png)  
_Dynamic bonus management UI showing a bonus with recipient, amount, and payment date._

- **What gets stored** (example):

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

- **Server-side**: On submit, the pipeline reads `form['bonuses_json_data']` and `json.loads(...)`. If the hidden JSON is missing, it can optionally fall back to scanning flat fields.

---

## CouchDB Integration

### Connection strategy (local vs container)

- In Docker/automation we run CouchDB as a sibling service on the same network and reach it at hostname `couchdb:5984`.
- In local dev we connect to `localhost:5984` (or an SSH tunnel) using the same credentials.
- Secrets are loaded via BitSwan secret groups (or `.env` files) and exposed as env vars:
  - `COUCHDB_PASSWORD`, `COUCHDB_USER`, and `COUCHDB_URL`.

### What we persist per document

- Raw form: `company_name`, `company_ID`, `VAT_id`, and other invoice data (e.g., company info, amounts, terms, emails).
- Enrichment from Fakturoid: `fakturoid_id` (unique identifier pairing our DB with Fakturoid), `invoice_number`, `variable_symbol` (payment reference), `fakturoid_status` (e.g., "sent", "paid"), `fakturoid_url` (public link).
- Bonuses: nested `bonuses` with recipients.
- `amount_paid` initialized to `0` for a follow‑up automation that checks paid status.
- Files/cloud: `pdf_path`, `pdf_filename`, and (after upload) `google_drive_file_id`, `google_drive_link`.
- Ops fields: `invoice_sent_date`, `save_status`, `error_message`, and a `timestamp` for audit.

### How we use it

- Company search and auto‑fill in the form (AJAX endpoints read recent docs).
- System of record for invoices and bonus structures (auditable history, idempotency, retries).
- Future automations (payment reminders, bonus payouts, dashboards, etc.).

![CouchDB document view](/images/invoiceautomation/CouchDBInvoice.png)  
_Sanitized CouchDB document showing invoice fields, Fakturoid enrichment, and nested bonuses._

---

## Fakturoid Integration

### What Fakturoid gives us

- Official invoice numbers (e.g., `25-000014`).
- The “variable symbol” (bank payment reference) used by customers in transfers.
- Polished, ready‑to‑send invoice PDFs and public invoice links.

The Fakturoid integration is a core part of our end‑to‑end invoice automation pipeline. When a new invoice is submitted, the pipeline invokes Fakturoid to create the invoice and return authoritative data. This happens after validation and before notifications or archiving.

We connect to Fakturoid using OAuth 2.0 Client Credentials, with secrets (`FAKTUROID_CLIENT_ID`, `FAKTUROID_CLIENT_SECRET`, `FAKTUROID_ACCOUNT_SLUG`) securely loaded from BitSwan’s secrets system.

**How it fits into the automation:**

- The pipeline passes all required customer and invoice details to the integration.
- The integration resolves or creates the customer (subject), ensuring idempotency.
- It creates the invoice and returns identifiers/metadata.
- We then download the PDF (`/invoices/{id}/download.pdf`) with retry handling for the 204 “not ready yet” case.

**Where it lives and how it’s called:**

- The logic is encapsulated in `FakturoidAPI`.
- The pipeline calls methods like `find_or_create_subject`, `create_invoice`, and `download_pdf`.
- A single `request()` method centralizes headers, auth, error handling, and JSON parsing.

**Summary of the `FakturoidAPI` class:**

- Manages authentication and builds the account‑scoped base URL.
- Exposes a minimal interface for:
  - `find_subject_by_name` / `find_or_create_subject`
  - `create_invoice`
  - `download_pdf`
- Ensures consistent, secure requests with clear error reporting.

---

## Google Drive Archiving

We archive the final invoice PDF to Google Drive using a service account. The pipeline saves a local copy (`invoices-pdf/…`), then uploads to a designated Drive folder and stores back `google_drive_file_id` and `google_drive_link` in CouchDB.

### Service account configuration

The pipeline loads the Google service account by serializing the entire JSON into an environment variable `GOOGLE_DRIVE_SERVICE_ACCOUNT_JSON` (the pipeline creates a temporary file for the SDK). We also read `GOOGLE_DRIVE_FOLDER_ID` and `GOOGLE_DRIVE_ACCOUNT_EMAIL` from the `google-drive` secret group.

```bash
# Provide the service account JSON via env var (redacted example)
export GOOGLE_DRIVE_SERVICE_ACCOUNT_JSON='{"type":"service_account","client_email":"svc@project.iam.gserviceaccount.com","private_key":"-----BEGIN PRIVATE KEY-----\\n...\\n-----END PRIVATE KEY-----\\n"}'
```

We use this solely as long‑term storage for invoice PDFs; the Drive link is convenient for sharing, and the file ID simplifies future automations.

---

## Email Notifications

This step turns an invoice into a deliverable message with a branded body and a PDF attachment. The pattern is reusable in other automations (orders, reports, statements).

- **Templates and placeholders**
  - Body and subject come from a template file (plain text) or from optional form fields for ad‑hoc tweaks.
  - We use simple placeholders like `[COMPANY_NAME]`, `[INVOICE_NUMBER]`, `[AMOUNT]`, `[CURRENCY]`, `[DUE_DATE]`, `[PAYMENT_REFERENCE]` and expand them from the current invoice data.
  - If no custom template is provided, a safe fallback is used.

- **SMTP configuration via secrets**
  - Credentials are read from a secret group (e.g., `secrets/smtp`) and exposed as environment variables: `SMTP_HOST`, `SMTP_PORT`, `SMTP_USER`, `SMTP_PASSWORD`.
  - In AOC/containers, secrets are mounted and auto‑loaded; locally, a `.env`‑style secrets file can be used.
  - If SMTP isn’t configured, the system switches to preview mode and prints the composed email to logs without sending.

- **Attachment handling**
  - After Fakturoid generates the invoice, we download the PDF to `invoices-pdf/…` (with retry logic for 204 "not ready").
  - The PDF is attached if present; sending still proceeds (with a warning) if it’s missing.

- **Preview and UX**
  - A live preview on the form mirrors the current subject/body as the user types and as placeholders resolve, so the final email is visible before sending.

![Email Preview Interface](/images/invoiceautomation/emailPreview.png)  
_Live email preview showing subject template, body template, and real-time placeholders getting filled in._

- **Send workflow**
  1. Load template (file or form), expand placeholders with invoice data.
  2. Configure SMTP from secrets (TLS).
  3. Compose MIME message and attach the PDF.
  4. Send to the customer’s email from the form.
  5. Update operational fields (e.g., `invoice_sent_date`) and persist to the database.

![Recieved E-mail](/images/invoiceautomation/receivedEmail.png)  
_Recieved e-mail with the invoice PDF in the attachments._

---

## Conclusion

This walkthrough shows how a single BitSwan pipeline can orchestrate a complete invoicing flow: a protected web form, CouchDB storage, Fakturoid invoice creation, Google Drive archiving, and email notifications—with extensibility points for search, dynamic bonuses, and previews. The same patterns apply to broader automations: replace the source, swap integrations, and keep the pipeline readable.

---


