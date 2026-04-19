# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A personal Claude Code toolbelt — a collection of reusable assets (rules, commands, skills) authored by Adrian Puente Z. (@ch0ks). All content is Markdown. There is no build system, no tests, and no runtime code.

## Directory Structure

```
rules/      → Global rule files injected into Claude\'s context via ~/.claude/rules/
commands/   → Custom slash command definitions (/command-name)
skills/     → Claude Code skill definitions (SKILL.md per skill)
docs/       → Supporting repository documentation, guides, examples, and curation pages (create when useful)
```

### `rules/`

Each `*.md` file is a behavioral guideline that Claude reads at session start. These are **already deployed** to `~/.claude/rules/` — edits here must be synchronized manually.

Current rules: `beads` (task tracking via `bd`), `ci-cd`, `code-review`, `documentation`, `git`, `python`, `sast-rules`, `security`, `testing`.

### `commands/`

Each `.md` file defines a custom slash command. The filename becomes the command name (e.g. `sessions.md` → `/sessions`). Commands are plain Markdown with step-by-step shell instructions that Claude executes when the command is invoked.

Current command: `security-semgrep-setup.md` → `/security-semgrep-setup`

### `skills/`

Each subdirectory holds one skill. The skill is defined entirely in `SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-name
description: shown to Claude for skill selection and trigger matching
allowed-tools: Bash, Read, Write, Grep, Glob
author: Adrian Puente Z. (@ch0ks)
---
```

Current skills: `security-code-review`, `security-iac-triage`, `security-vibe-patch`

The body is a numbered step procedure Claude follows when the skill is triggered. Skills can invoke Semgrep, write reports to `security-review/`, and create git branches/PRs.

## Security Skill Pipeline

The three security skills are designed to run in sequence:

1. **`/security-semgrep-setup`** (command) — Installs Semgrep, logs in to Semgrep Pro, runs a scan, saves `security-review/semgrep-results-YYYYMMDD.json`
2. **`security-code-review`** (skill) — Consumes the JSON artifact, adds manual analysis, writes `security-review/security-code-review-report-YYYYMMDD.md`
3. **`security-iac-triage`** (skill) — Reads the code review report, scores findings with CVSS 4.0 using IaC context, appends triage to the existing report
4. **`security-vibe-patch`** (skill) — Reads the code review report, generates minimal patches per finding, commits one patch per finding, writes `security-review/security-vibe-patch-report-YYYYMMDD.md`, opens a PR

All security output lands in `security-review/` at the target repository root (never in this toolbelt repo).

## Authoring Conventions

**New command:** Create `commands/<name>.md`. Start with the author comment `<!-- author: Adrian Puente Z. (@ch0ks) -->`. Use numbered steps with fenced bash blocks. Degrade gracefully — every failure path must print what failed and why before stopping.

**New skill:** Create `skills/<name>/SKILL.md` with the YAML frontmatter block. The `description` field is critical — it controls when Claude auto-selects the skill. List only the tools the skill actually uses in `allowed-tools`.

**New rule:** Create `rules/<topic>.md`. Rules are imperative — state what Claude must do, not what it should consider. Deploy to `~/.claude/rules/` after editing.

## Task Tracking

All work in this repo is tracked with `bd` (Beads). See `rules/private_beads.md` for the full workflow. Never use markdown task lists.

## Portfolio and Community Goals

This repository is both a personal toolbelt and a public portfolio. When making changes:

- **Prioritize curation** over expansion. Fewer polished assets beat many rough ones.
- **Make it discoverable**. A visitor should understand the repo in 60 seconds.
- **Enable contributions**. Help maintain `CONTRIBUTING.md`, issue templates, and PR templates.
- **Showcase security expertise**. Keep the security workflow as the flagship example.
- **Preserve attribution**. Follow the repository license exactly.
- **Keep it professional**. Avoid hype, vague claims, or undocumented experiments.

## Expected Top-Level Files

Maintain these when useful:
- `README.md` — clear quickstart + curated showcase
- `CONTRIBUTING.md` — how to propose new assets
- `CODE_OF_CONDUCT.md` — community standards
- `SECURITY.md` — vulnerability reporting
- `.github/ISSUE_TEMPLATE/` — skill submission, bug reports
- `.github/PULL_REQUEST_TEMPLATE.md` — quality checklist

## Style Standards

- Use lowercase kebab-case for all filenames
- Write concisely but completely
- Prefer examples over abstraction
- State assumptions and limitations
- Make side effects obvious
- Keep tone practical and credible

## When in Doubt

Choose the option that makes the repository:
- easier to use
- easier to contribute to
- better as a portfolio
- more consistently structured

