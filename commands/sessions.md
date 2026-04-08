<!-- author: Adrian Puente Z. (@ch0ks) — www.hackarandas.com -->
List all Claude Code sessions across all projects, sorted by most recent. Show project context, date, and the exact resume command for each session.

---

## Step 1 — Collect All Sessions

```bash
# Find all session jsonl files and extract metadata
for f in ~/.claude/projects/*/*.jsonl; do
  [ -f "$f" ] || continue

  # Extract session ID from filename
  session_id=$(basename "$f" .jsonl)

  # Skip subagent files — they are not resumable sessions
  [[ "$session_id" == agent-* ]] && continue

  # Extract project name from directory path — strip the hashed prefix
  project_raw=$(echo "$f" | sed 's|.*/projects/||' | sed 's|/.*||')

  # Clean up project name for display:
  # Remove leading -Users-<username>- prefix
  # Replace remaining hyphens with slashes to reconstruct the path
  project=$(echo "$project_raw" \
    | sed 's/^-Users-[^-]*-//' \
    | sed 's/^-home-[^-]*-//' \
    | sed 's/-\(github\|src\|code\|workspace\)/\/\1/g' \
    | sed 's/-/ /g')

  # Get last modified timestamp (macOS + Linux)
  modified=$(stat -f "%Sm" -t "%Y-%m-%d %H:%M" "$f" 2>/dev/null \
    || stat -c "%y" "$f" 2>/dev/null | cut -c1-16)

  # Get first user message from the jsonl as session context (first 80 chars)
  first_msg=$(grep '"role":"user"' "$f" 2>/dev/null | head -1 \
    | python3 -c "
import sys, json
try:
    line = sys.stdin.read()
    obj = json.loads(line)
    content = obj.get('message', {}).get('content', '')
    if isinstance(content, list):
        text = next((c.get('text','') for c in content if c.get('type')=='text'), '')
    else:
        text = str(content)
    print(text[:80].replace('\n', ' '))
except:
    print('')
" 2>/dev/null)

  echo "$modified|||$session_id|||$project_raw|||$project|||$first_msg"

done | sort -r
```

---

## Step 2 — Format and Display

Parse the output from Step 1 and display as a numbered, readable table:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 Claude Code Sessions
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 #   Date              Project                                  First Message
─────────────────────────────────────────────────────────────────────────────
 1   2026-03-20 14:32  spdrs-code / FundingEventQueue           security review of create_risk...
 2   2026-03-18 09:15  semgrep-crawler                          run semgrep pro scan on...
 3   2026-03-15 16:44  Security-Design-Reviews-SPDR-SPS         review the threat model for...
 ...
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

To resume a session:
  claude --resume <session-id>

Full session IDs:
  1  →  3bdbff51-3aab-4643-a01e-4e305b224650
  2  →  f805ff93-3b09-4221-be20-eb87bc07cb90
  3  →  fecbde4f-86aa-4f1d-857e-966843381d86
  ...
```

Show a maximum of 30 sessions by default.
Always show the full session ID list at the bottom — it is the key piece of information.

---

## Step 3 — Handle Arguments

If the user passes a number as an argument (e.g. `/sessions 2`), immediately print the resume command for that session:

```
Resume session #2:

  claude --resume f805ff93-3b09-4221-be20-eb87bc07cb90

Project:  semgrep-crawler
Date:     2026-03-18 09:15
Context:  run semgrep pro scan on...
```

If the user passes a project name fragment (e.g. `/sessions semgrep`), filter sessions to only those whose project path contains that string:

```bash
# Filter by argument if provided
$ARGUMENTS
```

---

## Step 4 — Handle Edge Cases

**No sessions found:**
```
No Claude Code sessions found in ~/.claude/projects/

This is expected if you have not started any Claude Code sessions yet,
or if sessions have been cleared.
```

**Single session:**
Display it directly with the resume command — no need for a table.

**Sessions older than 90 days:**
Mark them with `[90d+]` in the date column — these may have degraded context
but are still resumable.

```
 4   [90d+] 2025-12-01 10:20  AskMyFiles    build the file upload feature...
```

---

## Step 5 — Print Quick Reference

Always end with:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Tip: Sessions older than ~2 weeks may have lost context.
     Start a fresh session and reference the old one with:
       claude --resume <id> --continue
     or just open a new session in the same project directory.

Filter by project:  /sessions <project-name>
Resume by number:   /sessions <number>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```
