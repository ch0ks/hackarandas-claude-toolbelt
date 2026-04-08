# Example: Security Review Pipeline Output

The reports below are real outputs from running the full pipeline against [Apache Azkaban](https://github.com/azkaban/azkaban), an open-source batch workflow job scheduler. Nothing is redacted — this is exactly what the skills produce.

---

## What was run

```
/security-semgrep-setup       → Semgrep Pro installed, scan saved
/security-code-review         → Full report written
/security-vibe-patch          → Patches committed, PR opened
```

---

## Reports

### 🔍 Security Code Review Report

**[View report →](https://github.com/RozulIO/azkaban/blob/security/vibe-patch-20260406/security-review/security-code-review-report-20260406.md)**

What it shows:
- 15 findings total (10 manual + 5 Semgrep Pro)
- Full OWASP Top 10 coverage across Java source files
- CWE-mapped findings with evidence, taint flow analysis, and remediation code
- Risk matrix and compliance impact (SOC 2, ISO 27001, GDPR)
- Verdict: **NOT APPROVED** for production without P0 remediation

---

### 🩹 Security Vibe Patch Report

**[View report →](https://github.com/RozulIO/azkaban/blob/security/vibe-patch-20260406/security-review/security-vibe-patch-report-20260406.md)**

What it shows:
- 4 `[BLOCK]` findings processed from the code review report
- 2 patched: XXE injection (CWE-611), hardcoded credentials (CWE-798)
- 2 skipped: path traversal and plaintext password storage — LOW confidence, require engineering judgment
- One commit per patch, Semgrep-verified clean after each
- PR description with full patch inventory and manual remediation backlog

---

## What the pipeline does NOT show here

The IaC triage stage (`/security-iac-triage`) was not run on this project because Azkaban's deployment configuration was not in scope. On a project with Terraform, Kubernetes, or Docker Compose files, this stage would add CVSS 4.0 scores grounded in the actual infrastructure topology — adjusting each finding's risk up or down based on observable deployment facts.
