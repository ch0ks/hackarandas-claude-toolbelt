# Code Review Rules

## Approach

Be a thoughtful senior engineer, not a linter. The goal is to improve the code and share knowledge — not to find fault. Be direct, be specific, and always explain *why*.

---

## Severity Levels

Use these consistently so the author knows what must change vs. what is optional:

| Label | Meaning |
|-------|---------|
| `[BLOCK]` | Must fix before merge — correctness, security, data loss risk |
| `[WARN]` | Should fix — tech debt, maintainability, performance concern |
| `[NIT]` | Optional — style, naming, minor readability |
| `[QUESTION]` | Needs clarification before you can assess |
| `[PRAISE]` | Something done particularly well — call it out |

---

## Always Flag (`[BLOCK]`)

- Hardcoded secrets, credentials, or PII
- SQL injection, command injection, or XSS vectors
- Missing auth/authorization checks
- Data loss scenarios (missing transaction, silent swallow of errors)
- Race conditions or concurrency bugs
- Logic errors that produce wrong results
- Broken or missing tests for changed behavior
- API contract breakage without a migration path

---

## Flag as Warning (`[WARN]`)

- Functions over 40 lines that could be decomposed
- Deeply nested conditionals (more than 3 levels)
- Missing docstrings on public functions
- Inconsistent error handling (mix of returning errors and raising exceptions)
- Duplicate logic that should be extracted
- Missing type annotations on public functions (Python/TypeScript)
- Unpinned dependency versions
- `TODO` comments with no associated issue ID

---

## Skip or Minimize (`[NIT]` only)

- Whitespace and formatting — if a formatter (Black, Prettier, gofmt) handles it, don't comment
- Naming preferences that are purely subjective
- Style choices consistent with the rest of the file
- Code that is not in scope of the current change

Do not pile on nits. Pick the top 1–2 that genuinely improve readability. More than 3 nits per review is noise.

---

## What to Look For (in order)

1. **Correctness** — does it do what it claims? Are edge cases handled?
2. **Security** — any of the `[BLOCK]` security concerns above?
3. **Tests** — are changed behaviors tested? Do existing tests still make sense?
4. **Design** — is this the right abstraction? Does it fit the existing architecture?
5. **Readability** — will the next person understand this in 6 months?
6. **Performance** — only flag if there's a real, measurable concern, not theoretical

---

## How to Write Comments

Be specific and constructive. Explain the problem and suggest a fix:

**Good:**
```
[WARN] This function queries the database inside a loop. For a list of N items,
this is N+1 queries. Consider fetching all items in one query before the loop.
```

**Bad:**
```
This is inefficient.
```

For `[BLOCK]` items, always explain the risk:
```
[BLOCK] This compares tokens with `==`, which is vulnerable to timing attacks.
Use `hmac.compare_digest(a, b)` instead for constant-time comparison.
```

---

## What Not to Do

- Don't rewrite code just because you'd write it differently
- Don't comment on things the formatter will fix automatically
- Don't leave a review without at least one `[PRAISE]` if something was done well
- Don't block on `[NIT]` items — they're optional by definition
- Don't review tests less carefully than production code — tests are documentation
