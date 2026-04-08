# Git Rules

## Commit Messages

Use Conventional Commits format — always:

```
<type>(<scope>): <short summary>

[optional body]

[optional footer: issue refs, breaking changes]
```

**Types:** `feat` · `fix` · `chore` · `docs` · `refactor` · `test` · `perf` · `ci`

Rules:
- Summary line: imperative mood, max 72 chars, no period at end
- Body: explain *why*, not *what* — the diff shows what
- If work is tracked in Beads, append the issue ID: `(bd-abc)`
- Never mention Claude or AI tooling in commit messages

**Good:**
```
feat(auth): add refresh token rotation

Tokens are now single-use to limit exposure window from stolen tokens.
Previous tokens are invalidated on first use rather than expiry.

(bd-a3f8)
```

**Bad:**
```
fixed stuff
updated code
WIP
```

---

## Branch Naming

```
<type>/<short-description>
```

Examples:
```
feat/oauth-refresh-tokens
fix/null-pointer-login
chore/upgrade-dependencies
docs/api-authentication
```

- Use hyphens, not underscores
- All lowercase
- Max 50 chars
- Never commit directly to `main` or `master`

---

## Pull Requests

- Title follows the same Conventional Commits format as commit messages
- Description must include: **What**, **Why**, and **How to test**
- Link all related Beads issues in the PR description
- Keep PRs small and focused — one concern per PR
- Squash merge preferred; preserve individual commits only for complex features
- Never force-push to shared branches

---

## General

- Always pull with rebase: `git pull --rebase`
- Run `git status` before and after any significant operation
- Never use `git add .` blindly — stage files explicitly
- `git stash` before switching context; `git stash clear` only after confirming work is safe
- Prefer `git diff --staged` before committing to review what's going in
