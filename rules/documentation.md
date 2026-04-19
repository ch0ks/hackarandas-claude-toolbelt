# Documentation Standards

## Core Principle

Documentation is for the next person — often a developer who is not a security expert.
Write for clarity, not to demonstrate knowledge. If a developer has to re-read a sentence
twice, rewrite it.

---

## Docstrings

Use Google-style docstrings. See `rules/python.md` for the full format.

Security-specific additions for any function that:
- Handles authentication or authorization
- Processes untrusted input
- Performs cryptographic operations
- Makes outbound network calls
- Reads or writes sensitive data

...must include a `Security:` section:

```python
def verify_token(token: str, secret: str) -> dict:
    """Verify and decode a signed JWT token.

    Args:
        token: Raw JWT string from the Authorization header.
        secret: HMAC signing secret from environment config.

    Returns:
        Decoded payload as a dictionary if the token is valid.

    Raises:
        InvalidTokenError: If the token is expired, malformed, or the
            signature does not match.

    Security:
        Uses HS256. Secret must be at least 32 bytes of random data.
        This function does NOT validate the `aud` or `iss` claims —
        callers are responsible for validating application-specific claims.
        Do not log the token or secret at any log level.
    """
```

The `Security:` section should document: what trust assumptions the function makes,
what it does NOT protect against, and any caller responsibilities.

---

## ADRs — Architecture Decision Records

Create an ADR for every significant security architecture decision. "Significant" means:
a future engineer would reasonably ask "why did we do it this way?"

**File location:** `docs/adr/`
**Naming:** `ADR-<NNN>-<short-slug>.md` (e.g. `ADR-012-sast-taint-engine-choice.md`)
**Status values:** `proposed` · `accepted` · `deprecated` · `superseded-by-ADR-NNN`

### ADR Template

```markdown
# ADR-NNN: <Title>

**Date:** YYYY-MM-DD
**Status:** accepted
**Deciders:** <names or roles>

## Context

What problem are we solving? What constraints exist?
What made this decision non-trivial?

## Decision

What did we decide to do? State it clearly in one paragraph.

## Alternatives Considered

| Option | Pros | Cons | Reason rejected |
|--------|------|------|-----------------|
| Option A | ... | ... | ... |
| Option B | ... | ... | ... |

## Consequences

What becomes easier? What becomes harder?
What new risks does this introduce?
What must be true for this decision to remain valid?

## Security Implications

What are the security properties of this decision?
What threat model assumptions does it rely on?
What must be monitored or revisited if the threat model changes?
```

The `Security Implications` section is mandatory for all ADRs in a security engineering
context. It is optional for purely operational decisions.

---

## Runbooks

Runbooks are for humans under pressure. Write them for 2am incident conditions:
short sentences, numbered steps, no ambiguity.

**File location:** `docs/runbooks/`
**Naming:** `<trigger-event>.md` (e.g. `critical-sast-finding-in-release.md`)

### Runbook Template

```markdown
# Runbook: <Title>

**Trigger:** <What event or alert causes someone to open this runbook>
**Severity:** critical | high | medium
**Owner:** <team or role>
**Last reviewed:** YYYY-MM-DD

---

## Symptoms

- Bullet list of observable signs that this runbook applies.
- Be specific — include exact alert names, error messages, or metric thresholds.

## Immediate Actions (first 5 minutes)

1. Do this first.
2. Then this.
3. Do NOT do X — explain why briefly.

## Investigation Steps

1. Run: `<exact command>`
   Expected output: `<what you should see>`
   If you see something else: go to step N / escalate to <role>

2. Check: `<what to look at>`

## Resolution

### Scenario A: <most common cause>
Steps to resolve.

### Scenario B: <second most common cause>
Steps to resolve.

## Escalation

Escalate to <role> if:
- You cannot resolve within 30 minutes
- The issue affects production data
- <other specific condition>

## Post-Incident

- [ ] File a Beads issue for root cause fix
- [ ] Update this runbook if steps were inaccurate
- [ ] Schedule a blameless post-mortem if severity was critical
```

### Runbook Rules

- Every numbered step must be an action, not a description.
- Every command must be exact and copy-pasteable — no placeholders like `<your-value>`
  without a preceding step that defines what the value is.
- "Check the logs" is not a step. "Run `kubectl logs <pod> | grep ERROR | tail -50`" is a step.
- Review runbooks every 90 days or after any incident that required deviation from them.
- If a runbook was not used in 12 months, evaluate whether the scenario still exists.

---

## README Standards

Every tool, service, or rule set needs a README that answers in order:

1. **What is this?** — one sentence
2. **Why does it exist?** — the problem it solves
3. **How do I use it?** — quickstart, copy-pasteable
4. **How do I develop/contribute?** — setup, test, deploy
5. **Who owns it?** — team/contact

Security tools additionally need:
- Threat model or trust assumptions (what inputs are considered untrusted)
- Known limitations (what this does NOT detect or protect against)
- How to report a false positive or false negative

Keep READMEs under 200 lines. Anything longer belongs in `docs/`.

---

## What Not to Document

- How the code works — that's what the code is for
- Obvious things that any engineer would know
- Information that will be stale within a sprint (use Beads issues for that)
- Decisions that were never actually in question

Over-documentation is as harmful as under-documentation. It buries the signal.
