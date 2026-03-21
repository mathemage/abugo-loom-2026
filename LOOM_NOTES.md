# Loom Preparation Notes — Eliška's Marketing Analytics AI System

> **Assignment:** 30-minute Loom (max 10 min recording). No code required.
> **Context:** Two weeks into the job at Abugo. Eliška is a marketing analytics colleague covering 12 countries, living in SEMrush / Google Search Console / Google Ads.

---

## 1. What do I ask Eliška?

*Before touching any tool or writing any code — understand the problem, not the solution.*

### 1a. About her current workflow

| Question | Why I ask |
|---|---|
| Walk me through a typical week — which tools do you open, in which order, and what are you looking for? | Builds a mental model of the actual workflow before imagining automation |
| Which of those tasks feels most like "monkey work" — repeatable, low-judgment, just time-consuming? | Distinguishes automation candidates from genuine analytical work |
| When you said the French acquisition drop took half a day — can you replay that session? What did you actually click through? | Grounds the conversation in a real, named incident rather than generalities |
| How often do you do each of these tasks — daily, weekly, ad hoc? | Determines whether batch (scheduled) or event-driven (triggered) fits better |
| Who else reads your outputs — just you, or does a team/manager act on them? | Defines downstream consumers and output format requirements |

### 1b. About success and failure

| Question | Why I ask |
|---|---|
| If an AI surface flagged the French drop 6 hours earlier, what would you actually have done differently? | Tests whether speed is really the lever, or whether something else blocks action |
| What's a "false alarm" in your world? If the system pings you with noise, what's the cost? | Sets the precision/recall trade-off before we build anything |
| What does "insane search intent" mean to you — can you show me an example of a keyword you'd call that? | Calibrates vocabulary so we build to her mental model, not ours |
| Have you tried anything with AI already — ChatGPT, SEMrush AI features, anything? What worked and what didn't? | Avoids rebuilding something she already dismissed; finds real gaps |

### 1c. About data access and constraints

| Question | Why I ask |
|---|---|
| Do you have API access to SEMrush, GSC, and Google Ads — or do you mostly use the UIs? | Determines data pipeline feasibility before promising anything |
| Are the 12 countries in separate accounts/properties, or unified? | Single biggest source of complexity in multi-country systems |
| Is there any data that can't leave your machine or must stay in EU-only infrastructure? | GDPR / data residency constraints can kill a design entirely |
| How do you currently document findings — a shared doc, a Slack channel, Notion, email? | Defines the output layer: where does the agent *deliver* its answer? |

### 1d. Priority triage

| Question | Why I ask |
|---|---|
| Of the four problems you mentioned (drop detection, competitor spotting, keyword mis-ranking, untapped intent) — if you could only fix one in a week, which one saves you the most pain? | Avoids building four half-features; forces a real stack-rank |
| What does "done" look like for the first version? A Slack message? A Google Sheet? A PDF report? | Anchors scope to a concrete, testable deliverable |

---

## 2. What do I ask the Head of AI Automations?

*These questions are about infrastructure, standards, and what's already been built.*

| Question | Why I ask |
|---|---|
| Is there a company-wide agent framework or internal SDK I should build on top of? | Avoids NIH; aligns with existing patterns |
| What's the approved LLM provider and model tier — OpenAI GPT-4o, Claude, self-hosted? | Affects latency, cost, and data-residency constraints immediately |
| Are there existing connectors for SEMrush / GSC / Google Ads, or do I build them from scratch? | Biggest time sink in week one if missing |
| What does the standard logging / observability stack look like (Datadog, Grafana, custom)? | Can't bolt logging on later; need to know from day one |
| Where do long-running agent workflows run — Lambda, ECS, Airflow, something else? | Determines execution model (serverless vs. container vs. DAG) |
| Is there a human-approval layer or "AI inbox" pattern already in use? | Eliška's workflow will need a review step; don't reinvent it |
| What are the evaluation standards — do we use an LLM-as-judge, a human eval rubric, automated metrics? | Prevents shipping an agent nobody can measure |
| What's the on-call and incident-response expectation for AI agents that break? | Sets reliability expectations before I promise a schedule |

---

## 3. What system might actually help?

### 3a. System sketch

```
┌─────────────────────────────────────────────────────────────────────┐
│                  MARKETING INTELLIGENCE AGENT                        │
│                                                                       │
│  TRIGGERS (Scheduled + Event-Driven)                                 │
│  ├── Daily 07:00 CET: pull overnight data from GSC + SEMrush         │
│  ├── Hourly: watch Google Ads spend anomaly threshold                │
│  └── Ad hoc: Eliška can type "check France" in Slack                │
│                                                                       │
│  PIPELINE                                                             │
│  ┌──────────┐   ┌──────────────┐   ┌───────────────┐                │
│  │ Data     │──▶│  Analysis    │──▶│  Synthesis    │                │
│  │ Fetcher  │   │  Agent       │   │  Agent (LLM)  │                │
│  │ (APIs)   │   │  (rules +    │   │  writes human │                │
│  │          │   │   LLM)       │   │  readable     │                │
│  └──────────┘   └──────────────┘   │  summary      │                │
│                                    └───────┬───────┘                 │
│                                            │                         │
│                         ┌──────────────────▼──────────────────┐     │
│                         │       HUMAN-IN-THE-LOOP LAYER        │     │
│                         │  Eliška gets Slack digest with:      │     │
│                         │  • Anomaly flag + confidence score   │     │
│                         │  • 3–5 bullet insight summary        │     │
│                         │  • "Investigate" / "Dismiss" buttons │     │
│                         │  • Link to full data in Notion/Sheet │     │
│                         └──────────────────┬──────────────────┘     │
│                                            │                         │
│               ┌──────────────────┬─────────▼────────────────┐       │
│               │  AUTONOMOUS      │  HUMAN-APPROVED           │       │
│               │  (no approval)   │  (requires her click)     │       │
│               ├──────────────────┼───────────────────────────┤       │
│               │ • Fetch data     │ • File a keyword gap       │       │
│               │ • Detect anomaly │   ticket in Jira           │       │
│               │ • Draft summary  │ • Escalate to marketing    │       │
│               │ • Send Slack     │   lead                     │       │
│               │   digest         │ • Recommend budget shift   │       │
│               └──────────────────┴───────────────────────────┘       │
└─────────────────────────────────────────────────────────────────────┘
```

