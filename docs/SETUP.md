# Setup guide

Step-by-step configuration after importing the workflows. Estimated time:
20–30 min.

---

## 1. Import the workflows

Import in this order so sub-workflow references resolve:

1. `workflows/8-status-notifier.json`
2. `workflows/3-drift-guard.json`
3. `workflows/7-interview-prep.json`
4. `workflows/1-resume-intake.json`
5. `workflows/2-jd-to-tailored-resume.json`
6. `workflows/4-job-discovery.json`
7. `workflows/5-apply-outreach.json`
8. `workflows/6-reply-watcher.json`

**UI:** *Workflows → ⋯ → Import from File*.
**CLI (self-hosted):** `n8n import:workflow --input=workflows/8-status-notifier.json`

Each workflow carries a stable `id` (`job-automation-w1` … `w8`). The *Execute
Workflow* nodes reference callees by these ids, so importing with these files
keeps the links intact. If your n8n assigns new ids on import, re-open each
caller and re-select the target workflow in its *Execute Workflow* node
(see §6).

---

## 2. Connect credentials

In n8n *Credentials*, create and then select these on the relevant nodes:

| Credential | Used by | Notes |
|---|---|---|
| **Anthropic API** | all AI nodes | name it `Anthropic account` (matches the JSON) or re-pick it on each `Anthropic Chat Model` node |
| **Google Sheets OAuth2** | W1, W2, W4, W5, W6 | |
| **Google Sheets Trigger OAuth2** | W5 | the polling trigger uses a separate cred type |
| **Google Drive OAuth2** | W1, W2 | resume storage |
| **Gmail OAuth2** | W4, W5, W6 | reading job alerts, sending applications, watching replies |
| **Telegram API** | W8 | optional; remove the node or swap for Gmail if unused |

> The JSON references credentials by display name (e.g. `Anthropic account`). If
> your names differ, just open the flagged nodes and pick the right credential —
> n8n shows a red banner on any node with an unresolved credential.

---

## 3. Replace placeholders

Search-and-replace these across the imported workflows (or edit per node):

| Placeholder | Where | Replace with |
|---|---|---|
| `YOUR_GOOGLE_SHEET_ID` | W1, W2, W4, W5, W6 | your Sheet's ID (from its URL) |
| `YOUR_RESUMES_MASTER_FOLDER_ID` | W1 | Drive folder for original resumes (`/Resumes/Master/`) |
| `YOUR_RESUMES_FOLDER_ID` | W2 | Drive folder for tailored PDFs (`/Resumes/`) |
| `YOUR_JOB_ALERTS_LABEL_ID` | W4 | Gmail label id holding your job-alert emails |
| `YOUR_INDEED_OR_JOB_API_ENDPOINT` | W4 | your sanctioned Indeed connector / job-search API |
| `YOUR_TELEGRAM_CHAT_ID` | W8 | your Telegram chat id |

Create the two sheet tabs first — see `GOOGLE_SHEETS_SCHEMA.md`.

---

## 4. PDF rendering (Workflow 2)

W2 renders the tailored resume to HTML (Code node) then POSTs it to **Gotenberg**
at `http://gotenberg:3000/forms/chromium/convert/html`. Options:

- **Gotenberg** (recommended self-host): `docker run --rm -p 3000:3000 gotenberg/gotenberg:8`
  then point the HTTP node at the reachable host/port.
- **PDF.co / other HTML→PDF API:** swap the HTTP node's URL/auth for that API.
- **`wkhtmltopdf`:** replace the HTTP node with an *Execute Command* node
  (only on self-hosted n8n where the binary is installed).

If you only want the structured resume JSON (no PDF), you can disable the PDF +
Drive nodes; the `Applications` row still logs the ATS score.

---

## 5. AI model

All AI nodes use `claude-3-5-sonnet-20241022` via the **Anthropic Chat Model**
node at `temperature: 0` (deterministic) — except the outreach/prep drafts which
use a small non-zero temperature. To use OpenAI instead, replace each
`Anthropic Chat Model` node with an *OpenAI Chat Model* node and reconnect its
`ai_languageModel` output to the same Chain node.

---

## 6. Wire up the sub-workflow calls

These *Execute Workflow* nodes call other workflows by id:

- **W2** → `Call Drift Guard` → W3 (`job-automation-w3`)
- **W2 / W4 / W5 / W6 / W7** → `Notify (W8)` → W8 (`job-automation-w8`)
- **W6** → `Interview Prep (W7)` → W7 (`job-automation-w7`)

If any shows "workflow not found", open it and re-select the target from the
dropdown (n8n keys the link to the local workflow id, which can change on
import).

---

## 7. First run

1. **Activate W8, W3, W7** (sub-workflows can stay inactive; they run when
   called — but the callers must be able to find them).
2. Open **W1**, click the form URL, upload your real resume + targets → this
   fills `MasterProfile`.
3. Open **W2**, submit a JD + company + role → review the tailored resume; if the
   drift guard flags claims you'll get a notification instead of a finalized PDF.
4. Turn on **W4** (schedule) and **W6** (Gmail poll) once you trust the rest.
5. Keep the **W5 human-approval gate** on until you trust the output.

---

## Security notes

- Don't commit real credentials. n8n stores them encrypted in its own DB; this
  repo only contains placeholders.
- The reply watcher and job-alert reader only **read your own inbox** — they do
  not scrape third-party sites.
- Review every outgoing email (W5) before approving, at least initially.
