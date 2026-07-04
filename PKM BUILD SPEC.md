# PKM System — Claude Code Build Spec

Feed this to Claude Code inside VS Code. It is written as instructions to Claude Code, staged so nothing runs ahead of its dependencies. Do not skip stages.

---

## 0. Context for Claude Code

You are building a file-based personal knowledge management system. Core = one Obsidian vault, plain Markdown. Three top-level systems: `ATS/`, `Promitor/`, `Personal/`. An intelligence layer calls the Anthropic API to classify and route captured notes. An always-on box runs scheduled maintenance. Full device sync via Syncthing; Git for versioning and sandboxing.

Design constraints (non-negotiable, ADHD-driven):
- Mechanical enforcement over discipline. Validation runs automatically; the user never relies on remembering.
- No entry-time friction. Captures land messy and get normalised later by a scheduled pass — never block a capture for being malformed.
- Cost-minimising by default. Cheap local rules first, cheapest adequate model second, expensive models only when genuinely needed.
- Small. Do not add features beyond this spec. If you think of one, note it in `IDEAS.md`, do not build it.

---

## 1. Repository structure

```
pkm-system/
├── vault/                    # the Obsidian vault (synced by Syncthing)
│   ├── ATS/                  # PARA-per-system
│   │   ├── 1-Projects/
│   │   ├── 2-Areas/
│   │   ├── 3-Resources/
│   │   └── 4-Archive/
│   ├── Promitor/             # same PARA structure
│   ├── Personal/             # same PARA structure
│   ├── _inbox/               # raw captures land here
│   ├── _review/              # low-confidence items for human review
│   └── _templates/           # document templates, first-class
├── engine/                   # the intelligence + maintenance layer
│   ├── classify.py           # calls Anthropic API, routes inbox items
│   ├── normalise.py          # renames violations, fixes links
│   ├── linkcheck.py          # finds broken internal links
│   ├── credit_tracker.py     # logs token use + cost
│   ├── rules.py              # local pre-filter rules (regex/keyword/filetype)
│   └── config.yaml           # taxonomy, naming rules, model tiers, thresholds
├── dashboard/                # the command centre
│   └── (see Stage 6)
├── logs/
│   └── usage.jsonl           # append-only token/cost log
├── tests/
├── .githooks/
├── CLAUDE.md                 # project instructions Claude Code must obey
├── IDEAS.md                  # parking lot for deferred features
└── README.md
```

Claude Code: create this tree in Stage 2. Do not populate vault content — the user migrates that later.

---

## 2. Stage 1 — Scaffold + Git + sandbox workflow

1. `git init`. Create the tree above with `.gitkeep` in empty dirs.
2. Write `.gitignore`: exclude `logs/`, `.env`, `__pycache__/`, `.obsidian/workspace*`.
3. Establish branch workflow and document it in `README.md`:
   - `main` = live daily-driver.
   - Any structural experiment = feature branch (`git checkout -b experiment-<name>`).
   - Merge only after the user has lived with it. Delete branch if rejected.
   - For destructive experiments, instruct the user to clone the vault to a throwaway folder rather than branch.
4. Write `CLAUDE.md` (see Stage 3) — this must exist before any code that touches the vault.

## 3. Stage 2 — CLAUDE.md enforcement file

This is the drift-prevention core. It must contain, explicitly:
- The three-system + PARA structure, stated as the only valid file locations.
- Naming convention: ISO 8601 dates (`YYYY-MM-DD`), lowercase-hyphenated slugs, format `YYYY-MM-DD_slug.md` for dated notes, `slug.md` for evergreen. State it as a hard rule with examples of valid and invalid.
- Template conformance: every note type maps to a template in `_templates/`. List them.
- The rule that captures are never blocked — malformed input is normalised post-hoc, not rejected.
- Instruction that Claude Code must never invent folders, tags, or note types outside `config.yaml`.
- Instruction to log deferred feature ideas to `IDEAS.md`, never build them inline.

## 4. Stage 3 — Config as single source of truth

