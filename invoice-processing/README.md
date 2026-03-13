# AI-Driven Invoice Processing & Accounting Automation

A fully autonomous accounts payable system for SMEs.
From a messy inbox to a booked invoice in Moneybird — with AI-powered data
extraction, automatic duplicate detection, and a human-in-the-loop approval
flow via Slack.

---

## The Problem

Most SMEs still process purchase invoices largely by hand. Invoices arrive
as PDFs via email, staff open them one by one, and data gets manually typed
into accounting software. At 1,000 invoices per month, that adds up to
40–80 hours of administrative work — error-prone, unscalable, and expensive.

---

## What It Does

The system consists of two separate workflows that communicate via Airtable
as a shared data layer.

**Workflow A — The Intake Engine (Batch Process)**
Runs on a schedule (hourly, daily or weekly)

1. Fetches unread emails with attachments from Gmail (filtered on PDF/image)
2. Loops through invoices one by one (respects API rate limits)
3. Backs up the original file to Google Drive
4. Extracts invoice data using GPT-4o Vision (amount, VAT, vendor, date, etc.)
5. Normalizes and cleans the data via JavaScript (ISO dates, float amounts)
6. Checks Airtable for existing vendor — creates one if not found
7. Checks Airtable for duplicate invoice (vendor + invoice number combination)
8. Routes based on amount:
   - Under €100 → automatic processing (Fast Lane)
   - Over €100 → human approval required via Slack
9. Books approved invoices directly into Moneybird via API
10. Marks emails as processed and archives them

**Workflow B — The Button Listener (Event Driven)**
Runs only on human interaction

1. Listens for a POST request from the Slack interface
2. Decodes which user clicked and which invoice ID is involved
3. On Approve: retrieves clean data from Airtable, formats it for the
   Moneybird API, calculates ex-VAT amounts, and books the purchase invoice
4. On Reject: updates status in Airtable and sends feedback to Slack
5. Returns 200 OK to prevent Slack timeout errors

---

## Tech Stack

| Tool | Role |
|---|---|
| n8n | Workflow orchestration |
| Gmail | Invoice ingestion & email management |
| GPT-4o Vision | AI-powered invoice data extraction |
| Google Drive | Original file backup |
| Airtable | Shared data layer, vendor DB, audit trail |
| Slack (Block Kit) | Human-in-the-loop approval interface |
| Moneybird API | Final invoice booking in accounting software |
| JavaScript (code nodes) | Data normalization, Slack Block Kit builder, API formatter |

---

## Architecture Decisions

**Why two separate workflows instead of one?**

Workflow A is batch-driven and runs on a schedule. Workflow B is event-driven
and only fires on human interaction. Combining them in a single workflow would
create timing conflicts and make the system harder to debug and maintain.
Airtable acts as the shared data layer between both workflows — clean,
auditable, and decoupled.

**Why duplicate detection on vendor + invoice number instead of filename?**

Filenames are unreliable — vendors often reuse generic names like
"factuur.pdf". The duplicate check combines vendor identity and invoice number,
which is how duplicates actually occur in practice (e.g. a forwarded email
or a vendor sending the same invoice twice).

**Why GPT-4o Vision instead of standard OCR?**

GPT-4o Vision reads invoices as a human would. It ignores watermarks like
"KOPIE", corrects OCR errors in scanned documents, and handles
non-standard invoice layouts without requiring template configuration.

**Why Slack Block Kit for approvals?**

Block Kit renders as a native app experience inside Slack. Buttons disappear
after clicking to prevent double approvals. The approver never needs to leave
Slack or log into an external portal.

**Why batching (one-by-one loop) instead of parallel processing?**

Processing invoices sequentially respects the API rate limits of both
Moneybird and Airtable. Parallel processing would be faster but risks
hitting limits and causing partial failures that are difficult to recover from.

---

## Flow Overview
```
Gmail (unread invoices)
        ↓
Loop: process one by one
        ↓
Google Drive backup
        ↓
GPT-4o Vision — extract data
        ↓
JavaScript — normalize & clean
        ↓
Airtable — find or create vendor
        ↓
Airtable — duplicate check
    ↓ duplicate?
  Alert & stop
    ↓ unique?
Amount < €100?
  ↓ Yes                    ↓ No
Auto-book              Slack approval request
into Moneybird         (Block Kit)
                            ↓
                    Workflow B (Button Listener)
                            ↓
                    Approve → Moneybird API
                    Reject  → Airtable update + Slack feedback
```

---

## Business Case

| | Manual | Automated |
|---|---|---|
| Time per invoice | 3–6 min | 6–15 sec |
| 50 invoices | 2.5–5 hours | 5–15 min |
| 1,000 invoices/month | 40–80 hours | < 4 hours |

At €35/hour (all-in admin rate) and 1,000 invoices/month:
- Manual: €1,400–€2,800/month
- Automated: ~€140/month
- **Net saving: €1,260–€2,660/month (€15,000–€32,000/year)**

---

## Security & Stability

- **Crash-proof architecture:** Try/catch logic in JavaScript ensures a
  single unreadable invoice does not stop the entire batch. Failed invoices
  are flagged as "⚠️ Review required" and the loop continues.
- **AVG/GDPR compliant:** Data processed via OpenAI Enterprise API
  (not used for training). Temporary files archived immediately after
  processing. Full audit trail in Airtable.
- **Double-approval prevention:** Slack Block Kit buttons are removed
  after the first click.

---

## Known Limitations (V1.0)

1. **Single attachment:** The system processes the first attachment per email.
   Emails with multiple invoices are not yet supported.
2. **Moneybird contact:** Invoices are booked against a catch-all contact
   ("Diversen"). The bookkeeper manually links the final creditor in
   Moneybird to keep the contact database clean.
3. **Non-deterministic AI:** In <1% of cases the AI may write a vendor name
   differently (e.g. "KPN" vs "KPN B.V."), potentially creating a duplicate
   vendor entry. Partially mitigated with LOWER() normalization in Airtable.

---

## Key Concepts Demonstrated

- Two-workflow architecture with shared Airtable data layer
- GPT-4o Vision for unstructured document processing
- Smart business rules (auto-process vs. human approval threshold)
- Duplicate detection on compound key (vendor + invoice number)
- Slack Block Kit for human-in-the-loop approval UI
- Crash-proof batch processing with try/catch and loop architecture
- Full audit trail with AVG/GDPR compliance considerations
- Moneybird API integration with ex-VAT calculation

---

*Built by George Panaite — [boldnode.nl](https://boldnode.nl)
| [LinkedIn](https://linkedin.com/in/george-panaite)*
