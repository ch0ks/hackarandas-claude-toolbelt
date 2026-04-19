# SAST Rule Writing Standards

## Rule Anatomy — Required Fields

Every detection rule must have all of the following before it ships:

```yaml
id:          <tool-prefix>-<cwe-id>-<short-slug>     # e.g. sg-CWE089-sqli-format-string
name:        <Human readable name>
description: <What this detects and why it matters — 2-3 sentences max>
cwe:         CWE-<id>                                 # Primary CWE mapping
cwe_refs:    [CWE-<id>, ...]                          # Related CWEs if applicable
severity:    critical | high | medium | low | info
confidence:  high | medium | low
tags:        [<language>, <framework>, <category>]
references:  [<CVE or advisory URL if applicable>]
```

No rule merges without every field populated. "TODO" is not a valid value.

---

## Severity Definitions

| Severity | Criteria |
|----------|----------|
| **Critical** | Direct RCE, auth bypass, or mass data exfiltration. Exploitable with no preconditions. |
| **High** | SQLi, SSRF, XXE, command injection, broken crypto. Exploitable with low preconditions. |
| **Medium** | Stored XSS, path traversal, insecure deserialization, privilege escalation. Requires some context. |
| **Low** | Info disclosure, verbose errors, weak (not broken) crypto, missing security headers. |
| **Info** | Hygiene issues, deprecated APIs, patterns worth tracking but not exploitable alone. |

Confidence reflects FP likelihood, not severity. A high-confidence medium is better than a low-confidence high.

---

## Test Case Requirements

Every rule requires **at minimum**:
- **2 true positive (TP) samples** — code that must trigger the rule
- **2 true negative / false positive (FP) samples** — code that must NOT trigger the rule

Structure test cases as code files in `tests/rules/<rule-id>/`:

```
tests/rules/sg-CWE089-sqli-format-string/
├── tp_1_basic_format_string.py       # Direct sink via % formatting
├── tp_2_fstring_interpolation.py     # f-string variant
├── fp_1_parameterized_query.py       # Parameterized — must not fire
├── fp_2_hardcoded_literal.py         # No user input in query — must not fire
└── README.md                         # Explain what each case tests and why
```

Test cases must be minimal and self-contained. Do not use real production code as test cases.

---

## Taint Modeling

Explicitly document sources, sinks, and sanitizers for every data-flow rule.

**Sources** — where untrusted data enters:
- HTTP request parameters, headers, cookies, body
- File reads from user-controlled paths
- Database results (when used as input to another sink)
- Environment variables that may be attacker-influenced
- Deserialized data

**Sinks** — where untrusted data causes harm:
- SQL query construction
- Shell command execution
- File system writes/reads
- HTML rendering (XSS)
- Deserialization entry points
- SSRF-prone HTTP client calls
- Cryptographic key/IV material

**Sanitizers** — what breaks the taint:
- Parameterized query APIs
- Shell escaping functions (`shlex.quote`)
- HTML encoding (`markupsafe.escape`, `html.escape`)
- Path canonicalization + allowlist validation
- Schema validation libraries (`pydantic`, `marshmallow`)

Document sanitizers that are **incomplete or bypassable** — these are common FP sources.

---

## Syntactic vs. Semantic Analysis

Choose the right approach deliberately. Document the choice in the rule.

**Use syntactic (pattern-matching) when:**
- The vulnerability is detectable from local structure alone
- No dataflow tracking is needed
- The pattern has high signal and low noise at the AST level
- Speed is critical (e.g., pre-commit hooks)
- Examples: deprecated function calls, hardcoded string literals, dangerous flag combinations

**Use semantic (dataflow/taint) when:**
- The vulnerability requires tracking data across function boundaries
- Sanitizers exist that break the taint path
- The sink alone is not sufficient signal (too many FPs)
- Examples: SQLi, XSS, SSRF, path traversal, command injection

**Never use regex on source code** for security rules. Use AST-level patterns at minimum.
Document in the rule which analysis tier it uses and why.

---

## Precision-First Rule Development

Ship in this order — do not skip steps:

1. **Define the vulnerability class** — CWE, attack scenario, affected code patterns
2. **Write FP test cases first** — forces you to think about legitimate usage
3. **Write TP test cases** — cover variants (direct, indirect, aliased)
4. **Implement the rule** — target the TP cases, validate against FP cases
5. **Pilot on one internal codebase** — measure raw TP/FP ratio before rollout
6. **Set confidence based on pilot results:**
   - FP rate < 5% → `high`
   - FP rate 5–20% → `medium`
   - FP rate > 20% → rework or set `info` with a note
7. **Document known FP patterns** in the rule metadata

A rule with 40% FP rate does not ship. It gets reworked or scoped down.

---

## Rule Review Checklist

Before merging a new or modified rule:

- [ ] All required metadata fields populated
- [ ] CWE mapping verified against official CWE list
- [ ] Minimum 2 TP + 2 FP test cases present and passing
- [ ] Taint sources, sinks, and sanitizers documented (dataflow rules)
- [ ] Syntactic vs. semantic choice documented and justified
- [ ] Pilot FP rate measured and recorded
- [ ] Severity and confidence are consistent with definitions above
- [ ] Remediation guidance is actionable for a developer unfamiliar with the vuln class
- [ ] Rule description does not assume security knowledge — explain the risk plainly

---

## Remediation Guidance Standards

Every rule's remediation must:
- Show a vulnerable code snippet and a fixed code snippet side by side
- Name the specific safe API or pattern to use (not just "sanitize your input")
- Link to language/framework documentation for the safe pattern
- Note any edge cases where the fix is insufficient

Developers are the audience, not security engineers. Write accordingly.
