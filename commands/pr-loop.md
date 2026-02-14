---
description: Run the autonomous PR review loop agent. Detects current repo/branch, spawns agent in isolated worktree, handles CI and reviews via mail.
---

# PR Loop

Run the autonomous PR review loop agent on the current repository.

## Triggers

Invoke with: `/pr-loop`, "run the pr loop", "start pr review loop", "pr review agent"

## Arguments

$ARGUMENTS can be:
- `(empty)` — Use current branch and repo
- `<branch>` — Specify branch name
- `--background` — Run in background (default: foreground for first run)

## Behavior

**IMPORTANT**: This command MUST spawn a background agent immediately. Do NOT run pre-flight checks in the main session - the agent handles all of that.

1. **Detect context** (quick, no validation):
   - Current repo: `git rev-parse --show-toplevel`
   - Current branch: `git rev-parse --abbrev-ref HEAD`

2. **Spawn agent immediately** (fire and forget):
   ```
   Task({
     subagent_type: "pr-review-loop",
     prompt: `
       Execute the full PR review loop autonomously.

       repo: <detected repo path>
       branch: <detected branch or "auto" if on main with uncommitted changes>
       base: main

       Handle all pre-flight checks, branch creation, CI waiting, and triage.
       Send mail to USER when decisions are needed.
       Do NOT block - run the full loop.
     `,
     run_in_background: true
   })
   ```

3. **Return immediately to user**:
   ```
   🚀 PR Review Loop agent spawned in background.

   Repo: <repo>
   Branch: <branch>

   The agent will:
   - Create worktree for isolation
   - Handle branch creation if needed
   - Push and create PR
   - Wait for CI
   - Send you mail when triage/merge decisions are needed

   Respond with: pr-respond <project> <pr#> --triage "1:A,2:S"
   ```

**DO NOT** ask the user questions or run pre-flight checks in the main session. The agent handles everything autonomously.

## Example

```
User: "run the pr loop"

Claude: Detected ResultAnalyzer on branch feature/sync-highlight.
        Starting PR review loop agent in background...

        You'll receive a notification when triage is needed.
```
