# Task Tracking — Beads (bd)

## Core Rule

**NEVER use markdown TODOs, task lists, or any other tracking method.**
ALL issue tracking goes through `bd`. No exceptions.

---

## Session Start

At the start of every session, run:

```bash
bd prime          # Load current workflow context (~80 lines, always fresh)
bd ready --json   # See tasks with no open blockers
```

If `bd prime` is unavailable, fall back to:

```bash
bd list --json
bd ready --json
```

---

## Workflow

### 1. Find work
```bash
bd ready                      # Tasks with no blockers
bd ready --include-deferred   # Include future/deferred issues
```

### 2. Claim a task (atomic — sets assignee + in_progress)
```bash
bd update <id> --claim
```

### 3. Discover work while coding
When you find a bug, missing test, or new requirement while working:
```bash
bd create "Found bug in auth refresh" -t bug -p 1 \
  --deps discovered-from:<current-id> --json
```
File it immediately. Do not lose track of it.

### 4. Close when done
```bash
bd close <id> --reason "Completed" --json
```

### 5. Pick next work
```bash
bd ready --json
```

---

## Creating Issues

```bash
# Basic
bd create "Title" -t bug|feature|task|chore|epic -p 0-4 --json

# With description
bd create "Title" -t task -p 2 --description="What needs doing and why" --json

# With due date or deferral
bd create "Title" --due=+6h --json
bd create "Title" --defer=tomorrow --json

# Discovered while working on another issue
bd create "Title" -p 1 --deps discovered-from:<parent-id> --json

# Special characters in description — pipe via stdin to avoid shell escaping
echo 'Description with `backticks` and "quotes"' | bd create "Title" --description=- --json
```

Priority scale: `0` = critical, `1` = high, `2` = normal, `3` = low, `4` = someday.

---

## Updating Issues

```bash
bd update <id> --status in_progress
bd update <id> --title "New title"
bd update <id> --description "New description"
bd update <id> --design "Design notes"
bd update <id> --notes "Additional context"
bd update <id> --acceptance "Acceptance criteria"
```

**DO NOT use `bd edit`** — it opens `$EDITOR` which agents cannot use.

---

## Dependencies

```bash
bd dep add <child-id> <parent-id>          # child blocks on parent
bd dep tree <id>                           # Show dependency tree
```

Dependency types: `blocks`, `related`, `parent-child`, `discovered-from`, `conditional-blocks`.

Only `blocks` dependencies affect the `bd ready` queue.

---

## Querying

```bash
bd list --json          # All open issues
bd show <id> --json     # Full detail + audit trail
bd stats                # Project progress summary
bd dep tree <id>        # Dependency graph for an issue
```

Always use `--json` for programmatic use. Use `--agent` flag when spawning bd from scripts.

---

## Epics & Hierarchy

Beads supports hierarchical IDs:
- `bd-a3f8` — Epic
- `bd-a3f8.1` — Task under epic
- `bd-a3f8.1.1` — Sub-task

When planning large work: create an epic first, then file tasks as children with `--deps`.

---

## Issue Types & Statuses

**Types:** `bug` · `feature` · `task` · `chore` · `epic`

**Statuses:** `open` · `in_progress` · `blocked` · `deferred` · `closed`

---

## Git Commit Convention

Include the issue ID at the end of every commit message:

```
git commit -m "Fix auth token validation (bd-abc)"
```

This lets `bd doctor` detect orphaned issues (committed but not closed).

---

## End of Session

Before ending any session where code was changed:

```bash
# 1. File any remaining discovered work
bd create "..." --json

# 2. Close completed issues
bd close <id> --reason "Done" --json

# 3. Push (mandatory if Dolt remote is configured)
git pull --rebase
git push

# 4. Verify
bd ready --json     # What's next for the next agent/session
```

Beads acts as working memory between sessions. Start a fresh session often — it's cheaper and cleaner.
