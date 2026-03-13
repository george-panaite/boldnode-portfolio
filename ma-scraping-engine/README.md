# Automated Scraping Engine — M&A Due Diligence

An autonomous due diligence system for "Buy & Build" acquisition strategies.
From a single company name in the chat to a complete risk report — with dynamic
multi-platform scraping, AI sentiment analysis, and real-time Slack alerts.

---

## The Problem

For consultancy firms and Private Equity companies with a Buy & Build strategy,
the online reputation of an acquisition target is critical. Hidden technical debt
or a failing support department can destroy the value of a company — but these
risks often only surface after the acquisition. Analysts spend hours manually
searching, reading and categorising customer feedback scattered across the internet,
with no structured output to use in price negotiations.

---

## What It Does

The user types a single domain name into the chat. The workflow does the rest.

**Phase 1 — Automated Reconnaissance**
1. Chat trigger receives the target company URL or name
2. SerpAPI fetches organic Google results
3. AI router identifies the correct Trustpilot and G2 profile URLs,
   ignoring competitors and irrelevant results
4. Regex strips markdown artifacts and splits the URLs for the scrapers

**Phase 2 — Parallel Data Extraction (Async)**
5. Switch node routes each URL to the correct Apify scraper (G2 or Trustpilot)
6. Both scrapers run simultaneously
7. Merge node waits for all data streams to complete before continuing

**Phase 3 — Sentiment Analysis & Database Logging**
8. AI agent scores each review on Category (Bugs, Support, Pricing),
   Sentiment (Positive/Neutral/Negative) and Churn Risk (1–5)
9. Structured Output Parser enforces schema — no hallucinations reach the database
10. All data is logged to an Airtable M&A database

**Phase 4 — Distribution (Alerts & Reports)**
11. Risk filter checks for churn risk score 4 or 5 ("Action Needed")
12. JavaScript aggregates all critical risks into one formatted Slack alert
13. Second LLM reads the full dataset and writes a strategic M&A report
    (Executive Summary, Risk Analysis, Post-Merger advice)
14. Google Doc is created dynamically, report is written into it,
    link is shared in Slack and the chat

---

## Tech Stack

| Tool | Role |
|---|---|
| n8n | Workflow orchestration |
| SerpAPI | Google search for review profile discovery |
| Apify | Trustpilot & G2 review scraping |
| OpenAI GPT-4o | Sentiment analysis & M&A report generation |
| OpenAI GPT-4o-mini | AI routing (platform URL identification) |
| LangChain Structured Output Parser | Schema enforcement for AI output |
| Airtable | M&A review database |
| Google Docs | Dynamic report generation |
| Slack | Risk alerts & report delivery |
| JavaScript (code nodes) | Batch alert formatting, regex cleaning |

---

## Architecture Decisions

**Why a Switch architecture for scrapers instead of a single tool?**

Review platforms actively block scrapers with anti-scraping firewalls. A
single monolithic scraper fails entirely when blocked. The Switch architecture
routes each platform to its own dedicated Apify actor. Adding a new platform
is as simple as adding a new case to the Switch node — no rebuilding required.

**Why async scraping with a Merge node?**

Running Trustpilot and G2 scrapers simultaneously cuts total execution time
roughly in half. The Merge node acts as a gate — it waits for all data streams
to complete before passing data downstream, ensuring no reviews are lost.

**Why a Structured Output Parser for sentiment analysis instead of prompt instructions?**

Prompt instructions alone cannot guarantee valid JSON output from an LLM.
The Structured Output Parser enforces a strict schema and degrades the AI to
a reliable data parser — eliminating the risk of malformed output breaking
the workflow downstream.

**Why batch alerting via JavaScript instead of one Slack message per review?**

A target company with 30 negative reviews would generate 30 separate Slack
notifications — unreadable and disruptive. The JavaScript code node aggregates
all critical risks into a single formatted message with one API call.

---

## Flow Overview
```
Chat trigger (company name / domain)
        ↓
SerpAPI — Google search
        ↓
AI router — identify Trustpilot & G2 URLs
        ↓
Regex — clean & split URLs
        ↓
Switch — route to correct scraper
    ↓               ↓
Trustpilot        G2
scraper           scraper
    ↓               ↓
Merge — wait for all data
        ↓
AI agent — score each review
(Category / Sentiment / Churn Risk)
        ↓
Airtable — log to M&A database
        ↓
┌─────────────────────┬─────────────────────┐
Risk filter           Aggregate all reviews
(churn risk 4–5)             ↓
↓                     AI — write M&A report
JS — bundle alerts           ↓
↓                     Google Docs — generate
Slack alert                  ↓
                      Slack — share link
```

---

## Key Concepts Demonstrated

- Multi-platform async scraping with Merge gate
- AI-powered platform discovery via SerpAPI + LLM routing
- Structured Output Parser for schema-enforced AI responses
- Batch alert aggregation via JavaScript (single Slack API call)
- Modular Switch architecture for scraper extensibility
- Two-track output: operational alerts + strategic report
- Dynamic Google Docs generation from AI output

---

*Built by George Panaite — [boldnode.nl](https://boldnode.nl)
| [LinkedIn](https://linkedin.com/in/george-panaite)*
