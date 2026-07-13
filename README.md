# 📬 Multi-Source Weekly Leadership Digest Automation

A Make.com automation that replaces 3 hours of manual Monday morning report compilation with a fully automated, AI-synthesized leadership digest — delivered to the executive team's inbox every Monday at 8 AM.

---

## 📋 Project Overview

| Field | Details |
|---|---|
| **Platform** | Make.com |
| **LLM** | Google Gemini 2.5 Flash (swappable to Claude Haiku in ~30 seconds) |
| **Data Sources** | Notion project updates, Slack leadership channel, Calendar events |
| **Output** | HTML-formatted email — color-coded, source-tagged, executive-scannable |
| **Schedule** | Weekly trigger — Monday 8 AM |
| **Ops per run** | ~11 Make operations |
| **LLM cost per run** | ~$0.01 (Claude Haiku) |
| **Time saved** | ~3 hours/week → ~150 hours/year |
| **Status** | Deployed, tested, operational |

---

## 🎯 Problem

Every Monday morning, someone on the team spent 3 hours manually:
- Reviewing Notion for project status updates
- Scanning the Slack leadership channel for decisions, blockers, and announcements
- Pulling calendar events to summarize key meetings

Then writing a coherent digest and emailing it to leadership.

This scenario eliminates that entirely.

