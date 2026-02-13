# CLAUDE.md — everything-claude-code

## Purpose

Community collection of Claude Code patterns: skills, agents, hooks, contexts, and best practices.

## Ralph Mail System

This repo can communicate asynchronously with other projects using Ralph Mail.

### Sending Mail

Create markdown files in `ralph-outbox/` addressed to other projects:

```markdown
# Subject Line

**From**: everything-claude-code
**To**: ProjectName (e.g., TowerFEM, PLSLearner, ResultAnalyzer)
**Date**: YYYY-MM-DD HH:MM UTC
**Topic**: brief-topic-slug

---

## Message Content

Your message here...
```

The hub-watch daemon will route messages to the target project's `ralph-inbox/`.

### Receiving Mail

Check `ralph-inbox/` for messages from other projects. Messages are delivered by hub-watch running in amo-hub.

### Available Projects

- **TowerFEM**: C-based FEM solver for tower structural analysis
- **PLSLearner**: Python FastAPI service for ML-based tower generation
- **ResultAnalyzer**: Express/React web app for viewing analysis results
- **TowerLoads**: Tower load calculation and analysis
- **PLSDrawingReader**: PLS-CADD drawing file parser
- **ConcreteColumn**: Concrete column design
- **Boltgroup**: Bolt group analysis
- **biaxial-footing**: Biaxial footing design
- **amo-hub**: Knowledge repository and orchestration hub

### When to Use Ralph Mail

- Cross-project coordination (build failures, deployment requests)
- Knowledge queries (ask another project about its domain)
- Status updates across async sessions
- Broadcasting changes that affect multiple projects

### Examples

**Request deployment:**
```markdown
# Deploy Request - New Build Ready

**From**: everything-claude-code
**To**: amo-hub
**Date**: 2026-02-13 09:00 UTC
**Topic**: deploy-request

---

New build available for everything-claude-code artifacts.
Please deploy to production when ready.

Build: #12345
Artifacts: skills/, agents/, hooks/
```

**Query another project:**
```markdown
# Question: Tower Analysis Format

**From**: everything-claude-code
**To**: TowerFEM
**Date**: 2026-02-13 09:00 UTC
**Topic**: res-file-format

---

What is the format of the .res output file?
Need to document this for ResultAnalyzer integration.
```

## Directory Structure

- `skills/` — Reusable skill patterns
- `agents/` — Custom agent definitions
- `hooks/` — Event hooks (PreToolUse, PostToolUse, etc.)
- `contexts/` — Reusable context files
- `rules/` — Global coding rules and patterns
- `scripts/` — Utility scripts
- `docs/` — Documentation

## When Working Here

- Test skills/agents/hooks locally before committing
- Follow the contribution guidelines in CONTRIBUTING.md
- Document new patterns with clear examples
- Keep skills focused and composable
