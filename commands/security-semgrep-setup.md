<!-- author: Adrian Puente Z. (@ch0ks) — www.hackarandas.com -->
Check if Semgrep is installed and working. If not, install it using the best available method. Then log in to Semgrep Pro, enable the Pro engine, and run a local Pro scan in the current directory.

---

## Step 1 — Check if Semgrep is Already Installed

```bash
semgrep --version 2>/dev/null
```

If this succeeds, print the version and skip to Step 3 (login check).
If this fails, proceed to Step 2.

---

## Step 2 — Install Semgrep

Try each method in order. Move to the next only if the current one fails.

### Method 1 — Homebrew

```bash
# Check if brew is available
which brew 2>/dev/null
```

If brew is available:
```bash
brew install semgrep
```

If this succeeds, verify:
```bash
semgrep --version
```

Print: `✅ Semgrep installed via Homebrew` and skip to Step 3.
If brew is not available or `brew install` fails, try Method 2.

---

### Method 2 — pip (system)

```bash
# Check if python3 is available
which python3 2>/dev/null
```

If python3 is available:
```bash
python3 -m pip install semgrep
```

If this succeeds, verify:
```bash
semgrep --version
```

Print: `✅ Semgrep installed via pip` and skip to Step 3.
If python3 is not available or pip install fails, try Method 3.

---

### Method 3 — pip (user install)

```bash
python3 -m pip install --user semgrep
```

If this succeeds:
```bash
# Add user bin to PATH for this session if needed
export PATH="$PATH:$(python3 -m site --user-base)/bin"
semgrep --version
```

Print: `✅ Semgrep installed via pip --user`

If this also fails, print the following and stop:

```
❌ Semgrep installation failed.

Tried:
  1. brew install semgrep            — brew not available or failed
  2. python3 -m pip install semgrep  — failed
  3. python3 -m pip install --user   — failed

Manual options (macOS):   brew install semgrep
Manual options (Linux):   pipx install semgrep
Manual options (any):     npm install -g @semgrep/semgrep
Full docs:                https://semgrep.dev/docs/getting-started/

Skipping Semgrep Pro setup. Run /security-semgrep-setup again after resolving the install issue.
```

Do NOT proceed further if Semgrep could not be installed.

---

## Step 3 — Check Login Status
```bash
# Check for existing token in Semgrep settings file
token=$(grep "api_token" ~/.semgrep/settings.yml 2>/dev/null | awk '{print $2}' | tr -d '"')
```

If `$token` is non-empty, print: `✅ Already logged in (token found)` and skip to Step 4.

If `$token` is empty:
```bash
semgrep login
```

After the browser login flow completes, verify:
```bash
grep -q "api_token" ~/.semgrep/settings.yml 2>/dev/null
```

If the token now exists, print: `✅ Logged in to Semgrep Pro`

If still missing, print:
```
⚠️  Semgrep login failed or was skipped.

The Pro engine requires a Semgrep account.
Run `semgrep login` manually when ready, then re-run /semgrep-setup.

Continuing with OSS engine only — Pro taint analysis will not be available.
```

Continue to Step 4 regardless.

---

## Step 4 — Enable Semgrep Pro Engine

```bash
semgrep install-semgrep-pro 2>/dev/null
```

If this succeeds, print: `✅ Semgrep Pro engine installed`

If this fails, print:
```
⚠️  Could not install Semgrep Pro engine.

This usually means:
  - You are not logged in (complete Step 3 first)
  - Your account does not have Pro access

Continuing with OSS engine. Re-run /semgrep-setup after logging in.
```

Continue to Step 5 regardless.

---

## Step 5 — Run Local Semgrep Scan

### 5a. Prepare the output directory

```bash
# Use the git repo root if available, otherwise the current directory
REPO_ROOT=$(git rev-parse --show-toplevel 2>/dev/null || pwd)
SECURITY_REVIEW_DIR="$REPO_ROOT/security-review"
mkdir -p "$SECURITY_REVIEW_DIR"
DATE=$(date +%Y%m%d)
SEMGREP_JSON="$SECURITY_REVIEW_DIR/semgrep-results-${DATE}.json"
```

### 5b. Run the scan (human-readable output for the terminal)

```bash
semgrep scan --pro --config auto
```

Important:
- Use `semgrep scan`, NOT `semgrep ci` — this keeps the scan local and avoids pushing results to Semgrep Cloud
- Run from the current working directory
- If `--pro` flag fails because the Pro engine is not installed, fall back to:

```bash
semgrep scan --config auto
```

And note in output: `Running OSS scan (Pro engine not available)`

### 5c. Save JSON artifact for /security-code-review

After the human-readable scan completes, run the same scan again with `--json` and save to the `security-review/` directory:

```bash
semgrep scan --pro \
  --config=p/default \
  --config=p/owasp-top-ten \
  --config=p/secrets \
  --json 2>/dev/null | tee "$SEMGREP_JSON" > /dev/null

echo "Semgrep results saved: $SEMGREP_JSON"
```

If the JSON scan fails (e.g. Pro not available), fall back:
```bash
semgrep scan \
  --config=p/default \
  --config=p/owasp-top-ten \
  --config=p/secrets \
  --json 2>/dev/null | tee "$SEMGREP_JSON" > /dev/null
```

The saved file uses the same filename convention (`semgrep-results-YYYYMMDD.json`) and directory (`security-review/`) that `/security-code-review` and `/security-iac-triage` expect.

---

## Step 6 — Print Summary

After the scan completes, print a summary:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Semgrep Setup Summary
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Installation:  ✅ semgrep x.y.z
Login:         ✅ Logged in | ⚠️  Not logged in (OSS only)
Pro engine:    ✅ Installed | ⚠️  Not available
Scan:          ✅ Complete  | ❌ Failed
Engine used:   Pro | OSS
JSON artifact: /path/to/security-review/semgrep-results-YYYYMMDD.json
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

To run a full security review using these findings:
  /security-code-review
```

---

## Failure Principles

- Never exit silently — always print what failed and why
- Always suggest a manual fix when a step fails
- Never use `semgrep ci` — local scans only
- Each step degrades gracefully — a failure in login or Pro engine does not block the scan
- Only a total install failure stops the command
