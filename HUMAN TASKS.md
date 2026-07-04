# Non-Claude Tasks — What You Do Yourself

Claude Code builds the software. These are the things it can't: accounts, devices, physical setup, and decisions only you can make. Ordered so nothing blocks a later step.

## A. Before Claude Code starts

1. **Install the toolchain on your Mac**: VS Code, Claude Code, Git, Python 3.11+, Obsidian. (Node too if the dashboard ends up needing it — Claude Code will tell you.)
2. **Get an Anthropic API key** — console.anthropic.com → API keys. Put a small credit balance on it and set a billing alert. This key powers the classification layer; it is separate from any Claude Code / Claude.ai subscription.
3. **Store the key as an environment variable** (`ANTHROPIC_API_KEY`), never in the repo. Claude Code's spec assumes this.
4. **Decide the always-on box now**: the UGREEN DH2300 NAS is the recommended candidate over a Windows box — lower power, purpose-built to stay on. Confirm which you're using; it changes how the scheduled jobs get installed (systemd/cron on the NAS vs Task Scheduler on Windows).

## B. Sync setup (you, on each device)

5. **Install Syncthing** on: the always-on box, your Mac, and (for mobile) via a compatible client. Obsidian's own paid Sync is the friction-free alternative if Syncthing proves fiddly — decide based on tolerance for setup vs a few dollars a month.
6. **Pair the devices** and set the `vault/` folder to sync. Verify a test file propagates all three ways before trusting it.
7. **Confirm the sync-conflict behaviour** — create the same file on two devices deliberately, see how conflicts surface, so it doesn't surprise you later.

## C. Capture layer (Apple side — you configure)

8. **Install Drafts** on iPhone and Mac. This is the single capture inbox.
9. **Build the Drafts action** that writes a captured note into the vault's `_inbox/` folder (or a staging location the always-on box pulls from). This is the one piece of entry-time setup that matters — get it to two taps or fewer.
10. **Optional: Apple Shortcuts / voice capture** — a Shortcut that dumps voice-to-text into the same inbox. Given you capture by voice a lot, worth doing early.

## D. Git host decision

11. **Pick a Git remote** for offsite backup. No sensitive data, so GitHub private repo is clean and simplest. Create the repo, and you'll point Claude Code's local repo at it. (Self-hosting on the NAS is possible but unnecessary given no sensitivity constraint — don't add the complexity.)

## E. Decisions Claude Code will ask you for

12. **Your taxonomy specifics** — the PARA skeleton is set, but the *Areas* and known *Projects* per system are yours to name. Draft a rough list for ATS, Promitor, Personal before you start so `config.yaml` gets populated with real values, not placeholders.
13. **Naming-convention edge cases** — confirm you're happy with `YYYY-MM-DD_slug.md`. If you have an existing habit that conflicts, decide now.
14. **Confidence threshold** — you'll tune the auto-file vs review-queue cutoff after seeing real behaviour. Start conservative (more goes to review), loosen as you trust it.

## F. The migration (you drive, Claude Code assists)

15. **Export what's worth keeping from Notion.** Per the earlier audit logic: don't bulk-dump. Only migrate content that maps to a real, current use case. The dead aspirational pages stay behind — that's the point.
16. **Run Claude Code's normalisation pass on the imported content** so old files conform to the new naming/structure.
17. **Live with it for a week on a branch before merging to main.** Resist rebuilding. If something's wrong, note it and adjust — don't start over.

## G. Ongoing (minimal by design)

18. **Check the review queue** on your own cadence — no schedule, no guilt. It's the one recurring human touchpoint.
19. **Watch the credit tracker** for the first fortnight to confirm costs are trivial and spot any runaway operation.
20. **Glance at `IDEAS.md`** when tempted to add a feature — it's where v2 ideas wait so they don't derail v1.
