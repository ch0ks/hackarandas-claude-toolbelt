# Testing Rules

## Framework Preferences

| Language   | Framework       | Runner    | Coverage          |
|------------|-----------------|-----------|-------------------|
| Python     | `pytest`        | `pytest`  | `pytest-cov`      |
| JavaScript | `vitest`        | `vitest`  | `@vitest/coverage-v8` |
| TypeScript | `vitest`        | `vitest`  | `@vitest/coverage-v8` |
| Go         | `testing` (std) | `go test` | `go test -cover`  |

Use the framework already present in the project. Never introduce a second test framework.

---

## Coverage Thresholds

| Code type | Minimum | Target |
|-----------|---------|--------|
| Security-critical (auth, crypto, input validation, taint sinks) | **95%** | 100% |
| Core business logic and shared utilities | **90%** | 95% |
| General application code | **80%** | 90% |
| Generated boilerplate, migrations, auto-generated clients | exempt | exempt |

Enforce in CI:
```bash
# Python
pytest --cov=src --cov-report=term-missing --fail-under=80

# Go
go test -cover ./...
```

Coverage is a floor, not a goal. 80% with meaningful tests beats 95% with assertion-free
snapshot tests. Coverage that does not assert behavior is worthless.

---

## When to Write Tests

**Always — no exceptions:**
- Every public function or method
- Every bug fix — write the failing test first, then fix
- Every security control (auth, authz, input validation, crypto, rate limiting)
- Any function on a taint sink or sanitizer path
- Every SAST detection rule (see SAST Rule Testing below)

**Write before the fix (TDD):**
- Fixing a regression — the test encodes what broke
- Implementing a detection rule — FP cases before TP cases
- Implementing a well-defined algorithm with clear acceptance criteria

**Skip tests for:**
- Trivial one-liner property accessors with no logic
- Generated boilerplate and auto-generated clients
- Pure configuration objects with no branching

---

## Test Structure

### Naming

```
test_<function>_<scenario>_<expected_outcome>
```

```python
test_verify_token_expired_raises_invalid_token_error
test_sanitize_sql_input_with_injection_payload_strips_unsafe_chars
test_taint_sink_called_with_user_input_triggers_finding
test_taint_sink_called_after_sanitizer_no_finding
```

### Layout — AAA

```python
def test_verify_token_expired_raises() -> None:
    # Arrange
    expired_token = build_token(expires_delta=timedelta(seconds=-1))

    # Act / Assert
    with pytest.raises(InvalidTokenError, match="Token has expired"):
        verify_token(expired_token, secret=TEST_SECRET)
```

Always use `pytest.raises` with `match=` — assert on the exception message, not just the type.

---

## Test Categories & Directory Layout

```
tests/
├── unit/               # Fast, isolated, no I/O — run on every save
│   ├── test_auth.py
│   ├── test_sanitizers.py
│   └── ...
├── integration/        # Real deps (DB, HTTP) — run on PR
│   ├── test_pipeline.py
│   └── ...
├── security/           # Security-specific test suites
│   ├── test_injection.py       # Input validation / injection resistance
│   ├── test_auth_bypass.py     # Auth control tests
│   └── ...
└── rules/              # SAST rule corpus (see below)
    ├── <rule-id>/
    │   ├── tp_*.py
    │   ├── fp_*.py
    │   └── README.md
    └── ...
```

Mark categories with pytest markers:
```python
@pytest.mark.unit
@pytest.mark.integration
@pytest.mark.security
@pytest.mark.rules
```

Run selectively:
```bash
pytest -m unit                    # Fast feedback loop
pytest -m "unit or security"      # Pre-commit
pytest -m integration             # CI only
```

---

## SAST Rule Testing

Every detection rule requires a dedicated test corpus in `tests/rules/<rule-id>/`.
This is non-negotiable — see `rules/sast-rules.md` for the full requirement.

### Corpus Structure