📊 [View the full VSM — current vs. future state](https://github.com/robertciceroson/process-engineering-portfolio/blob/main/executive-report-automation-vsm.pdf)

---

## 🏗️ Architecture

```
Weekly Trigger (Monday 8 AM)
        ↓
Google Sheets → Notion_Updates tab     → Array Aggregator
        ↓
Google Sheets → Slack_Messages tab     → Array Aggregator
        ↓
Google Sheets → Calendar_Events tab    → Array Aggregator
        ↓
Gemini 2.5 Flash — LLM Synthesis
  (strict JSON output, source-tagged, 5-section schema)
        ↓
Gmail — HTML-formatted leadership digest → Inbox
```

### Module Breakdown

| # | Module | Purpose |
|---|---|---|
| 1 | Tools — Basic Trigger | Emits a single starter bundle. Weekly schedule (Monday 8 AM) set via the clock icon. |
| 2 | Google Sheets — Search Rows | Pulls all rows from `Notion_Updates` tab dated within the last 7 days. |
| 3 | Tools — Array Aggregator | Collapses Notion rows into a single bundle to prevent downstream bundle multiplication. |
| 4 | Google Sheets — Search Rows | Pulls all rows from `Slack_Messages` tab dated within the last 7 days. |
| 5 | Tools — Array Aggregator | Collapses Slack rows into a single bundle. |
| 6 | Google Sheets — Search Rows | Pulls all rows from `Calendar_Events` tab dated within the last 7 days. |
| 7 | Tools — Array Aggregator | Collapses Calendar rows into a single bundle. |
| 8 | Google Gemini — Create a Completion | LLM synthesis. Strict JSON output mode, source-tagged items, 5-section schema. |
| 9 | Gmail — Send an Email | Delivers the HTML-formatted leadership digest to the configured recipient. |

---

## 📤 Digest Output Structure

The email is structured for executive readability — color-coded sections, source-tagged items, and an executive summary at the top. Each line carries a `[Notion]`, `[Slack]`, or `[Calendar]` provenance tag.

```
📬 Weekly Digest — [date range]

Executive Summary
  2–3 sentences on the most important themes of the week

✅ Wins
  [Source] What went well

⚠️ Risks
  [Source] What's at risk or needs intervention

🔄 What Changed
  [Source] Material changes since last week

🎯 Needs Leadership Attention
  [Source] Items requiring exec decision or escalation
```

See [`sample-output/sample-digest-email.html`](sample-output/sample-digest-email.html) for a rendered example.

---

## 🧠 Key Design Decisions

**Sequential pulls with Array Aggregators instead of a Router.**
Make's Router branches don't truly converge for a single downstream LLM call. Sequential pulls with Array Aggregators is the canonical pattern — keeps operations linear (~11 ops/run) and is easier to debug when a single source fails.

**Strict JSON output mode on the LLM call.**
The Gemini module is configured with `responseMimeType: application/json`. No markdown fences, no commentary — output is reliably parseable by the Gmail template without a separate parse module.

**Empty-signal-beats-fake-signal principle.**
The system prompt explicitly instructs the model: *"If a section has nothing meaningful, return an empty array."* This prevents hallucinated filler content from reaching executives.

**Source-tagged items.**
Every digest line carries `[Notion]`, `[Slack]`, or `[Calendar]` so executives know exactly where each item came from — and can verify or follow up directly.

**Built on Gemini, easy swap to Claude.**
Gemini Flash was used for initial build and testing (free-tier key). Swapping to Anthropic Claude Haiku is a single module change once an API key is configured — no prompt changes required.

---

## 🤖 LLM System Prompt

The Gemini module uses a "senior chief of staff" persona with the following core principles:

- Empty signal beats fake signal — return empty arrays when a section has nothing meaningful
- Source-tag every item: `[Notion]`, `[Slack]`, or `[Calendar]`
- Be specific — use real names, real numbers, real dates from the data
- No fluff, no preamble, no editorializing — executives scan in 90 seconds
- One sentence per item, max two

Output schema enforced:

```json
{
  "week_range": "YYYY-MM-DD to YYYY-MM-DD",
  "executive_summary": "2-3 sentences on key themes",
  "wins": ["[Source] description"],
  "risks": ["[Source] description"],
  "what_changed": ["[Source] description"],
  "needs_leadership_attention": ["[Source] description"]
}
```

---

## 📁 Repository Structure

```
weekly-leadership-digest-automation/
├── Executive-Report-Automation-Updated.json   # Make.com blueprint — import to clone
├── README.md                                   # This file
├── sample-output/
│   └── sample-digest-email.html               # Rendered sample of the email output
└── data-schema/
    └── sheets-schema.md                        # Column layout for the 3 Google Sheets input tabs
```

---

## 🚀 How to Use

### Import the Blueprint

1. In Make.com, go to **Scenarios → Create a new scenario**
2. Click the three-dot menu → **Import Blueprint**
3. Upload `Executive-Report-Automation-Updated.json`

### Configure Connections

Replace the following with your own credentials:

| Module | What to configure |
|---|---|
| Modules 2, 4, 6 (Google Sheets) | Your Google account + Spreadsheet ID |
| Module 8 (Gemini) | Your Gemini API key (or swap to Claude) |
| Module 9 (Gmail) | Your Gmail account + recipient address |

### Set Up the Google Sheet

Create a Google Sheet with three tabs matching the schema in [`data-schema/sheets-schema.md`](data-schema/sheets-schema.md). Point modules 2, 4, and 6 to the correct tabs.

### Go Live

- Toggle the scenario **On**
- The trigger is set to Monday 8 AM — adjust via the clock icon on Module 1
- Review the first few weekly outputs; tune the system prompt if needed

---

## 🔄 Phase 2: Live Source Connections

The current build uses Google Sheets as stand-ins for Notion, Slack, and Calendar. Replacing these with live API connections is straightforward:

| Stand-in | Live replacement |
|---|---|
| `Notion_Updates` sheet | Make's Notion module — query a database |
| `Slack_Messages` sheet | Make's Slack module — fetch channel history |
| `Calendar_Events` sheet | Make's Google Calendar module — list events |

The LLM module and Gmail module require no changes for this upgrade.

---

## 💡 Related Portfolio Projects

- [AI Workflow ROI Analysis](https://docs.google.com/spreadsheets/d/1EVz2vaeOSHu5mjn9EpmmQTRi55DBbRtIgxBc3a2PN_I/) — quantifying automation value
- [HR Policy QA Bot](https://hr-policy-app-bot-mfgzhuyzqgkkdglxg4mnj2.streamlit.app) — RAG pipeline (LangChain + FAISS + Llama 3.3 70B)
- [Process Engineering Portfolio](https://github.com/robertciceroson/process-engineering-portfolio) — 19 VSM + BPMN diagrams

---

## Author

**Robert Cicero Son**  
AI Business Analyst · Scrum Master · Process Engineer · Make.com Automation Architect  
[linkedin.com/in/robert-son-0b33b3bb](https://linkedin.com/in/robert-son-0b33b3bb) · [github.com/robertciceroson](https://github.com/robertciceroson)
