# Strata Enquiry Classifier — AI Automation

A working AI-powered tool that classifies incoming client enquiries for a strata management firm, routes them to the right staff member, and drafts a suggested response — automatically.

Built for the Strata Management Consultants AI Developer assessment by **Kimberly Java**.

---

## Live Demo

Open `demo/index.html` in your browser. Enter your Groq API key and paste any client enquiry to see it classified instantly.

> You will need a free Groq API key from [console.groq.com](https://console.groq.com)

---

## What It Does

When a client submits an enquiry via the Tally web form:

1. **Tally** sends the form submission to an n8n webhook
2. **n8n** extracts and validates the fields (name, email, phone, role, subject, message)
3. **Groq** (llama-3.3-70b-versatile) classifies the enquiry — type, urgency, confidence score, assigned staff member, and draft response
4. **Brevo** sends an email directly to the assigned staff member with the AI summary and suggested response ready to copy
5. **Google Sheets** logs every enquiry for audit and reporting

Staff open their email, read the 2-sentence AI summary, copy the suggested response, and reply. No manual triage required.

---

## Enquiry Types Handled

| Type | Routed To |
|---|---|
| Maintenance Request | James Okoye (Maintenance Coordinator) |
| Levy / Fees | Priya Sharma (Accounts Officer) |
| OC Certificate | Sarah Mitchell (OC Manager) |
| AGM / Meetings | Sarah Mitchell (OC Manager) |
| Insurance | Linda Cross (Compliance Officer) |
| By-law / Compliance | Linda Cross (Compliance Officer) |
| New Client / Proposal | Marcus Webb (Business Development) |
| Legal Threat | Linda Cross (Compliance Officer) |
| General Enquiry | Sarah Mitchell (OC Manager) |

---

## Bonus Features

### Confidence Scoring
Every classification includes a `confidence` score (0–1) and a plain-English `confidence_reason` explaining why the AI is certain or uncertain. Low confidence triggers a clarification request in the suggested response.

### Prompt Engineering
The system prompt is fully visible in the demo UI — click "View system prompt & design notes" after any classification. Key design decisions:

- **Explicit categories** — the model is given exact enquiry types to choose from, not open-ended classification
- **Team roster embedded in prompt** — names, roles, and email addresses are in the system prompt so the model can assign directly
- **Business rules as explicit rules** — "any mention of VCAT = assign to Linda Cross" removes ambiguity
- **JSON output mode** — `response_format: json_object` forces clean parseable output every time
- **Edge case handling** — vague inputs return confidence < 0.4 and a clarification request rather than a confident wrong answer

### Error Handling
- Empty or missing message field → workflow exits with error notification
- Vague/nonsensical input → classified as General Enquiry, confidence < 0.4, suggested response asks for clarification
- API failures → caught and displayed clearly in the UI

### Automation Potential
Every classification includes an `automation_potential` field describing how that enquiry type could be automated further. The full n8n workflow (see `n8n/workflow.json`) demonstrates this — a legal threat automatically triggers a second escalation email on top of the standard routing.

---

## Tools Used

| Tool | Purpose |
|---|---|
| [Tally](https://tally.so) | Client-facing enquiry form with webhook support |
| [n8n](https://n8n.io) | Workflow automation — orchestrates the full pipeline |
| [Groq](https://groq.com) | LLM inference (llama-3.3-70b-versatile) — fast, free tier |
| [Brevo](https://brevo.com) | Transactional email — routes classified enquiry to staff |
| [Google Sheets](https://sheets.google.com) | Audit log of all enquiries and classifications |

---

## Setup Instructions

### Demo UI (quickest)
1. Open `demo/index.html` in any browser
2. Enter your Groq API key (free at [console.groq.com](https://console.groq.com))
3. Paste an enquiry or click an example chip
4. Click "Analyse enquiry"

### Full n8n Workflow
1. Import `n8n/workflow.json` into your n8n instance
2. Create an **HTTP Header Auth** credential named `Groq API Key` — header: `Authorization`, value: `Bearer YOUR_KEY`
3. Connect your Google account in the Google Sheets node
4. Create a Google Sheet named **STRATA ENQUIRY LOG** with headers matching the field list in the workflow
5. Update the Brevo node with your verified sender email and API key
6. Create a Tally form with fields: Name, Email, Phone, I am a (dropdown), Subject, Message
7. Set the Tally webhook URL to your n8n production webhook URL
8. Activate the workflow

---

## Repository Structure

```
strata-enquiry-classifier/
├── README.md
├── demo/
│   └── index.html          — standalone browser demo (Groq-powered)
├── n8n/
│   └── workflow.json       — importable n8n workflow
└── prompts/
    └── system-prompt.md    — full system prompt with design notes
```

---

## Design Decisions

**Why n8n over custom code?**
n8n lets me build, debug, and iterate the pipeline visually without managing infrastructure. Each node is independently testable, which made debugging the Tally → Groq → Brevo chain much faster than it would have been in a custom script.

**Why Groq over OpenAI?**
Speed and cost. Groq's inference is significantly faster than OpenAI for this use case (sub-second responses), and the free tier is sufficient for a demonstration. The llama-3.3-70b-versatile model produces classification quality comparable to GPT-4o for structured JSON tasks with a good system prompt.

**Why email over a dashboard?**
The task says staff currently read emails manually. Meeting them in their existing workflow (email inbox) means zero behaviour change required — they just get a better email. A dashboard requires staff to actively check another tool, which reduces adoption.

**Why Tally over a custom form?**
Tally has native webhook support, is free, and can be embedded on any website. It produces a clean structured payload that n8n can parse reliably. For the real client, this could be replaced with a WordPress-embedded form (WPForms + webhook) without changing the n8n workflow at all.

---

Built by Kimberly Java · AI Automation Specialist · [Portfolio](https://kimmyjava.github.io/portfolio/)
