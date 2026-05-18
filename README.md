# Trajectory Golf — Shared Brain

The single source of truth for what's going on with **Trajectory Golf LLC**, shared between Kyle, Claude (Anthropic), and Grok (xAI).

**Public repo** — chosen for Grok read access. Contains EIN, financial model, account IDs, and business strategy. Avoid committing actual tokens / passwords / API keys; account *identifiers* (Stripe `acct_*`, Optix plan IDs) are fine.

**Raw URL for AI fetch:** `https://raw.githubusercontent.com/kylehillman/trajectory-golf-brain/main/HANDOVER.md`

---

## Files

| File | Purpose |
|---|---|
| `HANDOVER.md` | The living document. Full project state — overview, tech, decisions, open tasks, code snippets, risks. Both AIs read this first every session. **§11 Session Log** at the bottom is the rolling changelog. |
| `README.md` | This file. The workflow. |

That's it. One doc, intentionally.

---

## The workflow

### When starting a session with Claude (Claude Code, locally)

Claude can read and write this repo directly. At session start:

```bash
cd /Users/kylehillman/golfsimulator/trajectory-golf-brain
git pull
```

Then tell Claude: *"Read HANDOVER.md, then [task]."*

When the session changes anything material (decision made, task closed, integration shipped, number revised), Claude updates `HANDOVER.md`:

1. Edit the relevant section in place (don't just add stale info to the bottom).
2. Append a new entry to **§11 Session Log** — date, AI, what changed, what's next.
3. Commit + push:
   ```bash
   git add HANDOVER.md
   git commit -m "session: <one-line summary>"
   git push
   ```

### When starting a session with Grok (grok.com, web)

Grok can fetch the public raw URL directly. The Grok Project's Custom Instructions (see below) tell it to do this automatically. If you ever need to do it manually, paste this into the chat:

> "Fetch https://raw.githubusercontent.com/kylehillman/trajectory-golf-brain/main/HANDOVER.md and read it before anything else. Then [task]."

Grok cannot push to GitHub directly. End any session that changed material state by asking:

> "Draft the new §11 Session Log entry and any HANDOVER.md edits as a single replacement block I can paste into my local file."

Copy Grok's output, paste into `HANDOVER.md` locally, then commit + push (Claude can do this, or run `git` directly):

```bash
git add HANDOVER.md
git commit -m "session: grok <one-line summary>"
git push
```

### When you (Kyle) make a unilateral decision outside a session

Same as above — edit `HANDOVER.md`, append a Session Log entry (`AI: none`), commit, push. The repo is the source of truth; both AIs trust it on next read.

---

## Conventions

### Session Log entry format

```markdown
### YYYY-MM-DD · <AI name or "Kyle"> · <one-line headline>

- What changed in HANDOVER.md (sections updated)
- Decisions made + rationale
- **Open for next session:** what the next AI should pick up
```

Newest entry on top of §11. Keep entries short — three to six bullets is plenty. The body of `HANDOVER.md` is where detail lives; the log is the breadcrumb trail.

### Editing `HANDOVER.md`

- **Edit in place.** If §2.3 "What's NOT yet done" has a task that just got done, *delete it from §2.3* and *update §2.2* — don't just add a note. The doc should always be readable cold as the current state, not a diff log.
- **Don't bloat the doc.** If a section grows past ~50 lines or gets stale weekly, that's a sign it should become its own file. Today, one file is enough.
- **Match the existing voice.** Tables and bullets > prose. Numbers > adjectives. Honest > hyped.
- **Don't commit secrets.** Tokens, API keys, passwords stay in `.env.local` on Kyle's machine and in Vercel. Account IDs and plan IDs (Optix, Stripe `acct_*`) are fine.

### Commit message convention

```
session: <ai> <headline>     # for AI-driven session updates
fix: <what>                  # corrections to wrong info
add: <what>                  # whole new sections / docs
```

---

## Why this exists

Kyle is solo-building Trajectory Golf part-time alongside SpaceX. Different sessions happen with different tools (Claude Code on the laptop, Grok on the phone between launch ops). Without a shared brain, every session re-litigates the same decisions and loses ground.

This repo guarantees: whichever AI you talk to next walks in already knowing what the other one just did.
