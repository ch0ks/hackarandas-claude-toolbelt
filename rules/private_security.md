# Security Rules

## Secrets & Credentials

**Never:**
- Hardcode API keys, passwords, tokens, or secrets in source code
- Commit `.env` files containing real credentials
- Log secrets, tokens, or PII — even at DEBUG level
- Store credentials in comments, docstrings, or test fixtures
- Put secrets in URLs, query strings, or request headers you log

**Always:**
- Load secrets from environment variables or a secrets manager
- Add `.env` to `.gitignore` immediately when creating a project
- Use `.env.example` with placeholder values to document required variables
- Rotate any secret that was accidentally committed — treat it as compromised

If you ever spot a hardcoded secret in code you're reviewing or editing, **stop and flag it immediately** before doing anything else.

---

## Environment Variables

```python
# Good
import os
api_key = os.environ["STRIPE_API_KEY"]  # Fails loudly if missing

# Bad
api_key = "sk_live_abc123..."
api_key = os.environ.get("STRIPE_API_KEY", "sk_live_abc123...")  # Default is a secret!
```

- Prefer `os.environ["KEY"]` over `os.environ.get("KEY")` for required secrets — fail fast, fail loud
- Use `python-dotenv` or equivalent only in local dev; never in production
- Document all required env vars in `README.md` and `.env.example`

---

## Dependencies

- Pin dependency versions in production (`requirements.txt`, `package-lock.json`, `go.sum`)
- Run audit tools regularly:
  ```bash
  pip audit                    # Python
  npm audit                    # Node
  go mod verify                # Go
  ```
- Never install packages from unknown or untrusted sources
- Review what a new dependency pulls in transitively before adding it
- Prefer smaller, well-maintained packages over large ones with many transitive deps
- Check last commit date and open issue count before adding a new dependency

---

## Input Validation

- Validate and sanitize all user input at the boundary — never trust external data
- Use parameterized queries for all database operations — never string-interpolate SQL
- Validate file uploads: type, size, and content — never trust the `Content-Type` header alone
- Reject unexpected fields in API payloads rather than silently ignoring them

---

## General

- Use HTTPS everywhere — never downgrade to HTTP, even internally
- Apply least-privilege principle: request only the permissions actually needed
- Set secure, `HttpOnly`, and `SameSite` flags on cookies
- Use constant-time comparison for secrets (`hmac.compare_digest`, not `==`)
- Never disable SSL certificate verification, even in tests
- When in doubt, default to more restrictive — it's easier to relax than to tighten

---

## When Reviewing Code

Flag immediately (block the PR):
- Any hardcoded secret or credential
- SQL/command injection risk
- Auth bypass or missing authorization check
- Sensitive data logged or returned in API responses

Flag as warning (discuss before merging):
- Unpinned dependency versions
- Missing input validation on external data
- Overly broad permissions requested
