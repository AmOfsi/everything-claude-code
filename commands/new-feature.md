# New Feature

Safely start a new feature branch with guardrails.

## Arguments

`<name>` — Feature branch name (without `feat/` prefix)

## Behavior

### Step 1: Safety Checks

```bash
REPO=$(git rev-parse --show-toplevel)
CURRENT=$(git rev-parse --abbrev-ref HEAD)

# Check 1: Must be on main
if [[ "$CURRENT" != "main" && "$CURRENT" != "master" ]]; then
    echo "⚠️  You're on '$CURRENT', not main."
    echo "   Stash or commit your work first, then: git checkout main"
    exit 1
fi

# Check 2: No uncommitted changes
if [[ -n $(git status --porcelain) ]]; then
    echo "⚠️  You have uncommitted changes."
    echo "   Commit them to current branch or stash: git stash"
    exit 1
fi

# Check 3: Check if main is behind origin
git fetch origin main --quiet
LOCAL=$(git rev-parse main)
REMOTE=$(git rev-parse origin/main)
if [[ "$LOCAL" != "$REMOTE" ]]; then
    echo "⚠️  Your main is behind origin/main."
    echo "   Pulling latest..."
    git pull origin main --ff-only
fi

# Check 4: Warn if PR loop is running
WORKTREES=$(git worktree list | grep -c ".worktrees" || echo 0)
if [[ "$WORKTREES" -gt 0 ]]; then
    echo "ℹ️  Note: PR loop worktree detected. That's fine - it's isolated."
    echo "   Your new feature will be based on current main."
    echo "   When the other PR merges, rebase with: git rebase origin/main"
fi
```

### Step 2: Create Branch

```bash
BRANCH_NAME="feat/<name>"
git checkout -b "$BRANCH_NAME"
echo "✅ Created branch: $BRANCH_NAME"
echo ""
echo "Next steps:"
echo "  1. Make your changes"
echo "  2. Commit: git commit -am 'feat: description'"
echo "  3. Run: /pr-loop"
```

### Step 3: Return

```
✅ Ready to work on: feat/<name>

Guardrails passed:
- [x] On main branch
- [x] No uncommitted changes
- [x] Main is up-to-date with origin
- [ ] PR loop running (isolated, won't conflict)

When done: /pr-loop
```
