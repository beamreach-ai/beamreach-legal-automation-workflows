# Immigration Intake — AI Visa Classification + Clio

Automate your immigration intake: receive any form submission, classify the US visa type with GPT-4o, and create a Clio contact + matter + document checklist in one workflow.

## Workflow overview

```
POST /webhook/immigration-intake
         │
         ▼
┌─────────────────┐
│  Format for AI  │  Flatten any form fields into a key-value string
└────────┬────────┘
         │
         ▼
┌─────────────────────┐
│  OpenAI GPT-4o      │  Classify visa · Extract client info · Generate checklist
│  (structured JSON)  │
└────────┬────────────┘
         │
         ▼
┌─────────────────┐
│  Parse Response │  Flatten AI JSON into n8n fields
└────────┬────────┘
         │
         ▼
┌─────────────────┐     ┌─────────────────┐     ┌──────────────────┐
│  Clio Contact   │ ──► │  Clio Matter    │ ──► │  Clio Note       │
│  (create)       │     │  (open)         │     │  (checklist)     │
└─────────────────┘     └─────────────────┘     └────────┬─────────┘
                                                          │
                                                          ▼
                                               ┌──────────────────┐
                                               │  Confirm Email   │
                                               │  to client       │
                                               └────────┬─────────┘
                                                        │
                                                        ▼
                                               ┌──────────────────┐
                                               │  200 OK response │
                                               └──────────────────┘
```

## Import

1. Download [workflow.json](workflow.json)
2. In n8n → Workflows → Import from file
3. The workflow imports in **inactive** state — configure credentials before activating

## Requirements

| Dependency | Version |
|------------|---------|
| n8n | 1.30+ |
| OpenAI API | GPT-4o |

**No community nodes required.** Works on n8n Cloud and self-hosted.

## Credentials to configure

| Node | Credential type | Notes |
|------|----------------|-------|
| OpenAI — Classify Visa | OpenAI API | Standard API key |
| Clio — Create Contact | Header Auth | See Clio setup below |
| Clio — Open Matter | Header Auth | Same credential |
| Clio — Add Checklist Note | Header Auth | Same credential |
| Send Confirmation Email | SMTP | Any SMTP server |

### Clio API Token setup

1. Log into Clio → **Settings** → **API Keys** → **Generate Token**
2. Copy the token
3. In n8n → **Credentials** → **New** → search **Header Auth**
4. Set:
   - **Name**: `Authorization`
   - **Value**: `Bearer <paste-your-token-here>`
5. Save as **"Clio API Token"** and assign it to all three Clio nodes

## Configuration

After importing, update these values in the workflow:

**Send Confirmation Email node:**
- `fromEmail` → your firm's sending address
- `fromName` → your firm name

**Optional — Clio practice area:**
In the **Clio — Open Matter** node, set `practiceAreaId` in Additional Fields
to auto-assign new matters to an immigration practice area. Find the ID in
Clio → Settings → Practice Areas.

## Webhook input

The webhook accepts **any JSON body** — no fixed schema required.
Send whatever fields your intake form collects:

```json
{
  "full_name": "Jane Smith",
  "email": "jane@example.com",
  "phone": "+1 212 555 0100",
  "nationality": "Indian",
  "current_visa": "H-4",
  "employer": "Tech Corp",
  "job_title": "Data Scientist",
  "annual_salary": "$130,000",
  "degree": "M.S. Computer Science",
  "situation": "My H-4 EAD is expiring. My company wants to file for my green card."
}
```

GPT-4o reads all fields and infers the visa pathway regardless of field names.

## AI output schema

The OpenAI node returns (after parsing):

```json
{
  "client": {
    "full_name": "Jane Smith",
    "email": "jane@example.com",
    "phone": "+1 212 555 0100",
    "country_of_citizenship": "India",
    "current_immigration_status": "H-4 EAD"
  },
  "matter": {
    "description": "EB-2 NIW — Jane Smith",
    "status": "Pending"
  },
  "classification": {
    "visa_category": "EB-2",
    "visa_subcategory": "NIW",
    "confidence": "HIGH",
    "reasoning": "Applicant holds M.S. degree, works in STEM, employer willing to sponsor. NIW may be faster than PERM-based EB-2 given India priority date backlog.",
    "alternative_pathways": ["EB-3", "H-1B (if currently on H-4 EAD and employer willing to file cap-exempt)"]
  },
  "document_checklist": [
    "Signed retainer agreement",
    "Copy of passport (all pages)",
    "H-4 visa stamp and I-94 record",
    "H-4 EAD card (front and back)",
    "Current résumé / CV",
    "M.S. diploma and transcripts",
    "Published papers, patents, or evidence of specialized contributions",
    "Letters of recommendation (3-5 from independent experts in the field)",
    "Employer support letter",
    "Recent pay stubs (last 3 months)"
  ],
  "urgency": "EXPEDITE",
  "notes": "H-4 EAD expiration creates employment authorization risk. Check current India EB-2 priority date before advising on timeline."
}
```

## Customizing the AI prompt

The system prompt is embedded in the **OpenAI — Classify Visa** HTTP Request node.
The canonical, editable version is at [../../prompts/visa-classification.md](../../prompts/visa-classification.md).

To update: copy the system prompt from that file and paste it into the node's `jsonBody` → `messages[0].content`.

## Production considerations

- **Deduplication**: The workflow always creates a new Clio contact. For production, add a
  "Search Contact" step before "Create Contact" and skip creation if the email already exists.
- **Error handling**: Add an Error Trigger workflow to catch failures and alert your team.
- **Rate limits**: OpenAI GPT-4o has per-minute token limits. For high volume, add a Wait node
  or use the Batch Trigger pattern.
- **PII**: Intake data passes through OpenAI. Review your OpenAI data processing agreement
  and firm policies before sending client PII.

---

Built by [Beamreach](https://beamreach.ai/immigration-law-automation.html)