```
tests/rules/sg-CWE089-sqli-format-string/
├── tp_1_basic_format_string.py       # Must trigger — direct sink via % format
├── tp_2_fstring_interpolation.py     # Must trigger — f-string variant
├── tp_3_concatenation.py             # Must trigger — string concatenation
├── fp_1_parameterized_query.py       # Must NOT trigger — safe API used
├── fp_2_hardcoded_literal.py         # Must NOT trigger — no user input
├── fp_3_sanitized_input.py           # Must NOT trigger — sanitizer in taint path
└── README.md                         # Taint path and what each case tests
```

Minimum: 2 TP + 2 FP. Target: 3+ TP covering variant patterns, 3+ FP covering
common legitimate usage that would produce a false positive.

### Rule Test Assertions

```python
# TP — must produce at least one finding
def test_rule_fires_on_format_string_injection(rule_runner) -> None:
    findings = rule_runner.run("sg-CWE089-sqli-format-string", "tp_1_basic_format_string.py")
    assert len(findings) >= 1
    assert findings[0].cwe == "CWE-089"
    assert findings[0].severity == "high"

# FP — must produce zero findings
def test_rule_silent_on_parameterized_query(rule_runner) -> None:
    findings = rule_runner.run("sg-CWE089-sqli-format-string", "fp_1_parameterized_query.py")
    assert findings == [], f"Unexpected findings: {findings}"
```

Always assert `findings == []` for FP cases — not `assert not findings`. The explicit
form produces clearer failure output.

### Regression Tests

When a rule produces a confirmed false positive in production:
1. Add the minimal reproducer as a new `fp_N_<slug>.py` in the rule corpus
2. Write a regression test asserting zero findings on that file
3. Reference the Beads issue ID in the test docstring
4. Fix the rule

This prevents the same FP from reappearing after rule updates.

---

## Security Control Tests

Security controls need adversarial test cases, not just happy-path coverage.
For each control, test:

| Control | Must test |
|---------|-----------|
| Input validation | Boundary values, null/empty, type confusion, oversized input, encoding variants |
| Authentication | Valid creds, invalid creds, expired tokens, malformed tokens, replay |
| Authorization | Correct role, wrong role, missing role, privilege escalation path |
| Sanitizers | Known payload variants for the vuln class (SQLi, XSS, path traversal, etc.) |
| Crypto | Known-bad inputs, key material validation, algorithm downgrade |
| Rate limiting | Under limit, at limit, over limit, distributed bypass patterns |

```python
@pytest.mark.security
@pytest.mark.parametrize("payload", SQLI_PAYLOADS)
def test_query_builder_rejects_injection_payloads(payload: str) -> None:
    with pytest.raises(ValidationError):
        build_query(user_input=payload)
```

Maintain `tests/security/payloads/` with curated payload lists per vulnerability class.
Reference them across tests rather than inlining them.

---

## Rules

- One logical assertion per test — multiple `assert` statements are fine if they test one behavior
- Never test implementation details — test observable behavior and security properties
- Tests must be deterministic — no `random`, no `sleep()`, no system time without mocking
- Mock at the boundary (HTTP clients, DB, filesystem) — never mock internal functions
- Fixtures over repeated setup code — scoped, minimal, explicit
- Never use production data or real credentials in tests — use factories or fixtures
- A test with no assertions is worse than no test — it creates false confidence
- Integration tests are opt-in via marker — never run by default in pre-commit

---

## Running Tests

```bash
# Unit only — fast feedback
pytest -m unit -x -q

# Unit + security — pre-commit
pytest -m "unit or security" -x -q

# Rule corpus only
pytest -m rules -v

# With coverage
pytest --cov=src --cov-report=term-missing --fail-under=80

# Full suite — CI
pytest --cov=src --cov-report=xml -x

# Go — unit
go test -short ./...

# Go — race detector (always in CI)
go test -race ./...
```

---

## Test Failure Output

A failing test must tell you immediately: what broke, what was expected, what was
received. If you need to read the source to understand a failure, the test is
under-specified.

```python
# Bad — failure says "assert False"
assert result

# Good — failure says exactly what went wrong
assert result == expected, (
    f"Rule {rule_id!r} fired on FP case {fp_file!r}.\n"
    f"Findings:  {result}\n"
    f"Expected:  no findings"
)
```

For security tests, failure messages must include the payload and the code path
that failed to reject it.
