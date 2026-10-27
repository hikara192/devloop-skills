# Security Review Checklist

Based on the [OWASP Top 10](https://owasp.org/Top10/) and the [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/). Use the grep hints as starting points only.

## Broken access control (CWE-284, CWE-639, CWE-862)

- Does every route that changes or returns data check authentication?
- Does every object lookup check that the current user owns or may access that object (IDOR)?
- Are admin routes protected by a server-side role check, not only by hiding the UI?
- Can a user change their own role, ownership, or tenant ID through a request body?

Hints: `router\.(get|post|put|patch|delete)`, `@app\.route`, `@RequestMapping`, `findById\(`, `findOne\(`.

## Injection (CWE-89, CWE-78, CWE-79, CWE-94)

- SQL built with string concatenation or template literals containing input.
- Shell commands built from input (`exec`, `system`, `subprocess` with `shell=True`, backticks).
- `eval`, `new Function`, dynamic `import()` or `require()` with input.
- Unescaped output in HTML, or `dangerouslySetInnerHTML`, `v-html`, `|safe` in templates.
- Template injection in server-side rendering.

Hints: `exec\(`, `eval\(`, `shell=True`, `innerHTML`, `dangerouslySetInnerHTML`, `\$\{.*\}.*(SELECT|INSERT|UPDATE|DELETE)`.

These hints overmatch. Test helpers and wrappers often reuse names such as `exec`. Confirm the call reaches a real sink by checking imports (`child_process`, `os/exec`, `subprocess`) and the arguments.

## Sensitive data exposure (CWE-200, CWE-312, CWE-532)

- Hard-coded credentials, API keys, or connection strings in source or tests.
- Secrets or personal data written to logs, error messages, or analytics events.
- Stack traces returned to clients in production mode.
- `.env` files, backups, or private keys committed to the repository.

Hints: `password\s*=`, `secret\s*=`, `token\s*=`, `console\.log\(.*(user|token|password)`.

## Authentication failures (CWE-287, CWE-307, CWE-613)

- Passwords hashed with a slow, salted algorithm (bcrypt, scrypt, argon2), not MD5 or SHA-1 alone.
- Rate limiting or lockout on login, password reset, and OTP verification.
- Session tokens regenerated after login. Logout invalidates the server-side session.
- JWTs verify the signature and the algorithm, and check expiry and audience. Reject `alg: none`.
- Password reset tokens are single-use, short-lived, and not derived from predictable values.

Hints: `jwt\.decode\(` without `verify`, `md5\(`, `sha1\(` used for passwords, `algorithms` option missing.

## Server-side request forgery (CWE-918)

- Outbound HTTP requests whose URL comes from user input.
- No allowlist of hosts, or no block on private and link-local ranges (127.0.0.0/8, 10.0.0.0/8, 169.254.169.254).
- Redirects followed without re-validating the destination.

Hints: `fetch\(`, `axios\.`, `requests\.get\(`, `http\.get\(`, `urllib`.

## Insecure deserialization and file handling (CWE-502, CWE-22, CWE-434)

- `pickle.loads`, `yaml.load` without a safe loader, Java native deserialization on untrusted data.
- File paths built from input without normalization. Check for `..` traversal and absolute path injection.
- Uploads that trust the client-provided content type or extension. Check size limits and storage location.

Hints: `pickle\.load`, `yaml\.load\(`, `ObjectInputStream`, `path\.join\(.*req\.`, `open\(.*request`.

## Insecure file handling (CWE-59, CWE-367, CWE-732)

- Temporary, lock, or backup files written under predictable names (`<file>.tmp`, `<file>.lock`) in a directory another user can write to. An attacker can plant a symlink there, and the write follows it.
- Writes without an exclusive create flag (`O_EXCL`, `wx`) or without a random name, so an existing symlink or file is silently reused.
- Sensitive data written with default permissions (for example `0644` under a typical umask). Use `0600` for private user data.
- Check-then-use races: `exists()` followed by `open()`, or permission checks on a path that can be swapped before use.
- Lock files left behind after a crash, with no stale-lock handling or a clear error message.
- Predictable paths read from environment variables or config without checking the owning directory.

Hints: `\.tmp['"`]`, `\.lock['"`]`, `writeFile\(`, `writeFileSync\(`, `existsSync\(`, `chmod`, `umask`, `0o6?44`.

## Security misconfiguration (CWE-16, CWE-942)

- Debug mode or verbose errors enabled in production configuration.
- CORS `Access-Control-Allow-Origin: *` combined with credentials.
- TLS certificate verification disabled (`verify=False`, `rejectUnauthorized: false`).
- Default credentials, open admin consoles, or directory listing enabled.
- Security headers missing where the app serves HTML (CSP, HSTS, X-Content-Type-Options).

Hints: `verify=False`, `rejectUnauthorized`, `DEBUG\s*=\s*True`, `origin: '\*'`, `cors\(\)`.

## Vulnerable and outdated components (CWE-1104)

- Lockfile present and committed, so builds are reproducible.
- Known vulnerabilities reported by the dependency audit for production dependencies.
- Abandoned dependencies with no security maintenance.

## Cryptography (CWE-327, CWE-330)

- Custom cryptographic code instead of vetted libraries.
- Weak algorithms for security purposes (DES, RC4, MD5, SHA-1 for signatures).
- `Math.random()`, `random.random()`, or `rand()` used for tokens, keys, or IDs.

Hints: `Math\.random\(`, `random\.random\(`, `createCipher\(`, `DES`, `RC4`.

## Logging and monitoring (CWE-778)

- Security-relevant events (failed logins, permission denials, admin actions) are logged.
- Logs do not contain secrets or full personal data.
