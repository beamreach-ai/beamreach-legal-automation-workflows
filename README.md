# Beamreach Legal Automation Workflows

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

Open-source n8n workflow library for immigration law firms — AI-powered intake, Clio integration, and document automation.

> **Built by [Beamreach](https://beamreach.ai/immigration-law-automation.html)** — we implement these systems for law firms. [Book a call →](https://beamreach.ai/immigration-law-automation.html)

---

## What's in this repo

### Workflows

| Workflow | Description | n8n Marketplace |
|----------|-------------|-----------------|
| [Immigration Intake + Clio](n8n-workflows/immigration-intake/) | Webhook → GPT-4o visa classification → Clio contact + matter + checklist → email | [View →](https://n8n.io/workflows/) |

### Supporting resources

| Folder | Contents |
|--------|----------|
| [`/prompts`](prompts/) | Reusable AI prompts (visa classification, document extraction) |
| [`/config`](config/) | Tweakable settings (Clio practice area, SMTP from address, OpenAI model) |
| [`/templates`](templates/) | Static document checklist templates per visa type |
| [`/icons`](icons/) | Branding assets — replace with your firm's logo |
| [`/diagrams`](diagrams/) | Architecture diagrams for the workflows |

---

## Quick start: Immigration Intake workflow

### What it does

1. Receives a webhook POST from any intake form (Typeform, Jotform, custom) — accepts any fields
2. Sends all intake data to **GPT-4o**, which:
   - Classifies the appropriate US visa category (H-1B, EB-2 NIW, K-1, Asylum, etc.)
   - Extracts structured client info (name, email, phone, DOB, nationality)
   - Generates a case-specific document checklist (6-12 items, ordered by urgency)
3. Creates a **Clio Contact** for the prospective client
4. Opens a **Clio Matter** with the AI-generated description
5. Attaches a **Note** to the matter with the full AI summary and document checklist
6. Sends the client a **confirmation email** with their preliminary assessment and document list
7. Returns a JSON `200 OK` to the form

### Prerequisites

- n8n (cloud or self-hosted, v1.30+)
- OpenAI API key (GPT-4o)
- Clio account with API access
- SMTP credentials (or use n8n's built-in email)

### Installation

**1. Install the Clio community node**

In n8n → Settings → Community Nodes → Install → `@beamreach/n8n-nodes-clio`

**2. Import the workflow**

In n8n → Workflows → Import from file → select [n8n-workflows/immigration-intake/workflow.json](n8n-workflows/immigration-intake/workflow.json)

**3. Configure credentials**

| Credential | Where to create |
|------------|----------------|
| OpenAI API | n8n Credentials → New → OpenAI |
| Clio OAuth2 | n8n Credentials → New → Clio OAuth2 (see [@beamreach/n8n-nodes-clio setup](https://github.com/beamreach-ai/n8n-nodes-clio#setup)) |
| SMTP | n8n Credentials → New → SMTP |

**4. Update config nodes**

Inside the workflow, update the **Send Confirmation Email** node:
- `fromEmail` → your firm's email address
- `fromName` → your firm name

Optionally update [config/clio.json](config/clio.json) with your Clio practice area ID.

**5. Activate and test**

Activate the workflow. Send a test POST to the webhook URL:

```bash
curl -X POST https://your-n8n.com/webhook/immigration-intake \
  -H "Content-Type: application/json" \
  -d '{
    "full_name": "Maria Gonzalez",
    "email": "maria@example.com",
    "phone": "+1 415 555 0100",
    "nationality": "Mexican",
    "current_status": "F-1 OPT",
    "employer": "Acme Corp",
    "job_title": "Software Engineer",
    "salary": "$120,000",
    "message": "My OPT expires in 6 months. My employer wants to sponsor me."
  }'
```

You should receive a structured JSON response and see a new contact + matter in Clio.

---

## Customization

### Intake form fields

The webhook accepts **any JSON body** — no fixed schema. The workflow flattens all fields into
a key-value string and passes it to GPT-4o. Works with:

- Typeform (use the webhook integration)
- Jotform (webhook output)
- Gravity Forms (Zapier/webhook add-on)
- Custom HTML forms
- Direct API calls from your website

### AI prompt

Edit [prompts/visa-classification.md](prompts/visa-classification.md) to:
- Add new visa categories
- Adjust document checklist depth
- Change urgency rules
- Support additional jurisdictions

### Confirmation email

The HTML email template lives directly in the **Send Confirmation Email** node. Edit it to
match your firm's branding. Add your logo from the [icons/](icons/) folder.

### Clio settings

Edit [config/clio.json](config/clio.json) to set:
- `practiceAreaId` — auto-assign new matters to a practice area
- `noteSubject` — customize the note title
- `defaultMatterStatus` — `Pending`, `Open`, etc.

---

## Custom n8n nodes

This repo uses two companion npm packages:

| Package | Status | npm |
|---------|--------|-----|
| [`@beamreach/n8n-nodes-clio`](https://github.com/beamreach-ai/n8n-nodes-clio) | Available | [![npm](https://img.shields.io/npm/v/@beamreach/n8n-nodes-clio.svg)](https://www.npmjs.com/package/@beamreach/n8n-nodes-clio) |
| [`n8n-nodes-lawmatics`](https://github.com/beamreach/n8n-nodes-lawmatics) | Coming soon | — |

Both packages are MIT-licensed and can be installed directly from npm into any n8n instance.

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     Intake Flow                                  │
│                                                                  │
│  Webhook ──► Format ──► OpenAI ──► Parse ──► Clio Contact       │
│   (any form fields)    GPT-4o               ──► Clio Matter      │
│                        visa type                ──► Clio Note    │
│                        + checklist              ──► Email        │
│                        + urgency                ──► 200 OK       │
└─────────────────────────────────────────────────────────────────┘
```

All AI processing uses OpenAI's structured output (`response_format: json_object`) for reliable
parsing. The prompt is maintained separately in `/prompts/visa-classification.md`.

---

## Contributing

PRs welcome. To add a workflow:

1. Create a folder under `n8n-workflows/your-workflow-name/`
2. Add `workflow.json` (exported from n8n) and `README.md`
3. Update this README's workflow table

---

## License

MIT © [Beamreach](https://beamreach.ai)

---

## Professional setup

Want this running in your firm in a week? We handle:
- n8n deployment (cloud or self-hosted)
- Clio OAuth2 integration
- Custom intake form connection
- Email template with your branding
- Attorney review queue and escalation logic

[Talk to us →](https://beamreach.ai/immigration-law-automation.html)
