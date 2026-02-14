---
description: "[DEPRECATED] Use /pr-loop instead. This skill has been replaced by the autonomous PR review loop agent."
---

# PR Review Loop (Deprecated)

This skill has been replaced by `/pr-loop`.

## Migration

Use `/pr-loop` instead. The new command:
- Runs autonomously in a background agent
- Uses an isolated git worktree (doesn't block your session)
- Communicates via Ralph Mail for triage decisions
- Handles CI waiting, review fetching, and fixes automatically

## Usage

Just type `/pr-loop` to start the autonomous PR review loop on your current branch.
