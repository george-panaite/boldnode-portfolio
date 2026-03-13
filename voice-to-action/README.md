# Voice-to-Action: AI Executive Operations Engine

A "fire and forget" voice automation system for managers and entrepreneurs on the go.
Speak one voice message via Telegram and the AI autonomously updates your calendar,
project database and team chat — simultaneously, without opening a single app.

---

## The Problem

Managers and entrepreneurs lose valuable ideas and time because they're constantly
on the move. Quickly noting a task, scheduling a meeting or updating the team
requires constant context switching and screen time. The result: admin piles up
until the end of the day — or gets forgotten entirely.

---

## What It Does

1. Receives a voice note via Telegram
2. Downloads the binary audio file
3. Transcribes audio to text using OpenAI Whisper
4. AI Brain (GPT-4o-mini) analyzes the transcription and generates a structured
   nested JSON object with separate data packages per tool
5. Parallel routing checks which tools are needed (calendar / notion / slack)
6. Autonomous execution across all relevant systems simultaneously

---

## Tech Stack

| Tool | Role |
|---|---|
| n8n | Workflow orchestration |
| Telegram Bot API | Voice note input trigger |
| OpenAI Whisper | Audio-to-text transcription |
| GPT-4o-mini | Intent analysis & structured JSON generation |
| Google Calendar | Meeting scheduling & attendee invites |
| Notion | Project database updates |
| Slack | Team notifications |

---

## Architecture Decisions

**Why a nested JSON object instead of plain text output?**

The AI generates a single nested JSON with separate keys for `calendar`, `notion`
and `slack`. This strict separation prevents cross-contamination between tools —
a task update does not affect the calendar invite, and vice versa. Each tool only
receives exactly the parameters it needs.

**Why parallel If-nodes for routing?**

A single voice message can trigger multiple tools simultaneously. Using parallel
routing nodes (checking `!= null` per key) allows all relevant actions to fire
at the same time rather than sequentially. This keeps execution fast and
independent per tool.

**Why GPT-4o-mini instead of GPT-4o?**

This task — intent classification and structured data extraction — does not
require the full reasoning capacity of GPT-4o. GPT-4o-mini handles it reliably
at significantly lower cost, which matters at scale.

---

## Flow Overview
```
Telegram voice note
        ↓
Download audio file
        ↓
OpenAI Whisper — transcribe to text
        ↓
GPT-4o-mini — analyze intent → nested JSON
        ↓
Parse & clean JSON (code node)
        ↓
┌───────────────┬───────────────┬───────────────┐
calendar?       notion?         slack?
↓               ↓               ↓
Schedule        Update          Notify
Meeting         Project DB      Team
```

---

## Production Upgrade: Contact Management

In this MVP, attendee email addresses are hardcoded in the AI system prompt.
In a production setup, this would be replaced by a dynamic Notion contacts table
containing the names, email addresses and roles of all project stakeholders.

The workflow would then:
1. Extract the attendee name from the transcription
2. Query the Notion contacts table for the matching email address
3. Pass the retrieved email to the Google Calendar node

This keeps contact data centralized, maintainable and project-specific —
without hardcoding any personal information in the workflow.

---

## Key Concepts Demonstrated

- Voice-to-structured-data pipeline (Whisper + GPT-4o-mini)
- Nested JSON architecture for multi-tool routing
- Parallel execution across three external systems
- Zero-UI interaction model (audio only, no dashboard required)
- Cost-conscious model selection (GPT-4o-mini over GPT-4o)
- Production upgrade path documented (dynamic contact management)

---

*Built by George Panaite — [boldnode.nl](https://boldnode.nl)
| [LinkedIn](https://linkedin.com/in/george-panaite)*
