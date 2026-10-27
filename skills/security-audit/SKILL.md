---
name: Security Audit
description: "Security review of application code, configuration, and dependencies. Finds secrets in code, injection flaws, broken authorization, SSRF, unsafe deserialization, insecure defaults, and vulnerable dependencies, and reports each verified issue with severity, location, and a fix. Use when the user asks for a security audit, vulnerability scan, or whether code is safe to ship."
license: MIT
---

# Security Audit

Find real, exploitable weaknesses and report them with enough evidence that someone can act on them. Verified findings matter more than long lists.

## Rules of engagement

- Analyze only code and configuration the user has access to and asked you to review.
- Do not run exploits against live systems. Demonstrate issues by reading code paths, or by running tests in a local environment.
- Never print full secret values. Redact them to the first four characters.
- Do not install security tools without asking. Use tools that are already present.

## 1. Define scope and threat model

Write 3 to 5 bullets before scanning:

- **Entry points**: HTTP routes, GraphQL resolvers, CLI arguments, file uploads, webhooks, message consumers, cron jobs.
- **Trust boundaries**: where data moves from untrusted (users, third parties) to trusted (database, shell, internal services).
- **Assets**: credentials, personal data, payments, admin functions.
- **Auth model**: how users authenticate, how roles are checked, where authorization happens.

## 2. Automated sweep

Run these only when the relevant files exist. Skip anything not present.

```sh
# Secrets: common key prefixes and private key headers
grep -rInE "AKIA[0-9A-Z]{16}|-----BEGIN [A-Z ]*PRIVATE KEY-----|ghp_[A-Za-z0-9]{36}|sk-[A-Za-z0-9]{20,}|xox[baprs]-" --exclude-dir={node_modules,.git,dist,vendor} .

# Dependency audit, chosen by lockfile
npm audit --omit=dev            # package-lock.json or pnpm/yarn equivalents
pip-audit                       # requirements.txt or pyproject.toml
cargo audit                     # Cargo.lock
govulncheck ./...               # go.sum
```

Treat tool output as leads, not findings. Confirm each one by reading the code. Pattern hints overmatch: test helpers often share names with dangerous calls (for example, a helper named `exec`). Before reporting an injection finding, confirm the call reaches a real sink, such as an import of `child_process` or a query built from input.

## 3. Manual review by category

Work through the categories in `references/owasp-checklist.md`. Prioritize:

1. Authorization on every state-changing or data-returning endpoint, including object-level checks.
2. Injection into SQL, shell, templates, LDAP, and HTML output.
3. Secrets and sensitive data in source, configuration, logs, and error responses.
4. Input handling for file paths, URLs, and deserialization.
5. Files and shared state: predictable temporary or lock file names, symlinks followed on write, file permissions on sensitive data, and checks that race with later use.
6. Authentication flows: session handling, password reset, token expiry, rate limiting on sensitive actions.
7. Defaults: debug mode, permissive CORS, disabled TLS verification, wildcard permissions.

## 4. Verify each finding

For every candidate issue, trace the path:

1. **Source**: where untrusted data enters.
2. **Path**: which functions it passes through and whether any of them validates or encodes it.
3. **Sink**: where it is used dangerously.

Assign a confidence level:

- **Confirmed**: you traced source to sink and can describe a concrete exploit.
- **Likely**: the path is clear, but a condition you could not check (config, middleware, network) decides exploitability.
- **Discarded**: you could not support it. Do not report it.

## 5. Report

Lead with a summary: number of findings by severity, and the most urgent item.

Use this table for findings:

| ID | Severity | Confidence | CWE | Location | Issue and exploit | Fix |
| -- | -------- | ---------- | --- | -------- | ----------------- | --- |
| SEC-1 | High | Confirmed | CWE-639 | `api/orders.ts:58` | Any logged-in user can read any order by changing `orderId`. No owner check. | Filter by `userId` from the session in the query. |

Severity scale:

- **Critical**: unauthenticated remote compromise, exposed production credentials, or mass data exposure.
- **High**: authenticated privilege escalation, injection, or access to other users' data.
- **Medium**: exploitable with unusual conditions, or significant defense-in-depth gap.
- **Low**: hardening issue with limited impact.

Then add a **Not checked** section listing what you could not review (for example, infrastructure, runtime configuration, third-party services). Silence about an area is not a clean result.

## 6. If a secret is exposed

Say clearly that the secret must be considered compromised. Recommend rotating it first, then removing it from the code. Note that deleting a secret from the current commit does not remove it from git history. Suggest a history rewrite only with the user's explicit approval, and warn that every clone must be updated.
