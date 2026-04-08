# hackarandas-claude-toolbelt

A curated set of Claude Code rules, commands, and skills for security engineering — by [Adrian Puente Z. (@ch0ks)](https://hackarandas.com).

---

## What's in here

| Type | Location | What it does |
|------|----------|--------------|
| **Rules** | `rules/` | Behavioral guidelines injected into every Claude session |
| **Commands** | `commands/` | Custom `/slash-commands` with step-by-step shell procedures |
| **Skills** | `skills/` | Reusable task definitions triggered by Claude's skill matcher |

---

## Flagship: Security Review Workflow

Four skills and commands that form a complete, auditable security review pipeline. Run them in sequence on any repository.

### Stage 1 — Setup
**`/security-semgrep-setup`** (command)

Checks for Semgrep, installs it if missing (Homebrew → pip → pip --user), logs in to Semgrep Pro, enables the Pro engine, and runs a local scan. Saves a dated JSON artifact to `security-review/semgrep-results-YYYYMMDD.json` for downstream stages.

```
/security-semgrep-setup
```

### Stage 2 — Code Review
**`security-code-review`** (skill)

Consumes the Semgrep JSON artifact, runs interprocedural taint analysis, and performs manual review across OWASP Top 10 categories (injection, auth, crypto, secrets, SSRF, and more). Writes a formal Security Code Review Report with CWE mappings, evidence, and remediation guidance to `security-review/security-code-review-report-YYYYMMDD.md`.

```
/security-code-review
```

### Stage 3 — IaC-Grounded Triage
**`security-iac-triage`** (skill)

Reads the code review report and your IaC files (Terraform, Kubernetes, Docker Compose, CloudFormation, Azure Pipelines) to extract deployment facts. Scores each finding with CVSS 4.0, justifying every vector against observable infrastructure evidence. Appends the triage section to the existing report. Supports three scoring postures: Strict, Standard, and Lenient.

```
/security-iac-triage
```

### Stage 4 — Vibe Security Patching
**`security-vibe-patch`** (skill)

Reads the code review report and generates minimal, precise remediation patches using the [Vibe Security Patching](https://hackarandas.com) methodology. One commit per finding. Skips findings where a correct fix cannot be determined statically (LOW confidence). Writes a Security Vibe Patch Report to `security-review/`, then opens a pull request with a structured security PR description.

```
/security-vibe-patch          # patch [BLOCK] findings only (default)
/security-vibe-patch warn     # patch [BLOCK] + [WARN] findings
/security-vibe-patch all      # patch all unresolved findings
```

**Pipeline output** (all at the target repository root, never here):
```
security-review/
├── semgrep-results-YYYYMMDD.json
├── security-code-review-report-YYYYMMDD.md
└── security-vibe-patch-report-YYYYMMDD.md
```

---

## Install

Clone this repo and copy assets to your global Claude config directory.

```bash
git clone https://github.com/ch0ks/hackarandas-claude-toolbelt.git
cd hackarandas-claude-toolbelt

# Create target directories if they don't exist
mkdir -p ~/.claude/rules ~/.claude/commands ~/.claude/skills

# Deploy
cp rules/*.md ~/.claude/rules/
cp commands/*.md ~/.claude/commands/
cp -r skills/* ~/.claude/skills/
```

Rules and commands take effect immediately in new Claude Code sessions. Skills are picked up automatically when Claude matches the skill description to your request.

---

## All Assets

### Skills

| Skill | Description |
|-------|-------------|
| `security-code-review` | Full SAST + manual security review with formal report |
| `security-iac-triage` | CVSS 4.0 triage grounded in IaC deployment context |
| `security-vibe-patch` | Minimal security patches, one commit per finding, PR |

### Commands

| Command | Description |
|---------|-------------|
| `/security-semgrep-setup` | Install Semgrep Pro and run a local scan |
| `/sessions` | List all Claude Code sessions with resume commands |

### Rules

Rules are behavioral guidelines applied globally to all sessions where this toolbelt is deployed.

| Rule | What it enforces |
|------|-----------------|
| `beads` | Task tracking via `bd` (Beads) — no markdown TODOs |
| `ci-cd` | SAST gate placement, blocking policy, artifact retention |
| `code-review` | Severity labels, what to flag, how to write review comments |
| `documentation` | Docstrings, ADRs, runbooks, README standards |
| `git` | Conventional commits, branch naming, PR structure |
| `python` | Style, type annotations, exception handling, testing |
| `sast-rules` | Rule anatomy, severity definitions, test corpus requirements |
| `security` | Secrets handling, input validation, dependency hygiene |
| `testing` | Framework choices, coverage thresholds, test structure |

---

## Author

Adrian Puente Z. ([@ch0ks](https://github.com/ch0ks)) — [hackarandas.com](https://hackarandas.com)

## License

MIT — see [LICENSE](LICENSE).
