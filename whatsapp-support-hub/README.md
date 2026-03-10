# 24/7 Smart Customer Support Hub (v2.0)

An intelligent WhatsApp automation system for small business owners.
Incoming messages are handled by a dual-agent AI, synced to a CRM,
and forwarded to the owner via Telegram — with full human-in-the-loop
control per customer.

---

## The Problem

Small business owners like plumbers, contractors and freelancers
receive WhatsApp messages around the clock. Manually reading,
qualifying and responding to every message is time-consuming and
leads to missed opportunities. This system handles it autonomously
while keeping the owner in control.

---

## What It Does

**Incoming message handling:**
1. Receives inbound WhatsApp messages
2. Checks if the sender is an existing customer or new lead
3. Automatically creates a dedicated Telegram topic per customer
4. Merges all customer data for further processing

**Dual-agent AI:**
5. Checks if the bot is active for this customer (ACTIVE status)
6. Agent 1 — Receptionist: handles the conversation with the customer
7. JavaScript security layer: blocks hallucinations and internal codes
   from leaking into customer responses
8. Agent 2 — Analyst: analyses sentiment and extracts key information

**CRM sync & notifications:**
9. Updates Google Sheets (CRM) in real time
10. Spam filter: suppresses notifications for low-value messages
11. Sends update to the correct Telegram topic with action buttons:
    Start, Stop, and Call Direct

**Bi-directional sync:**
12. Owner types a reply in Telegram
13. Message is sent directly to the customer via WhatsApp

**Human-in-the-loop controls:**
14. Webhook: STOP — pauses the AI for a specific customer
15. Webhook: START — reactivates the AI for a specific customer
16. Status changes take effect in real time

---

## Tech Stack

| Tool | Role |
|---|---|
| n8n | Workflow orchestration |
| WhatsApp Business API | Inbound & outbound messaging |
| OpenAI GPT-4o | Dual-agent AI (receptionist + analyst) |
| Telegram Bot API | Owner notifications & bi-directional sync |
| Google Sheets | CRM — customer data & conversation summaries |
| JavaScript (code node) | Security filter for AI output |
| Webhooks | Human-in-the-loop START/STOP controls |

---

## Architecture Decisions

**Why two AI agents instead of one?**

Each agent has a single responsibility. The Receptionist focuses
entirely on conversation quality. The Analyst focuses entirely on
extracting structured information and filtering noise. Combining
both tasks in one agent reduces reliability and makes the system
harder to debug.

**Why a JavaScript security layer between AI and WhatsApp?**

The AI can occasionally output internal status codes or formatting
that should never reach the customer. The code node acts as a
filter — blocking any response that contains internal patterns
before it is sent via WhatsApp.

**Why Telegram for owner notifications?**

Telegram's Topics feature allows one channel per customer —
the owner sees a clean, organised inbox with one thread per contact.
Combined with the bi-directional sync, the owner can reply directly
from Telegram without opening WhatsApp.

**Why webhooks for START/STOP instead of a dashboard?**

Speed. The owner can pause or reactivate the AI per customer with
a single button tap inside Telegram — no separate app or interface
required.

---

## Flow Overview
```
Inbound WhatsApp message
        ↓
Check: existing customer or new lead?
        ↓
Create Telegram Topic (if new)
        ↓
Merge customer data
        ↓
Check: bot status ACTIVE?
    ↓ Yes                    ↓ No
AI Receptionist          Telegram notification
        ↓                    (bot is off)
Security filter
        ↓
Send WhatsApp reply
        ↓
AI Analyst — extract info & filter noise
        ↓
Update CRM (Google Sheets)
        ↓
Telegram notification to owner

--- Sub-flows ---

Telegram → WhatsApp (bi-directional sync)
Webhook STOP → pause bot per customer
Webhook START → reactivate bot per customer
```

---

## Key Concepts Demonstrated

- Dual-agent AI architecture with separation of responsibilities
- Bi-directional messaging sync (WhatsApp ↔ Telegram)
- Human-in-the-loop controls via webhook + Telegram buttons
- Real-time CRM updates within the same flow
- JavaScript security layer to sanitize AI output
- Per-customer bot status management
- Spam filtering to reduce notification noise

---

> **Note:** This workflow was built for a Dutch plumbing business
> (fictional demo). System prompts and CRM fields are in Dutch.

---

*Built by George Panaite — [boldnode.nl](https://boldnode.nl)
| [LinkedIn](https://linkedin.com/in/george-panaite)*
