# Google Sheets schema

Create **one** Google Sheet (note its ID from the URL — the long string between
`/d/` and `/edit`) with **two tabs**: `MasterProfile` and `Applications`.

Put the exact column headers below in **row 1** of each tab (the workflows map to
these header names).

---

## Tab: `MasterProfile`

The single source of truth about your real background. Written by **Workflow 1**,
read by **Workflow 2** and **Workflow 4**. Never auto-edited after intake — only
you change it.

| column | type | notes |
|---|---|---|
| `timestamp` | datetime | when intake ran (auto) |
| `target_roles` | text | roles you're targeting (from the intake form) |
| `target_city` | text | preferred location |
| `salary_range` | text | desired range |
| `master_resume` | long text | full plain text of your real resume |
| `skills_json` | long text (JSON) | structured inventory: `{skills, tools, projects:[{name,description,metrics}], experience:[{role,company,dates,bullets}]}` |
| `resume_master_link` | text | Google Drive link to the uploaded original |

> Workflow 4 also reads optional `desired_roles` / `desired_city` — if you prefer,
> reuse `target_roles` / `target_city` and adjust the node mappings, or add those
> columns too. The discovery Code node already falls back across both names.

---

## Tab: `Applications`

The tracker. Appended by **Workflow 2**, updated by **Workflow 5** (on send) and
**Workflow 6** (on reply).

| column | type | notes |
|---|---|---|
| `timestamp` | datetime | auto, when the row was created |
| `company` | text | |
| `role` | text | |
| `jd_text` | long text | full JD, for reference |
| `resume_version_link` | text | Google Drive link to the PDF used |
| `ats_score` | number | from the tailor agent, 0–100 |
| `date_applied` | date | set by W5 when the application is sent |
| `channel` | text | email / linkedin / naukri / referral |
| `status` | text | applied / viewed / interview / assessment / rejected / offer |
| `last_reply_date` | date | updated by the reply watcher (W6) |
| `notes` | text | free text / reply summary |
| `approve` | boolean | set `TRUE` to let **W5** send the application for this row |

> The `approve` column is the human-approval switch for Workflow 5. Leave it
> blank/FALSE until you've reviewed the tailored resume; set it `TRUE` to release
> the application.