Write `engine/config.yaml` holding: the taxonomy (systems, PARA folders), naming rules (regex patterns), model tiers (which model for which job), confidence threshold for auto-file vs review queue, and per-model pricing (input/output per Mtok) for the credit tracker. Everything else reads from here — no hardcoded values elsewhere. When pricing changes, the user edits one file.

## 5. Stage 4 — The engine

Build in this order, each independently testable:

1. **`rules.py`** — local pre-filter. Zero API cost. Regex/keyword/filetype rules that resolve obvious cases (e.g. anything mentioning a known ATS project → `ATS/`). Return a destination or `None` (escalate).
2. **`credit_tracker.py`** — wrap every API call. Log to `logs/usage.jsonl`: timestamp, operation, model, input tokens, output tokens, computed cost from `config.yaml`. This wrapper is used by everything that calls the API, so build it before `classify.py`.
3. **`classify.py`** — the router. Flow per inbox item:
   - Run `rules.py` first. If it resolves, file it, log a zero-cost decision, done.
   - Else call the API through the credit tracker wrapper. Use prompt caching for the taxonomy/routing context (it's fixed — don't resend instruction tokens every call). Use the cheapest tier model (Haiku) for folder-routing.
   - If confidence ≥ threshold → file to destination. Else → move to `_review/` with a note on why.
   - Never delete the original capture until the copy is confirmed filed.
4. **`normalise.py`** — scheduled pass. Scan vault, detect naming violations against `config.yaml` regex, rename to conform, update all internal links that referenced the old name. Output a change report to `logs/`. This is batch, not per-capture.
5. **`linkcheck.py`** — report broken internal links. Report only; do not auto-delete.

## 6. Stage 5 — Validation + hooks + scheduled jobs

1. **Tests** (`tests/`): naming-convention linter, template-conformance check, link integrity. These are the "treat the PKM as a software project" robustness layer.
2. **Pre-commit hook** (`.githooks/`): run the linters on staged vault files. Per the user's answer, this **warns and auto-fixes** where possible (runs `normalise.py` on staged files) rather than hard-failing. Only hard-fail on genuinely unresolvable structural corruption.
3. **Scheduled jobs on the always-on box** (cron or systemd timer — the box picks up whatever load it can):
   - `classify.py` on `_inbox/` — frequent (e.g. every 15 min).
   - `normalise.py` — daily.
   - `linkcheck.py` — daily, feeds the dashboard.
   - `credit_tracker` aggregation — hourly rollup for the dashboard.
   Document exact schedules in `README.md`. Make them idempotent — a missed run self-heals on the next.

## 7. Stage 6 — Dashboard with live credit tracker

Single command-centre page. Minimum viable content:
- **Live API credit tracker**: current month spend, running total, cost-per-operation breakdown (which operations cost most — this drives future optimisation), token volume. Reads from `logs/usage.jsonl`.
- **Review queue count**: how many items sit in `_review/` awaiting a human call.
- **Broken-link count** from `linkcheck.py`.
- Links into the three systems.

Implementation: simplest thing that renders on Mac + iPhone. An Obsidian dashboard note using Dataview for vault-side counts, plus a small locally-served HTML/JSON view for the credit data (the always-on box serves it, or write the rollup as a Markdown table into a dashboard note the vault already syncs). Prefer the latter — one less moving service, and it rides the sync you already have. Confirm approach with the user before building this stage.

## 8. Cost-optimisation (build into the engine, not bolted on)

- Model tiering via `config.yaml` — Haiku for routing, escalate only for genuine reasoning.
- Local rules resolve the obvious cases at zero cost before any call.
- Prompt caching on the fixed taxonomy context.
- Batch scheduled processing, not per-capture wake-ups.
- The usage log is the optimisation instrument: it shows which operations to attack next.

## 9. Explicitly deferred (do not build in v1)

Log these in `IDEAS.md`: weekly digest (rejected), cross-system link suggestions (v2), duplicate detection, auto-tagging beyond folder routing, priority/sentiment scoring, graph dashboards.

## 10. Build order summary

Scaffold+Git → CLAUDE.md → config.yaml → rules → credit_tracker → classify → normalise → linkcheck → tests+hooks → scheduled jobs → dashboard. Each stage committed separately. Nothing merged to `main` until tested.
