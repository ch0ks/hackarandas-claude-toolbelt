# Contributing

Contributions are welcome. This is a curated toolbelt — fewer polished assets beat many rough ones.

## What belongs here

**Skills** — Reusable task definitions that Claude executes on request. A good skill has a clear trigger, produces a concrete artifact (a report, a branch, a file), and degrades gracefully when its prerequisites are missing.

**Commands** — Step-by-step shell procedures invoked as `/slash-commands`. A good command is stateless, idempotent where possible, and prints exactly what it did or why it failed.

**Rules** — Behavioral guidelines that apply globally. A good rule is imperative, specific, and explains the *why* when it isn't obvious.

## Quality bar

Before submitting:

- [ ] **One clear purpose.** The asset does one thing and its name reflects it.
- [ ] **Frontmatter is complete.** Skills require `name`, `description`, `allowed-tools`, and `author`.
- [ ] **Graceful degradation.** Every failure path prints what failed, why, and how to recover. Never fail silently.
- [ ] **Attribution included.** Commands include the author HTML comment. Skills include the `author` frontmatter field.
- [ ] **No hardcoded assumptions.** No hardcoded paths, usernames, or project-specific logic.
- [ ] **Tested manually.** Run the skill or command against a real repository before submitting.

## What not to submit

- Experimental or incomplete assets — finish them first
- Assets that duplicate existing ones without clear improvement
- Assets with opinionated defaults that aren't documented or configurable
- Generic productivity tools unrelated to security or development workflow

## How to submit

1. Fork the repository.
2. Add your asset in the correct directory (`skills/<name>/SKILL.md`, `commands/<name>.md`, or `rules/<topic>.md`).
3. Follow the naming and authoring conventions in [CLAUDE.md](CLAUDE.md).
4. Open a pull request. Describe what the asset does, what it produces, and how you tested it.

## Author format

All assets must include attribution in the standard format:

- **Commands:** `<!-- author: Your Name (@handle) — yoursite.com -->`
- **Skills frontmatter:** `author: Your Name (@handle) — yoursite.com`
