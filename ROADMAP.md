# TriOS Roadmap — Automation Tracking, Search, and Scope Growth

Written 2026-07-04, after v2 shipped. Three pillars, phased so nothing runs ahead of its dependencies (same discipline as the PKM build spec).

---

## Where we are

| Layer | Status |
|---|---|
| Dashboard v2 (Home, Dump w/ auto-sort, Tasks, Work+Stu, Learn, System, Project mode) | **Live** — `trios-dashboard-sync.html` |
| Supabase backend (tasks, ideas, learning, inbox, settings; RLS; Sydney; free tier) | **Live** — project `trios` |
| Calendar edge function (iCal proxy) + rule-based "Suggest my day" | **Live** |
| Hosting for phone/TV | **Not done** — next infrastructure step |
| PKM vault + Python engine (classify/normalise/linkcheck) | **Spec'd, not built** — blocked on your HUMAN TASKS items |
| AI features (capture classification, AI day planner) | **Parked** — blocked on Anthropic API key |

---

## Pillar 1 — See and track every automated task

**Goal:** one Automations tab where every background job in your life-system is visible: what it is, when it last ran, whether it worked, what it cost.

**Design — two small Supabase tables:**

- `automations` — the registry. One row per job: name, kind (edge function / NAS cron / cloud agent), schedule ("every 15 min", "daily 02:00"), status (active/paused), description.
- `automation_runs` — the log. One row per execution: started, finished, ok/error, items processed, tokens + cost (for AI jobs), short log excerpt.

Every job — current and future — writes a run row when it starts and finishes (a 5-line helper each script calls). This is the credit-tracker idea from your PKM spec, generalised to *all* automation, not just API spend.

**Dashboard Automations tab shows:** each job with a status dot (green = last run ok, amber = overdue, red = failed), last/next run, a small success history, month-to-date cost tile. Failures surface as a Home notice card — same "query me when there's an issue" pattern as the capture review queue. Projectable to the monitor like any other board.

**What gets registered over time:**
1. Calendar fetch (already running — first registry entry)
2. NAS engine jobs when built: classify inbox (15 min), normalise (daily), linkcheck (daily)
3. AI day planner + AI capture classifier (when API key exists)
4. Any scheduled Claude cloud agents / Zapier–Make scenarios we add later

---

## Pillar 2 — Search everything

Staged, cheapest-first:

- **S1 — Global search bar (dashboard):** searches tasks, ideas, learning items, inbox, and task notes via Postgres full-text. Keyboard shortcut, results grouped by type, jump straight to the item. *No new infrastructure — can build now.*
- **S2 — Artifacts & documents registry:** a new `artifacts` table — name, type (doc / link / file / claude-artifact / drive file), location/URL, area, tags. You register the things you keep losing: specs, backups, dashboards, key Drive docs, artifact URLs. Search covers it; the System tab lists it. *Can build now; becomes the seed of your future wiki.*
- **S3 — Vault full-text:** once the PKM vault exists, the engine indexes every markdown note (title, headings, body) into the same search. Results deep-link to the note (Obsidian URI). *Blocked on vault.*
- **S4 — Semantic search (optional):** pgvector embeddings in Supabase so "that note about contractor budgets" works even when the words don't match. Only if S3 proves insufficient — measure first, don't assume.

---

## Pillar 3 — Add or expand scope, safely

**The rule (yours, from the spec):** ideas get parked, not built. Scope grows through a gate, not by accretion.

**The gate:**
1. Everything enters via the **Ideas wall** (or FEEDBACK.md / chat)
2. When an idea earns promotion, we scope it in one sitting: what table(s), what tab, what automation, what it must NOT do
3. Build it as an addition (new tab/module), live with it a week, keep or kill
4. Killed features leave cleanly — modules don't tangle

**Make the structure data, not code:** currently ATS/Promitor/Personal and "Stu" are hardcoded. A planned refactor moves these to config:
- `domains` — add a fourth life area (or rename one) without touching code
- `people` — Stu generalised: a waiting-on/raise-with board for any person (a second client, an accountant, a VA)

**Already-queued expansions (in priority order as of today):**
1. Wiki / templates / tools compository — lives in the vault (Phase 3)
2. Finances glance — Promitor cash/invoice summary from Zoho Books
3. Routines/habits strip on Home
4. Weekly review flow (look back over done + automation log + spend)

---

## Phases

| Phase | What | Needs from you |
|---|---|---|
| **1. Now** | Host dashboard (GitHub Pages) → phone + TV bookmarks. Global search S1. Artifacts registry S2. Automations tables + tab (calendar job as first entry). | GitHub account (HUMAN TASKS item D) |
| **2. Engine** | PKM vault scaffold + Python engine per spec, every job logging to `automation_runs`. AI capture classifier + AI day planner. Cost tile live. | Anthropic API key, NAS decision, Obsidian/Syncthing setup |
| **3. Knowledge** | Vault full-text search in dashboard. Wiki/templates compository in vault, surfaced in a dashboard tab. | Notion export/migration choices |
| **4. Optional** | Semantic search. `domains`/`people` config refactor. Drive/Gmail/Todoist integrations. Weekly digest. | Only appetite |

**Standing dependencies checklist (all from your HUMAN TASKS doc):** GitHub account/repo → unlocks hosting + versioning. Anthropic API key + small credit → unlocks all AI features. NAS confirmed → unlocks scheduled engine. Obsidian + Syncthing → unlocks vault.

---

*Change requests: small stuff in chat, batches in FEEDBACK.md. This file is the scope contract — if a build request isn't on here, it goes through the Ideas gate first.*
