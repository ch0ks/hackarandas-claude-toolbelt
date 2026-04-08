# Security Policy

## Scope

This repository contains Markdown files — rules, commands, and skill definitions for Claude Code. There is no runtime code, server, or compiled binary.

Security-relevant issues include:

- A command or skill that could cause **data loss** or **destructive side effects** when followed
- A command or skill that **misleads the user** about what it does
- **Hardcoded credentials or sensitive values** in examples

Issues that are out of scope: theoretical risks from following the shell commands in a misconfigured environment, general Claude Code behavior not caused by these assets.

## Reporting a Vulnerability

Use [GitHub private vulnerability reporting](https://github.com/ch0ks/hackarandas-claude-toolbelt/security/advisories/new) to report a vulnerability confidentially.

If you prefer email: contact details are available at [hackarandas.com](https://hackarandas.com).

Please include:
- Which file contains the issue
- What the harmful behavior is
- A minimal reproduction scenario

## Response

Confirmed issues will be corrected promptly. Credit will be given in the fix commit unless you prefer to remain anonymous.