**What it does:**
- Pulls keyword rankings, traffic, and spend data across all 12 countries automatically
- Detects anomalies (drops, spikes) using statistical rules + LLM context
- Spots new competitor domains appearing in keyword SERPs
- Identifies keyword/content mismatches ("this article ranks for the wrong keyword")
- Surfaces high-intent untapped keywords by country

**What Eliška gets at the end:**
A concise Slack digest every morning (and an alert if something fires intra-day) with:
- Country, metric, change, magnitude, confidence
- 2–3 sentence LLM-written explanation of likely cause
- A direct link to the underlying data
- One-click action buttons for follow-through tasks that need her approval

**Where she stays in the loop:**
Any action that has downstream consequences (escalation, budget recommendation, tagging a ticket) requires her explicit "Approve" click. The agent never writes to external systems without her go-ahead.

**Where the agent acts autonomously:**
Data collection, anomaly detection, draft generation, and notification delivery. These are pure information-surfacing steps with zero side-effects.

---

### 3b. One-week build plan

**Approach:** Python-first, async, thin layer of orchestration over existing APIs.

| Layer | Choice | Reasoning |
|---|---|---|
| Language | Python 3.12 | Broadest API client ecosystem; team likely already uses it |
| Agent orchestration | LangGraph (or Prefect if heavy DAGs exist) | LangGraph gives controllable step-by-step flow with built-in state; easy to add human-in-the-loop nodes |
| LLM | GPT-4o via OpenAI (or existing company LLM) | Best instruction-following for structured JSON output; swap model via env var |
| Data connectors | SEMrush API v3, Google Search Console API, Google Ads API | Official clients; no scraping |
| Delivery | Slack Bolt (webhook or socket mode) | Eliška already lives in Slack; zero new UI to learn |
| Storage | Postgres (or existing company DB) | Persist run history, anomaly baselines, dismissed alerts |
| Scheduling | GitHub Actions cron / AWS EventBridge / Airflow (whatever exists) | Match company standard — don't add infra |

**Logging:**
- Structured JSON logs (stdout) with `run_id`, `country`, `trigger`, `step`, `duration_ms`, `llm_tokens_used`
- Every LLM call logged with prompt hash + response (no PII) for replay
- Alert if a country's data fetch fails — Eliška should know the digest is incomplete

**Error handling:**
- Each country processed independently — one API failure doesn't abort the rest
- Exponential backoff on rate limits (SEMrush is strict)
- If LLM synthesis fails, fall back to templated rule-based summary (never send a blank digest)
- Dead-letter queue for failed Slack deliveries

**Evaluations:**
- **Week 1 (offline):** Replay last 4 weeks of data; manually verify that every known anomaly Eliška remembers was flagged (recall test)
- **Week 2 (online):** Eliška rates each Slack digest "useful / noisy / missed something" — tracked in a simple 3-button Slack survey
- **LLM-as-judge:** For the LLM-written explanation, a separate GPT-4o call checks: "Is this explanation factually consistent with the data? Yes/No + reason" — logged for weekly review
- **Baseline KPI:** Time Eliška spends in SEMrush/GSC before vs. after (self-reported weekly)

---

## 4. What would I NOT build here, and why?

### ❌ A fully autonomous agent that acts on findings without approval

**Why not:** Eliška said herself "I don't always know what I'm looking for until I find it." That's a signal that the domain is not yet well-specified enough for autonomous action. Keyword decisions affect campaigns across 12 countries. A false-positive auto-action could mis-tag content, shift budgets, or file noise tickets. The cost of an autonomous mistake far outweighs the time saved by removing Eliška from the loop.

### ❌ A custom dashboard / BI tool

**Why not:** Eliška already has SEMrush and Google Search Console dashboards. Building another UI she has to check is adding pull-based work, not removing it. The value is in pushing the right signal to where she already is (Slack), not in a prettier view she has to remember to open.

### ❌ Multi-modal / voice / chat interface (for now)

**Why not:** A conversational agent sounds exciting but is extremely hard to evaluate and debug. "French acquisition dropped" is not a well-formed query — an open-ended chat interface will give inconsistent, hard-to-reproduce answers. Start with structured, scheduled digests. Add a chat interface in v2 only after the structured layer proves its value.

### ❌ Training a custom ML model on her data

**Why not:** We have fewer than 12 country-level data points per day. That's nowhere near enough to train a reliable anomaly detection model from scratch. Statistical methods (z-score, IQR, rolling mean ± 2σ) will outperform a custom model with this data volume, be explainable, and be deployable in hours instead of weeks.

### ❌ Full coverage of all 12 countries simultaneously on day one

**Why not:** Pick 2–3 countries (France and Czech Republic, since those appeared in the Slack message) for the first sprint. Prove the pipeline works end-to-end, calibrate the anomaly thresholds, collect Eliška's feedback. Then scale out the country list. Shipping a fragile 12-country system in week one is a recipe for alert fatigue and eroded trust.

---

*Last updated: 2026-03-21*
