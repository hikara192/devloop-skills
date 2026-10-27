# Code Review Checklist

Use this as a memory aid. Only raise an item when the diff actually touches the relevant area.

## Correctness

- [ ] Every branch of new conditionals is reachable and correct.
- [ ] Boundary values are handled: empty collections, zero, negative numbers, max sizes.
- [ ] Null, undefined, and missing keys are handled where the input can be absent.
- [ ] Async code awaits every promise. No fire-and-forget work that should be awaited.
- [ ] Errors are propagated or handled. Nothing is caught and silently ignored.
- [ ] Time zones, locale, and floating-point arithmetic are handled correctly for money or dates.
- [ ] Resource cleanup happens on every path (files, connections, locks, timers).

## Security

- [ ] All external input is validated at the boundary: HTTP params, headers, files, queue messages, env vars.
- [ ] Queries are parameterized. No string concatenation into SQL, shell commands, or HTML.
- [ ] File paths built from input are normalized and confined to an allowed directory.
- [ ] New endpoints enforce authentication and authorization, including object-level checks (user A cannot read user B's record).
- [ ] No secrets, tokens, or personal data in code, logs, error messages, or URLs.
- [ ] Outbound requests built from user input are restricted (SSRF).
- [ ] Cookies and tokens use secure defaults (httpOnly, secure, sameSite, short expiry).

## Data integrity and concurrency

- [ ] Multi-step writes are in a transaction or are idempotent.
- [ ] Read-modify-write sequences are protected against races (conditional updates, locks, optimistic versioning).
- [ ] Retries cannot create duplicates (idempotency keys or unique constraints).
- [ ] Migrations are backward compatible with the currently deployed code, or the deploy order is documented.
- [ ] Deletions are safe: soft delete where needed, cascade rules reviewed.

## Compatibility

- [ ] Public function signatures, API response fields, and event payloads are unchanged, or the change is versioned and documented.
- [ ] New required config values have defaults or are documented.
- [ ] Removed features have a deprecation path.

## Performance

- [ ] No database query inside a loop (N+1). Look for `await` inside `for` or `map`.
- [ ] Lists coming from the database or external APIs are paginated or bounded.
- [ ] New query filters or sorts are covered by an index.
- [ ] Large payloads are streamed or chunked instead of loaded entirely into memory.
- [ ] Caching is considered where the same expensive call repeats per request. Cache invalidation is correct.

## Observability

- [ ] Failure paths log enough context to debug (IDs, operation name) without logging sensitive data.
- [ ] New background jobs or endpoints have metrics or logs consistent with the rest of the codebase.

## Tests

- [ ] The changed behavior has at least one test that fails without the change.
- [ ] Tests assert specific outcomes, not only that the code runs.
- [ ] Edge cases from the correctness section have tests.
- [ ] Tests do not depend on wall-clock time, random values, or execution order without control.
