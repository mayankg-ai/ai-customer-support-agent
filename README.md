# 🤖 AI Customer Support Agent

**An autonomous support triage system built on n8n — turns raw customer messages into structured, routed, SLA-tracked tickets in seconds.**

[![n8n](https://img.shields.io/badge/Built%20with-n8n-EA4B71?style=flat-square&logo=n8n&logoColor=white)](https://n8n.io)
[![Google Gemini](https://img.shields.io/badge/LLM-Google%20Gemini-4285F4?style=flat-square&logo=googlegemini&logoColor=white)](https://ai.google.dev)
[![Google Sheets](https://img.shields.io/badge/Data-Google%20Sheets-34A853?style=flat-square&logo=googlesheets&logoColor=white)](https://sheets.google.com)
[![Status](https://img.shields.io/badge/status-production--ready-success?style=flat-square)]()

Most "AI support bots" stop at generating a polite reply. This one does the operational work that actually costs support teams money: it reads the message, judges the emotional temperature, classifies the issue, assigns priority and ownership, stamps an SLA, and writes a complete ticket record to a live database — without a human touching it.

---

## 🎯 Business Problem

Support teams lose more time to **triage** than to the actual fixes.

Every inbound message forces a human to answer the same six questions before real work begins:

| Question | Cost when done manually |
|---|---|
| What is this customer actually asking? | 1–3 min of reading and interpretation |
| How upset are they? | Subjective, inconsistent across agents |
| Which category does this belong to? | Misrouted tickets bounce between teams |
| How urgent is it? | Priority inflation — everything becomes "high" |
| Who owns it? | Handoff delays, dropped ownership |
| When is it due? | SLAs breached before anyone notices |

The downstream damage is predictable and expensive:

- ⏱️ **Slow first response** — the single strongest driver of customer churn signals
- 🔀 **Misrouted tickets** — an average of 2 extra hops before reaching the right team
- 😤 **Missed escalations** — angry high-value customers sit in the same queue as password resets
- 📉 **No usable data** — nothing is logged in a structured way, so leadership can't see patterns
- 💸 **Headcount that scales linearly with volume** — 2× tickets means 2× agents

Small teams and D2C brands feel this hardest: they have real support volume but no budget for Zendesk-tier tooling or a dedicated triage layer.

---

## 💡 Solution

A single n8n workflow that acts as an **autonomous Tier-0 support agent**.

A customer message enters. The AI Agent — powered by a Google Gemini model with persistent conversation memory — interprets it, extracts 18 structured business fields, writes a complete ticket row to Google Sheets, and returns a professional, context-aware reply to the customer.

```
Customer message  →  AI reasoning  →  Structured ticket  →  Routed + SLA-tracked  →  Reply sent
                        (< 5 seconds, no human in the loop)
```

**What makes this different from a chatbot:**

- It produces **structured data**, not just conversation — every interaction becomes a queryable record
- It makes **routing and priority decisions**, not just replies
- It carries **conversation memory**, so follow-up messages are understood in context
- It attaches a **confidence score** to its own output, so low-confidence cases can be flagged for human review
- The database is **Google Sheets** — meaning any non-technical ops manager can read, filter, and act on it on day one

---

## ⚡ Features

### 🧠 Intelligence Layer
- **Intent understanding** — parses free-form, messy, real-world customer language
- **Sentiment detection** — positive / neutral / negative classification on every message
- **Emotion tagging** — a finer-grained read than sentiment alone (frustrated, confused, anxious, satisfied)
- **Conversation memory** — multi-turn context retention so the agent doesn't ask customers to repeat themselves
- **Confidence scoring** — the agent self-reports certainty, creating a natural human-in-the-loop trigger

### 🗂️ Triage & Routing Layer
- **Automatic issue classification** into support categories
- **Priority assignment** based on issue type, urgency, and customer sentiment combined
- **Team routing** — each ticket is assigned to the correct support team automatically
- **Escalation flagging** — high-risk conversations are marked before they become complaints
- **Customer type awareness** — routing and priority adapt to customer tier
- **CSAT risk prediction** — flags conversations likely to end in a poor satisfaction score

### ⏳ SLA Layer
- **Ticket ID generation** — unique, traceable identifier per issue
- **Estimated response time** calculated from priority
- **SLA deadline stamping** — an explicit due timestamp on every ticket
- **Status tracking** from creation onward

### 💬 Customer Experience Layer
- **Professional response generation** — on-brand, empathetic, context-aware replies
- **Instant acknowledgement** — first response time measured in seconds, not hours

### 📊 Data Layer
- **Full ticket persistence to Google Sheets** — 18 structured fields per ticket
- **Reporting-ready output** — the sheet doubles as a live support analytics dashboard

---

## 🏗️ Workflow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    n8n WORKFLOW EXECUTION                        │
└─────────────────────────────────────────────────────────────────┘

        ┌──────────────────────────┐
        │  💬 Chat Message Trigger  │   Inbound customer message
        └────────────┬─────────────┘
                     │
                     ▼
        ┌──────────────────────────┐
        │      🤖 AI Agent          │◄──────┐
        │  (reasoning + tool use)   │       │
        └────────────┬─────────────┘       │
                     │                     │
         ┌───────────┼───────────┐         │
         ▼           ▼           ▼         │
   ┌──────────┐ ┌─────────┐ ┌─────────────┴──────┐
   │ 🧠 Gemini │ │ 🗃️ Chat │ │  📊 Google Sheets   │
   │  (LLM)    │ │ Memory  │ │      (Tool)         │
   └──────────┘ └─────────┘ └────────────────────┘
         │           │                │
         │           │                │
         ▼           ▼                ▼
   Classification  Multi-turn    Ticket row
   Sentiment       context       persisted
   Priority        retention     (18 fields)
   SLA logic
   Response draft
                     │
                     ▼
        ┌──────────────────────────┐
        │  ✅ Response to Customer  │
        │  + Ticket logged & routed │
        └──────────────────────────┘
```

### Node Breakdown

| Node | Role |
|---|---|
| **Chat Message Received** | Entry point — captures inbound customer messages |
| **AI Agent** | Orchestrator — reasons over the message and decides which tools to call |
| **Google Gemini (Chat Model)** | Language and reasoning engine behind the agent |
| **Conversation Memory** | Maintains per-session context across multiple turns |
| **Google Sheets (Tool)** | Structured ticket persistence and retrieval |

### Ticket Schema

The workflow writes the following fields to Google Sheets on every ticket:

| # | Field | Type | Purpose |
|---|---|---|---|
| 1 | Customer Name | Text | Identity |
| 2 | Email | Text | Contact + dedup key |
| 3 | Ticket ID | Text | Unique traceable reference |
| 4 | Category | Enum | Routing input |
| 5 | Priority | Enum | Queue ordering |
| 6 | Urgency | Enum | Time-sensitivity signal |
| 7 | Sentiment | Enum | Customer emotional state |
| 8 | Emotion | Text | Granular emotional read |
| 9 | Summary | Text | One-line issue digest for agents |
| 10 | Status | Enum | Lifecycle tracking |
| 11 | Assigned Team | Text | Ownership |
| 12 | Customer Type | Enum | Tier-aware handling |
| 13 | Confidence Score | Numeric | Agent self-assessment |
| 14 | Estimated Response Time | Duration | Expectation setting |
| 15 | SLA Deadline | Timestamp | Breach monitoring |
| 16 | Escalation Status | Enum | Human-review trigger |
| 17 | CSAT Risk | Enum | Churn early-warning signal |
| 18 | Created At | Timestamp | Audit trail |

---

## 🛠️ Tech Stack

| Layer | Technology |
|---|---|
| **Orchestration** | n8n (workflow automation) |
| **AI Agent Framework** | n8n AI Agent node (tool-calling architecture) |
| **LLM** | Google Gemini (Flash class — optimized for latency and cost) |
| **Memory** | n8n Conversation Memory (session-scoped context) |
| **Database** | Google Sheets API |
| **Interface** | n8n Chat Trigger (webhook-ready for site widgets, WhatsApp, or email gateways) |
| **Prompt Design** | Structured system prompting with enforced JSON-style field extraction |

---

## 🧩 Sample Business Use Cases

**🛒 D2C / E-commerce brand**
Order status, refund, and delivery complaints are auto-classified and routed. Angry high-value customers are flagged for escalation before they post a public review.

**💻 SaaS company**
Bug reports separate cleanly from billing questions and feature requests. Engineering stops triaging billing tickets. P1 issues get an SLA stamp the moment they arrive.

**🏢 Agency / service business**
Client requests are logged as tickets with owners and deadlines instead of disappearing into a WhatsApp thread.

**🏥 Clinic / local service business**
Appointment, billing, and general enquiries are separated automatically, with an instant professional reply outside business hours.

**📈 Support leadership**
The Sheet becomes a live analytics layer — category volume trends, sentiment over time, SLA breach rate, and CSAT risk concentration, without any BI tooling.

---

## 📈 Business Impact

| Metric | Before | After |
|---|---|---|
| First response time | Hours (or next business day) | **Seconds** |
| Triage time per ticket | 3–5 minutes of human effort | **~0** |
| Routing accuracy | Depends on the agent on duty | **Consistent, rule-driven** |
| Missed escalations | Discovered after the complaint | **Flagged at intake** |
| Ticket data captured | Inconsistent or none | **18 structured fields, every time** |
| Cost of scaling volume | Linear with headcount | **Near-flat** |

**In plain terms:** a team handling 100 tickets a day recovers roughly 5–8 hours of pure triage labour per day, and gains a structured dataset it never had before. Agents stop sorting and start solving.

> ⚠️ Figures above reflect the operational load the workflow removes, based on standard manual triage timings — not vendor benchmarks.

---

## 📸 Screenshots

### 1. n8n Workflow Architecture
![n8n Workflow](worflow.png)
*The complete agent workflow — chat trigger, AI Agent orchestration, Gemini model, conversation memory, and Google Sheets tooling.*

### 2. AI Agent Responding to a Customer
![AI Response](assets/screenshots/ai-response.png)
*Live conversation showing intent understanding, contextual memory, and professional response generation.*

### 3. Google Sheets Ticket Management
![Google Sheets Tickets](assets/screenshots/google-sheets.png)
*Structured ticket records with category, priority, sentiment, assigned team, SLA deadline, and escalation status.*

---

## 🚀 Future Improvements

- [ ] **Migrate persistence to PostgreSQL / Supabase** — Sheets is ideal for demos and small teams, but relational storage unlocks joins, indexing, and volume
- [ ] **RAG over knowledge base** — vector-store retrieval so the agent answers from real product documentation instead of general reasoning
- [ ] **Multi-channel intake** — WhatsApp Business API, email (IMAP), Slack, and website widget feeding the same agent
- [ ] **Automated SLA breach alerts** — scheduled monitor that pings the assigned team before a deadline is missed
- [ ] **Human-in-the-loop handoff** — auto-route below a confidence threshold to a live agent with full context attached
- [ ] **Live analytics dashboard** — Looker Studio or Metabase layer on top of the ticket store
- [ ] **Multilingual support** — language detection with reply generation in the customer's language
- [ ] **Feedback loop** — capture actual CSAT scores and compare against predicted CSAT risk to tune the agent
- [ ] **Evaluation suite** — a labelled test set to measure classification and priority accuracy across prompt versions

---

## 🎓 Key Learnings

**Agent design ≠ prompt writing.** The hard part wasn't getting good text out of the model — it was constraining the output into fields a business system can actually consume. Enforcing a strict schema in the system prompt did more for reliability than any amount of prompt polish.

**Structured output is the real product.** A friendly reply is nice. A routed, prioritised, SLA-stamped ticket row is what saves a team money. I designed the schema before I designed the prompt.

**Memory changes the interaction fundamentally.** Without conversation memory the agent treats every message as a cold start and customers repeat themselves. Adding session context was the single largest jump in perceived quality.

**Confidence scoring makes autonomy safe.** Letting the agent state its own uncertainty turns "fully automated and occasionally wrong" into "automated with a defined escape hatch" — which is the difference between a demo and something a business will actually deploy.

**Pick the model for the job.** A Flash-class model handles classification and templated response generation at a fraction of the latency and cost of a frontier model. Over-specifying the LLM is a common and expensive mistake in automation work.

**Design for the person reading the output.** Choosing Google Sheets wasn't a technical shortcut — it meant a non-technical ops manager could use the system on day one with zero onboarding. Adoption beats elegance.

---

## 👤 About Me

**Mayank Gupta** — AI Automation Engineer specialising in n8n workflows, AI agents, and business process automation.

I build automation systems that replace repetitive operational work — support triage, lead qualification, data entry, and reporting — for D2C brands, SaaS teams, and agencies. My focus is on automations that produce **structured, actionable business data**, not just AI demos.

**What I build:**
- 🤖 AI agents and multi-step n8n workflows
- 📊 CRM, spreadsheet, and database automation pipelines
- 🎯 Lead qualification and enrichment systems
- 💬 Customer support and communication automation

**Currently:** Business Development Partner at Flowmingo AI · based in Punjab, India · available for freelance automation projects.

📫 **Let's connect**

[![LinkedIn](https://img.shields.io/badge/LinkedIn-Connect-0A66C2?style=for-the-badge&logo=linkedin&logoColor=white)](https://www.linkedin.com/in/mayank-gupta-india)
[![Email](https://img.shields.io/badge/Email-Reach%20out-EA4335?style=for-the-badge&logo=gmail&logoColor=white)](mailto:your.email@example.com)
[![GitHub](https://img.shields.io/badge/GitHub-Follow-181717?style=for-the-badge&logo=github&logoColor=white)](https://github.com/your-username)

---

<div align="center">

**If this project is useful to you, a ⭐ is appreciated.**

*Open to freelance automation work and collaboration.*

</div>
