# Sync Main

Safely update your local main branch after PRs have merged.

## Arguments

None

## Behavior

### Step 1: Safety Checks

```bash
CURRENT=$(git rev-parse --abbrev-ref HEAD)
MAIN_BRANCH="main"
if git show-ref --verify --quiet refs/heads/master; then
    MAIN_BRANCH="master"
fi

# Check for uncommitted changes
if [[ -n $(git status --porcelain) ]]; then
    echo "⚠️  You have uncommitted changes."
    echo "   Options:"
    echo "   1. Commit them: git commit -am 'wip'"
    echo "   2. Stash them: git stash"
    echo "   3. Discard them: git checkout -- ."
    exit 1
fi
```

### Step 2: Update Main

```bash
# If on a feature branch, stash position and switch
if [[ "$CURRENT" != "$MAIN_BRANCH" ]]; then
    echo "ℹ️  Currently on '$CURRENT', switching to $MAIN_BRANCH..."
    FEATURE_BRANCH="$CURRENT"
    git checkout "$MAIN_BRANCH"
fi

# Pull latest
echo "Pulling latest $MAIN_BRANCH..."
git pull origin "$MAIN_BRANCH" --ff-only

# If we were on a feature branch, offer to rebase
if [[ -n "$FEATURE_BRANCH" ]]; then
    echo ""
    echo "✅ $MAIN_BRANCH updated."
    echo ""
    echo "Your feature branch '$FEATURE_BRANCH' may need rebasing."
    echo "To rebase: git checkout $FEATURE_BRANCH && git rebase $MAIN_BRANCH"
fi
```

### Step 3: Clean Up Merged Branches (optional)

```bash
# List local branches that have been merged to main
MERGED=$(git branch --merged "$MAIN_BRANCH" | grep -v "^\*" | grep -v "$MAIN_BRANCH" | xargs)
if [[ -n "$MERGED" ]]; then
    echo ""
    echo "These local branches are merged and can be deleted:"
    echo "$MERGED"
    echo ""
    echo "To delete: git branch -d <branch>"
fi

# Clean up worktrees
STALE_WORKTREES=$(git worktree list --porcelain | grep -c "prunable" || echo 0)
if [[ "$STALE_WORKTREES" -gt 0 ]]; then
    echo ""
    echo "Cleaning up stale worktrees..."
    git worktree prune
fi
```

### Step 4: Return

```
✅ Main synced successfully.

Status:
- main: up-to-date with origin/main
- Merged branches: <list or "none">
- Stale worktrees: cleaned

Next: /new-feature <name> or continue on current branch
```
