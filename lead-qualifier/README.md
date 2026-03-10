# AI Lead Orchestrator — Automated Qualification & Follow-up

An autonomous lead qualification system built in n8n. 
From raw form submission to qualified, routed deal — 
in seconds, without human intervention.

---

## The Problem

Most businesses process inbound leads manually. 
Someone reads the message, judges the intent, 
copy-pastes data into a spreadsheet, and sends a follow-up. 
This takes time, introduces errors, and doesn't scale.

---

## What It Does

1. Captures leads via webhook (Tally form) or manual trigger for testing
2. Normalizes and cleans incoming data
3. Filters duplicates by email address — no lead is processed twice
4. Sends the lead message to Google Gemini for AI analysis
5. Extracts intent and sentiment from the AI response
6. Routes the lead to the right action:
   - Sales VIP (hot lead)
   - Sales Standard
   - Support ticket
   - Manual review (when AI is uncertain)
7. Logs everything to Google Sheets
8. Sends an automated follow-up email via Gmail

---

## Tech Stack

| Tool | Role |
|---|---|
| n8n | Workflow orchestration |
| Google Gemini | Intent & sentiment analysis |
| Google Sheets | Lead logging & CRM |
| Gmail | Automated follow-up emails |
| Tally | Lead intake form |
| JavaScript (code node) | AI output parsing & data merging |

---

## Architecture Decisions

**Why a separate code node for merging data?**

n8n passes only its own output to the next node. 
After the AI node runs, the original customer data 
(name, email) is no longer in the active data stream. 
The code node explicitly retrieves the original data 
from the Deduplication node and merges it with the 
AI output — ensuring clean, complete records downstream.

**Why not let the AI return the customer data too?**

Two reasons:
1. AI models can hallucinate — subtly altering an email 
   address or name without flagging it as an error.
2. Data minimization (GDPR) — the AI only receives 
   what it needs to do its job: the message content. 
   Not the name or email.

**Fallback logic**

If the AI returns unparseable output, the system defaults 
to `INTENTIE: HANDMATIG` and `SENTIMENT: NEUTRAAL`, 
routing the lead to manual review rather than crashing.

---

## Flow Overview
```
Webhook / Manual Trigger
        ↓
Data Normalization
        ↓
Deduplication Firewall (by email)
        ↓
AI Reasoning Agent (Google Gemini)
        ↓
JavaScript: Parse AI output + merge customer data
        ↓
Intent Router (switch node)
        ↓
┌─────────────┬──────────────┬─────────────────┐
Sales VIP   Sales Standard  Support Ticket   Manual Review
        ↓
Gmail follow-up
```

---

## Key Concepts Demonstrated

- Agentic AI integration in a business workflow
- Structured output parsing from unstructured AI responses
- Cross-node data retrieval in n8n
- Duplicate detection
- Human-in-the-loop fallback design
- GDPR-conscious architecture (data minimization)

---

*Built by George Panaite — [boldnode.nl](https://boldnode.nl) 
| [LinkedIn](https://linkedin.com/in/george-panaite)*
