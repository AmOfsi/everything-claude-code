# Ralph Mail System

## Quick Reference

**Send mail**: Create `.md` file in `ralph-outbox/`
**Receive mail**: Check `ralph-inbox/` (delivered automatically every 5s)
**Archive old mail**: Happens automatically (30+ min old)

## Sending Mail

Create `ralph-outbox/your-message.md`:

```markdown
# Message Subject

**From**: [this-project]
**To**: ProjectName (or ALL_PROJECTS for broadcast)
**Date**: YYYY-MM-DD HH:MM UTC

---

Your message content here...
```

## Available Projects

- TowerFEM, PLSLearner, ResultAnalyzer, TowerLoads
- PLSDrawingReader, ConcreteColumn, Boltgroup, biaxial-footing
- TowerBatchRun, everything-claude-code, amo-hub

## Examples

**Ask a question:**
```markdown
# Question: File Format

**From**: ProjectA
**To**: TowerFEM
**Date**: 2026-02-13 09:00 UTC

What is the .res file format structure?
```

**Broadcast update:**
```markdown
# Breaking Change: API Update

**From**: PLSLearner
**To**: ALL_PROJECTS
**Date**: 2026-02-13 09:00 UTC

/generate endpoint now requires `voltage` parameter.
```

**Request deployment:**
```markdown
# Deploy Request

**From**: ProjectA
**To**: amo-hub
**Date**: 2026-02-13 09:00 UTC

New build ready: #12345
Please deploy when convenient.
```

## Mail Lifecycle

1. **Create** message in your `ralph-outbox/`
2. **Delivered** by mail-router daemon (5s poll) to recipient's `ralph-inbox/`
3. **Enriched** by local LLM (lgrep code search + ollama) — creates `.enriched.md` companion
4. **Read** by recipient session — check both `.md` and `.enriched.md` files
5. **Marked read** — rename to `.resolved-<original-name>` when acted on
6. **Archived** — resolved mail gets archived; unread mail stays for 24 hours

## Reading Mail (for Claude sessions)

When checking `ralph-inbox/`:
1. Read all `.md` files (skip `.resolved-*` hidden files — those are already processed)
2. Check for `.enriched.md` companions — these have pre-gathered code context from lgrep
3. After acting on a message, rename it: `mv file.md .resolved-file.md`
4. NEVER delete mail — always rename to `.resolved-*` so it can be archived properly
