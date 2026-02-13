# Two-Tier Knowledge Consolidation System

**Status**: Design Phase
**Author**: Claude Sonnet 4.5
**Date**: 2026-02-06
**Target User**: amo (limited coding background, multiple concurrent ralph loops)

## Executive Summary

This document describes a two-tier knowledge consolidation system that automatically promotes local project learnings to global knowledge when patterns prove cross-project valuable. The system runs autonomously alongside existing ralph loops, requires minimal manual intervention, and integrates seamlessly with the existing continuous learning infrastructure.

**Problem**: Ralph loops across ConcreteColumn, PLSLearner, biaxial-footing, and backend-migration generate valuable domain knowledge that stays siloed in project-specific files. A pattern learned in one project doesn't benefit others.

**Solution**: A background consolidation script that scans project knowledge, identifies cross-project patterns, and promotes them to global skills with confidence-based automation.

**Key Benefit**: Knowledge flows automatically from project-specific to global without manual copying, keeping `~/.claude/skills/learned/` fresh while preserving project-specific nuance.

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                    KNOWLEDGE FLOW                            │
└─────────────────────────────────────────────────────────────┘

TIER 1: LOCAL (Project-Scoped)
├── ~/projects/PLSLearner/knowledge/*.md         (18 docs)
├── ~/projects/ConcreteColumn/progress.txt       (ralph learnings)
├── ~/projects/biaxial-footing/CLAUDE.md         (conventions)
└── ~/projects/backend-migration/knowledge/*.md  (patterns)
                    ↓
            ┌───────────────┐
            │ Consolidator  │  ← Runs every 30 min
            │   (Python)    │     or post-ralph
            └───────────────┘
                    ↓
        ┌─────────────────────┐
        │  Promotion Engine   │
        │  - Pattern matching │
        │  - Confidence check │
        │  - Deduplication    │
        └─────────────────────┘
                    ↓
    ┌───────────────┴───────────────┐
    │                               │
    ↓ AUTO-PROMOTE                  ↓ QUEUE FOR REVIEW
(verified + 2+ projects)        (verified + 1 project)
    │                               │
    ↓                               ↓
~/.claude/skills/learned/      ~/.claude/homunculus/
    pls-file-formats.md        promotion-queue/
    structural-glossary.md         candidate-001.json
    excel-patterns.md              candidate-002.json
                    ↓
            SessionStart Hook
                    ↓
        Loaded into every session


TIER 2: GLOBAL (Loads Everywhere)
├── ~/.claude/skills/learned/*.md           (domain knowledge)
├── ~/.claude/homunculus/evolved/skills/    (behavioral skills)
├── ~/.claude/homunculus/evolved/commands/  (learned commands)
└── ~/.claude/hooks/load-instincts.sh       (injection point)
```

## Promotion Rules

| Confidence | Projects | Cross-Project Utility | Action |
|------------|----------|----------------------|--------|
| `verified` | 2+ | Any | **Auto-promote** to global |
| `verified` | 1 | High | **Queue** for human review |
| `verified` | 1 | Low/Medium | **Stay local** |
| `high` | 2+ | High | **Queue** for human review |
| `high` | 1 | Any | **Stay local** until confirmed |
| `medium` | Any | Any | **Stay local** |
| `low` | Any | Any | **Stay local** |
| `unknown` | Any | Any | **Stay local** |

### Cross-Project Utility Scoring

The consolidator uses Claude to score patterns 0-100 based on:

- **Generality** (30 points): Pattern applies to multiple domains
- **Frequency** (25 points): Referenced often in observations
- **Stability** (20 points): Hasn't changed across versions
- **Impact** (15 points): High value when applied
- **Clarity** (10 points): Well-documented, clear examples

Score → Utility:
- 70-100: High
- 40-69: Medium
- 0-39: Low

## Promotion Candidate Format

Candidates are stored as JSON at `~/.claude/homunculus/promotion-queue/*.json`:

```json
{
  "id": "candidate-20260206-143022",
  "source": {
    "projects": ["PLSLearner", "ConcreteColumn"],
    "files": [
      "~/projects/PLSLearner/knowledge/tower-geometry.md",
      "~/projects/ConcreteColumn/progress.txt:lines-145-167"
    ]
  },
  "pattern": {
    "type": "domain-knowledge",
    "category": "structural-engineering",
    "title": "Sign Convention: Compression vs Tension",
    "confidence": "verified",
    "content": "In structural analysis: compression forces are NEGATIVE, tension forces are POSITIVE. This matches Eurocode convention and differs from some US textbooks.\n\nExamples:\n- Concrete column under load: N = -500 kN (compression)\n- Tension rod: N = +200 kN (tension)\n- Bending: positive = tension at bottom fiber",
    "cross_references": [
      "EC2 Section 3.1.7",
      "PLS-Tower load case conventions"
    ]
  },
  "scoring": {
    "cross_project_utility": 85,
    "generality": 28,
    "frequency": 22,
    "stability": 20,
    "impact": 12,
    "clarity": 3
  },
  "action": "auto-promote",
  "target_skill": "~/.claude/skills/learned/structural-engineering-glossary.md",
  "merge_strategy": "append-section",
  "created_at": "2026-02-06T14:30:22Z",
  "reviewed_at": null,
  "promoted_at": null
}
```

## Integration with Existing Systems

### Continuous Learning Observer

The consolidator **complements** the observer, doesn't replace it:

| System | Focus | Output | Frequency |
|--------|-------|--------|-----------|
| Observer | Claude's **behavior** | `instincts/personal/*.jsonl` | Every 5 min |
| Consolidator | Project **domain knowledge** | `promotion-queue/*.json` | Every 30 min |

**Handoff**: Observer generates instincts → `/evolve` clusters them → consolidator promotes domain patterns that appear cross-project.

### /evolve Command

Current `/evolve` workflow:
1. User runs `/evolve`
2. Clusters `instincts/personal/*.jsonl` → `evolved/skills/`
3. Loads into SessionStart hook

**New workflow** (no change to user):
1. User runs `/evolve` (unchanged)
2. Clusters behavioral instincts (unchanged)
3. **NEW**: Consolidator runs automatically after `/evolve`
4. Scans all project knowledge
5. Auto-promotes verified cross-project patterns
6. Queues borderline candidates
7. SessionStart hook loads both evolved skills AND promoted domain knowledge (unchanged)

### SessionStart Hook

Current `load-instincts.sh`:
```bash
# Loads evolved skills
for skill in ~/.claude/homunculus/evolved/skills/*.md; do
  inject_as_context "$skill"
done

# Loads evolved commands
for cmd in ~/.claude/homunculus/evolved/commands/*.md; do
  inject_as_context "$cmd"
done
```

**New** (add one loop):
```bash
# Loads evolved skills (unchanged)
for skill in ~/.claude/homunculus/evolved/skills/*.md; do
  inject_as_context "$skill"
done

# Loads evolved commands (unchanged)
for cmd in ~/.claude/homunculus/evolved/commands/*.md; do
  inject_as_context "$cmd"
done

# NEW: Loads promoted domain knowledge
for knowledge in ~/.claude/skills/learned/*.md; do
  inject_as_context "$knowledge"
done
```

## Implementation Plan

### Phase 1: Basic Consolidator (Week 1)

**Goal**: Get cross-project pattern detection working

**Deliverables**:
1. Python script `~/.claude/homunculus/bin/consolidate.py`
2. Scans project dirs: `~/projects/*/knowledge/*.md`, `~/projects/*/progress.txt`, `~/projects/*/CLAUDE.md`
3. Extracts patterns using simple heuristics:
   - Headers that appear in 2+ projects
   - Repeated terminology (frequency analysis)
   - Explicit cross-references (e.g., "See also: PLSLearner/...")
4. Outputs `promotion-queue/*.json` (no auto-promotion yet)
5. Manual review command: `/review-promotions`

**Complexity**: Low (basic file scanning, JSON output)

**Validation**: Run on existing PLSLearner + ConcreteColumn knowledge, expect 5-10 candidates

### Phase 2: Claude-Powered Scoring (Week 2)

**Goal**: Use Claude to score cross-project utility

**Deliverables**:
1. Add `score_utility()` function to consolidator
2. For each candidate, send pattern + context to Claude Haiku
3. Prompt: "Score this pattern's cross-project utility (0-100) based on generality, frequency, stability, impact, clarity. Return JSON."
4. Store scores in candidate JSON
5. Implement auto-promotion: `verified + 2+ projects + score >= 70`

**Complexity**: Medium (Claude API integration, prompt engineering)

**Validation**: Auto-promote 2-3 verified patterns, queue 3-5 borderline

### Phase 3: Deduplication & Merging (Week 3)

**Goal**: Avoid duplicate knowledge in global skills

**Deliverables**:
1. Semantic similarity checker (cosine similarity on embeddings)
2. Merge strategies:
   - `append-section`: Add new section to existing file
   - `replace-section`: Overwrite outdated section
   - `create-new`: New global skill file
3. Conflict resolution: If 2 patterns score high but conflict, queue both for human review
4. Update existing `~/.claude/skills/learned/*.md` files in-place

**Complexity**: Medium-High (embeddings API, merge logic)

**Validation**: Promote PLSLearner patterns to `pls-file-formats.md`, verify no duplication

### Phase 4: Background Automation (Week 4)

**Goal**: Run consolidator without manual intervention

**Deliverables**:
1. Cron job or systemd timer: runs every 30 min
2. Post-ralph hook: runs after each ralph loop completes
3. `/consolidate` command: manual trigger
4. Notifications: desktop notification when promotions occur
5. Log file: `~/.claude/homunculus/logs/consolidation.log`

**Complexity**: Low (shell scripting, cron setup)

**Validation**: Let it run for 1 week, verify logs show promotions

### Phase 5: Human Review UI (Week 5)

**Goal**: Easy approval for queued candidates

**Deliverables**:
1. `/review-promotions` command
2. Interactive TUI (using `rich` library):
   - List candidates
   - Show diff (what would change in global skill)
   - Approve/Reject/Edit
3. Approved → promoted, Rejected → archived
4. Editing spawns editor on candidate JSON

**Complexity**: Medium (TUI, interactive editing)

**Validation**: Queue 10 candidates, review and approve 5, verify promotion

## Script Interfaces

### Main Consolidator

```bash
~/.claude/homunculus/bin/consolidate.py [OPTIONS]

Options:
  --projects DIR [DIR ...]   Projects to scan (default: ~/projects/*)
  --threshold SCORE          Min score for auto-promotion (default: 70)
  --dry-run                  Show what would be promoted (no changes)
  --verbose                  Detailed logging
  --force                    Skip confirmation prompts

Examples:
  # Scan all projects, auto-promote verified cross-project patterns
  consolidate.py

  # Dry run to see what would happen
  consolidate.py --dry-run

  # Scan only PLSLearner and ConcreteColumn
  consolidate.py --projects ~/projects/PLSLearner ~/projects/ConcreteColumn

  # Lower threshold (more aggressive promotion)
  consolidate.py --threshold 50
```

### Review Command

```bash
~/.claude/homunculus/bin/review-promotions.py

Interactive TUI:
  [1] Sign Convention: Compression vs Tension (score: 85)
      Sources: PLSLearner, ConcreteColumn
      Target: structural-engineering-glossary.md
      Action: [A]pprove / [R]eject / [E]dit / [S]kip

  > a

  Promoted! Updated ~/.claude/skills/learned/structural-engineering-glossary.md
```

### Background Service

```bash
# Install systemd timer (runs every 30 min)
~/.claude/homunculus/bin/install-consolidator-service.sh

# Or add to crontab
*/30 * * * * ~/.claude/homunculus/bin/consolidate.py --quiet >> ~/.claude/homunculus/logs/consolidation.log 2>&1
```

### Post-Ralph Hook

```bash
# Add to ralph.sh at end of each iteration
echo "[$(date)] Ralph iteration complete, running consolidator..."
~/.claude/homunculus/bin/consolidate.py --projects $(pwd) --quiet
```

## Example Workflows

### Workflow 1: PLSLearner Pattern Auto-Promotion

1. Ralph loop in PLSLearner learns: "Material section `[E_Material]` always precedes `[Tower Geometry]`"
2. Writes to `~/projects/PLSLearner/knowledge/file-structure.md` with confidence `verified`
3. 30 minutes later, consolidator runs
4. Scans PLSLearner knowledge, finds pattern
5. Checks ConcreteColumn knowledge, finds reference to same pattern (different wording)
6. Scores pattern: generality=25, frequency=20, stability=18, impact=12, clarity=10 → **85**
7. Pattern is `verified` + 2 projects + score >= 70 → **auto-promote**
8. Merges into `~/.claude/skills/learned/pls-file-formats.md`
9. Logs: `[2026-02-06 14:30] Auto-promoted: PLS file structure (85 score, 2 projects)`
10. Next session anywhere: pattern loads automatically via SessionStart hook

### Workflow 2: Single-Project Pattern Queued

1. ConcreteColumn learns: ".NET `[Formula]` attributes extracted at build time"
2. Writes to `~/projects/ConcreteColumn/CLAUDE.md` with confidence `verified`
3. Consolidator runs, finds pattern
4. Only 1 project, but scores high utility (70)
5. `verified` + 1 project + high utility → **queue for review**
6. Creates `~/.claude/homunculus/promotion-queue/candidate-20260206-143022.json`
7. Desktop notification: "1 promotion candidate queued for review"
8. User runs `/review-promotions`
9. TUI shows pattern, user approves
10. Promoted to `~/.claude/skills/learned/dotnet-patterns.md`

### Workflow 3: Deduplication Merge

1. PLSLearner has: "Compression is negative, tension is positive"
2. ConcreteColumn has: "Sign convention: compression < 0, tension > 0"
3. Both patterns score high, both `verified`, both reference EC2
4. Consolidator detects semantic similarity (cosine = 0.92)
5. Merges into single entry in `structural-engineering-glossary.md`:
   ```markdown
   ### Sign Convention: Compression vs Tension

   **Convention**: Compression forces are NEGATIVE, tension forces are POSITIVE.

   **Standard**: Eurocode (EC2 Section 3.1.7)

   **Examples**:
   - Concrete column under load: N = -500 kN (compression)
   - Tension rod: N = +200 kN (tension)
   - Bending moment: M+ causes tension at bottom fiber

   **Source**: Verified across PLSLearner and ConcreteColumn projects
   ```
6. Removes duplicate from local project files (optional, configurable)

## File Format Specifications

### Project Knowledge Metadata

To help consolidator parse local knowledge, add YAML frontmatter to `knowledge/*.md` files:

```markdown
---
confidence: verified
category: file-formats
cross_project: true
tags: [pls-tower, geometry, sections]
last_verified: 2026-02-06
---

# Tower Geometry Section

The `[Tower Geometry]` section always appears after `[E_Material]`...
```

### Progress.txt Pattern Markers

Ralph loops can mark patterns for promotion using special comments:

```
[2026-02-06 14:30] LEARNED: Sign convention - compression is negative
[confidence: verified]
[category: structural-engineering]
[cross-project: true]

In EC2 calculations, compression forces are always negative values...
```

Consolidator regex: `\[LEARNED: (.+?)\]` followed by metadata tags.

### Global Skill File Structure

All `~/.claude/skills/learned/*.md` files follow this template:

```markdown
# Skill Title

**Category**: domain-knowledge | behavioral | tooling
**Last Updated**: 2026-02-06
**Confidence**: verified
**Source Projects**: PLSLearner, ConcreteColumn

## Overview

Brief description of what this skill covers.

## Patterns

### Pattern 1: Title

**Context**: When to use this pattern

**Example**:
```python
# Code example
```

**Verified In**: PLSLearner (18 instances), ConcreteColumn (4 instances)

### Pattern 2: Title

...

## Cross-References

- See also: `structural-engineering-glossary.md`
- Related: `pls-file-formats.md`

## Changelog

- 2026-02-06: Added Pattern 2 from ConcreteColumn
- 2026-01-15: Initial creation from PLSLearner
```

## Performance Considerations

### Scan Efficiency

- **Incremental scanning**: Only scan files modified since last run (use `mtime`)
- **Ignore build artifacts**: Skip `node_modules/`, `__pycache__/`, `.git/`, `bin/`, `obj/`
- **Parallel processing**: Scan multiple projects concurrently (Python `multiprocessing`)
- **Cache embeddings**: Store pattern embeddings, recompute only on content change

### Claude API Usage

- **Batch scoring**: Score 10 patterns per API call (cheaper than 10 separate calls)
- **Use Haiku**: Scoring doesn't need Sonnet (3x cost savings)
- **Cache prompts**: Use prompt caching for system prompt (90% discount on repeated tokens)
- **Throttling**: Max 1 API call per second to avoid rate limits

### Storage

- **Projection**: 100 projects × 20 knowledge files × 10 KB = 20 MB (negligible)
- **Promotion queue**: Max 100 candidates × 5 KB = 500 KB (negligible)
- **Logs**: Rotate after 10 MB (`logrotate` compatible)

## Security & Privacy

### EFLA Proprietary Data

**Critical**: PLSLearner processes proprietary .tow files from EFLA hf.

**Safeguards**:
1. Consolidator NEVER sends raw .tow file contents to Claude API
2. Only sends **extracted patterns** (generic, no client names/project IDs)
3. Confidence levels require human verification before promotion
4. Global skills contain NO client-specific data
5. Promotion queue candidates reviewed before leaving local machine

**Example SAFE pattern**:
```markdown
✅ "Material section `[E_Material]` precedes `[Tower Geometry]`"
```

**Example UNSAFE pattern** (NEVER promote):
```markdown
❌ "Project XYZ-2024 for Client ABC uses 3-leg tower at GPS coordinates..."
```

### API Key Management

Consolidator needs Claude API key:

```bash
# ~/.claude/homunculus/config.json
{
  "api_key": "${ANTHROPIC_API_KEY}",  # Read from env var
  "model": "claude-haiku-4.5",
  "max_tokens": 1024
}
```

Never hardcode keys, always use environment variables.

## Future Enhancements

### Phase 6+: Advanced Features

1. **Confidence boost**: Patterns used successfully in 3+ sessions → auto-upgrade `high` → `verified`
2. **Deprecation detection**: Pattern not referenced in 90 days → queue for archival
3. **Version control**: Track global skill changes in git, allow rollback
4. **Pattern search**: CLI to search global skills: `claude-search "sign convention"`
5. **Multi-user sync**: Share global skills across team (EFLA use case)
6. **Web UI**: Browse/edit global skills in browser (alternative to TUI)
7. **LLM-powered merging**: Use Claude to intelligently merge conflicting patterns
8. **Cross-language patterns**: Detect patterns that apply to Python, C#, TypeScript simultaneously

## Success Metrics

After 4 weeks of operation, measure:

1. **Promotion rate**: 5-10 patterns/week promoted to global
2. **False positive rate**: < 10% of auto-promotions need rollback
3. **Coverage**: 80%+ of cross-project patterns detected
4. **Latency**: Consolidation completes in < 60 seconds
5. **User satisfaction**: amo approves 70%+ of queued candidates (indicates good scoring)

## Open Questions

1. **Should consolidator run per-project or globally?** → Start global, add per-project opt-in later
2. **How to handle contradictory patterns?** → Queue both, let human decide
3. **Delete local knowledge after promotion?** → No, keep for context, just mark as `promoted: true`
4. **Notify on every promotion?** → No, daily digest email/notification
5. **Integrate with git?** → Yes, commit global skill changes automatically with message: `chore: consolidate knowledge from PLSLearner (3 patterns)`

## Implementation Checklist

### Phase 1
- [ ] Create `consolidate.py` script
- [ ] Implement project scanning (knowledge/, progress.txt, CLAUDE.md)
- [ ] Extract patterns with heuristics (headers, frequency, cross-refs)
- [ ] Output promotion queue JSON
- [ ] Test on PLSLearner + ConcreteColumn

### Phase 2
- [ ] Add Claude API integration
- [ ] Implement `score_utility()` function
- [ ] Test scoring on 10 sample patterns
- [ ] Implement auto-promotion logic
- [ ] Validate: 2-3 auto-promotions, 3-5 queued

### Phase 3
- [ ] Add embeddings API (OpenAI or Voyage)
- [ ] Implement semantic similarity checker
- [ ] Define merge strategies (append, replace, create)
- [ ] Test deduplication on PLSLearner knowledge
- [ ] Validate: no duplicate entries in global skills

### Phase 4
- [ ] Create systemd timer (30 min interval)
- [ ] Add post-ralph hook to ralph.sh template
- [ ] Create `/consolidate` command
- [ ] Add desktop notifications (notify-send)
- [ ] Setup log rotation

### Phase 5
- [ ] Build review TUI with `rich`
- [ ] Implement approve/reject/edit flows
- [ ] Test interactive review with 10 candidates
- [ ] Add keyboard shortcuts (j/k navigation, a/r/e actions)
- [ ] Validate: smooth UX, < 10 seconds per review

## Conclusion

This two-tier knowledge consolidation system solves the siloed knowledge problem by automatically promoting verified cross-project patterns to global skills. It integrates seamlessly with existing continuous learning infrastructure, requires minimal manual intervention after Phase 4, and preserves EFLA proprietary data security.

**Next Steps**:
1. Review this design doc
2. Approve Phase 1 implementation
3. Create `~/.claude/homunculus/bin/` directory
4. Start coding `consolidate.py`

**Estimated Timeline**: 5 weeks to full automation (Phase 1-4), +1 week for review UI (Phase 5)

**Effort**: ~20 hours total (4 hours/week for 5 weeks) — achievable for limited coding background with Claude assistance
