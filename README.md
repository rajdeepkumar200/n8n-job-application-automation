# n8n Job Application Automation

A set of **8 importable n8n workflows** that automate a truthful, ATS-friendly
job-application pipeline: resume intake → JD-tailored one-page resume → drift
guard (anti-fabrication) → job discovery → human-gated apply/outreach → reply
classification → interview prep → status notifications.

Each workflow ships with its AI-agent system prompts pre-filled. Credentials,
sheet IDs, and folder IDs are placeholders you connect/replace in your own n8n.

> **Ethics & ToS:** LinkedIn and Naukri prohibit automated logins/applications.
> This design avoids that: job discovery uses a sanctioned Indeed connector plus
> a Gmail watcher on **your own** job-alert inbox, and sending an application
> **always** passes through a human-approval gate. See `docs/SETUP.md`.

---

## Workflows

| # | File | Purpose | Trigger |
|---|---|---|---|
| 1 | `workflows/1-resume-intake.json` | Extract a structured skills/experience inventory from your real resume → `MasterProfile` | Form upload |
| 2 | `workflows/2-jd-to-tailored-resume.json` | Paste a JD → truthful, one-page, ATS-friendly resume (+ATS score), render PDF, log to `Applications` | Form |
| 3 | `workflows/3-drift-guard.json` | Fact-check the tailored resume against your real inventory; flags any unsupported claim | Sub-workflow (called by W2) |
| 4 | `workflows/4-job-discovery.json` | Pull Indeed + your Gmail job-alerts, rank by genuine fit, send a shortlist | Schedule (daily 8am) |
| 5 | `workflows/5-apply-outreach.json` | Draft a cold email/cover letter, **human-approve**, send with resume, update tracker | Sheet approval |
| 6 | `workflows/6-reply-watcher.json` | Classify incoming reply emails, update tracker, branch to interview prep | Gmail (every 15m) |
| 7 | `workflows/7-interview-prep.json` | Generate role-specific Q&A grounded in your resume | Sub-workflow (called by W6) |
| 8 | `workflows/8-status-notifier.json` | Shared notifier — routes events to Telegram/Email | Sub-workflow (called by all) |

### Dependencies between workflows

```
W1 ──> MasterProfile sheet ──> W2, W4
W2 ──calls──> W3 (drift guard) ──> W8 (notify)
W4 ──calls──> W8
W5 ──calls──> W8
W6 ──calls──> W7 (interview prep) ──> W8
                └─────────────────────> W8
```

W3, W7, W8 are **sub-workflows** invoked via the *Execute Workflow* node. Import
them first so the callers can resolve them.

---

## Quick start

1. **Import order:** W8, W3, W7 first (sub-workflows), then W1, W2, W4, W5, W6.
   In n8n: *Workflows → Import from File* (or `n8n import:workflow --input=<file>`).
2. **Create the data store:** one Google Sheet with two tabs, `MasterProfile`
   and `Applications` — see `docs/GOOGLE_SHEETS_SCHEMA.md`.
3. **Connect credentials** (Anthropic, Google Sheets/Drive/Gmail, optional
   Telegram) and **replace placeholders** — see `docs/SETUP.md`.
4. **Run W1 once** to populate `MasterProfile`, then use W2 to tailor resumes.

The smallest useful slice is **W1 + W2 + W3** ("paste a JD, get a truthful
one-page resume"). Everything else expands outward from there.

---

## Requirements

- n8n (cloud or self-hosted), with the **LangChain / AI** nodes available
- Node.js `>=22.22` if self-hosting via npm
- An **Anthropic** API key (workflows use `claude-3-5-sonnet`); swap the model
  node for OpenAI if you prefer
- A **Google** account (Sheets, Drive, Gmail nodes)
- *(optional)* a **Telegram** bot for push notifications
- *(for W2 PDF)* a Gotenberg instance, or substitute PDF.co / `wkhtmltopdf` —
  see `docs/SETUP.md`

All AI nodes use the `chainLlm` + `outputParserStructured` pattern so the model
returns strict JSON rather than free-form reasoning.

See **`docs/SETUP.md`** for full step-by-step configuration.
