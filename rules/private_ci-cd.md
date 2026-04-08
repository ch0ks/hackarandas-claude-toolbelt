# CI/CD Pipeline Conventions

## SAST Gate Placement

SAST runs at multiple pipeline stages with different scope and blocking behavior:

```
[pre-commit] → [PR check] → [merge gate] → [nightly full scan] → [release gate]
```

| Stage | Scope | Blocking | Timeout |
|-------|-------|----------|---------|
| **pre-commit** | Changed files only | No — warn only | 30s |
| **PR check** | Changed files + diff context | Yes — on critical/high | 5 min |
| **Merge gate** | Full repo, incremental | Yes — on critical/high | 15 min |
| **Nightly full scan** | Full repo, all rules | No — file issues | 60 min |
| **Release gate** | Full repo, release rules | Yes — on critical/high/medium | 30 min |

Never run a full-repo scan as a blocking PR check — it will be disabled within a week.

---

## Blocking vs. Non-Blocking Policy

**Block the pipeline on:**
- `critical` findings — always, no exceptions
- `high` findings — always, unless explicitly suppressed with a dated justification comment
- `high-confidence medium` findings — at merge gate and release gate

**Do not block on:**
- `low-confidence medium` findings — file a Beads issue, continue
- `low` findings — report only, never block
- `info` findings — aggregate into weekly digest, never block

A suppressed `critical` or `high` requires:
1. A comment in code with `# nosec: <rule-id> — <justification> — reviewed by <name> <date>`
2. A Beads issue filed with expiry date for re-evaluation
3. Security team sign-off for `critical`

---

## PR Integration

Every PR with SAST findings must surface:
- Finding count by severity in the PR description (auto-generated)
- Inline annotations on the diff for new findings only
- A link to the full finding detail in the artifact store
- Diff-aware mode — only report findings introduced by this PR, not pre-existing ones

**Never fail a PR for pre-existing findings.** Developers did not introduce them.
Track pre-existing findings separately and address via a dedicated remediation backlog.

Findings on unchanged lines are `[WARN]` only — annotate but do not block.

---

## Failure Thresholds

Define thresholds per pipeline stage explicitly. Do not use tool defaults.

```yaml
# Example threshold config — adjust per project risk profile
pr_check:
  block_on:
    - severity: critical
      confidence: any
    - severity: high
      confidence: high

merge_gate:
  block_on:
    - severity: critical
      confidence: any
    - severity: high
      confidence: any
    - severity: medium
      confidence: high

release_gate:
  block_on:
    - severity: critical
      confidence: any
    - severity: high
      confidence: any
    - severity: medium
      confidence: high|medium
  warn_on:
    - severity: medium
      confidence: low
    - severity: low
      confidence: high
```

Document threshold rationale in the pipeline config — thresholds without justification
get loosened without discussion.

---

## Artifact Retention

| Artifact | Retention | Storage |
|----------|-----------|---------|
| Full scan results (SARIF/JSON) | 90 days | Artifact store |
| PR finding annotations | Duration of PR + 30 days | CI system |
| Nightly scan results | 1 year | Artifact store |
| Release gate results | Indefinite | Artifact store + security archive |
| Suppression audit log | Indefinite | Security archive |

SARIF format is preferred for all tool output — it's portable across tools and integrates
with GitHub Advanced Security, GitLab, and most security dashboards.

Always retain the raw tool output alongside any normalized format. Raw output is needed
for tool version comparison and regression analysis.

---

## Performance Standards

SAST that is slow gets disabled. Enforce these:

- Pre-commit hook: **< 30 seconds** — if it exceeds this, scope it down or remove it
- PR check: **< 5 minutes** — diff-aware scanning only, parallelize by file type
- Merge gate: **< 15 minutes** — incremental scan on changed modules
- Nightly: no hard limit, but alert if it exceeds **60 minutes**

Profile scan time per rule regularly. Remove or demote rules that contribute > 20% of
scan time with < 5% of actionable findings.

---

## New Rule Rollout Process

Never enable a new rule directly in blocking mode:

1. **Shadow mode** (1–2 weeks) — run rule, log findings, block nothing
2. **Measure FP rate** — if > 20%, rework before proceeding
3. **Warn mode** (1–2 weeks) — annotate PRs, block nothing, collect developer feedback
4. **Blocking mode** — enable at appropriate pipeline stage per threshold policy

Skip shadow/warn mode only for `critical` severity rules with `high` confidence
and a measured FP rate < 5% from the pilot codebase.

---

## Dependency Scanning

Run as a separate job from SAST — different cadence, different blocking policy:

- On every PR: scan new/modified dependencies only
- Nightly: full dependency graph scan
- Block on: known exploited CVEs (CISA KEV list), CVSS ≥ 9.0 with a fix available
- Warn on: CVSS 7.0–8.9, no fix available CVEs
- Never block on: CVEs with no fix and no known exploit

Pin tool versions in CI config. A dependency scanner that auto-updates its vuln DB
mid-sprint causes noise spikes. Update the DB on a weekly scheduled cadence.
