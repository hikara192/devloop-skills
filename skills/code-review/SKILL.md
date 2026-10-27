---
name: Code Review
description: "Structured code review of diffs, branches, staged changes, or pull requests. Use when the user asks to review code, a PR, a diff, or recent changes. Prioritizes correctness, security, data integrity, and performance over style, and reports each finding with severity and file:line references."
license: MIT
---

# Code Review

Review changes the way a senior engineer would: find what can break in production first, then what will hurt maintainability, and skip what tooling already handles.

## 1. Determine the scope

Pick the first option that applies:

- PR number or URL given: `gh pr view <n>` and `gh pr diff <n>`.
- Branch given: `git diff <base>...<branch>` (base is usually `main` or `master`).
- Otherwise: `git diff --staged`, then `git diff`.
- If both are empty and the repository has a default branch, compare `HEAD` with it.
- If there is no base to compare against (for example, the repository has one commit or no default branch), review the latest commit: `git show HEAD`. Read the full files it touches at `HEAD`, and state in the report that the scope is the latest commit.

If the scope is still ambiguous, ask one question before reviewing.

## 2. Read beyond the diff

A diff alone hides most bugs. For each changed function or type:

- Read the full file, not only the hunk.
- Find callers and implementations with `grep` and check that the change keeps their assumptions valid.
- Check related tests. Note whether the change is covered.

## 3. Review in priority order

1. **Correctness**: wrong conditions, off-by-one errors, null/undefined paths, incorrect async handling, lost errors, broken invariants.
2. **Security**: unvalidated input, injection (SQL, shell, template, path traversal), authz checks missing on new endpoints, secrets in code or logs, unsafe deserialization.
3. **Data integrity**: race conditions, non-atomic multi-step writes, missing transactions, migrations that lock or drop data, non-idempotent retries.
4. **Compatibility**: changed public signatures, API response shapes, config keys, or database columns that existing clients depend on.
5. **Performance**: N+1 queries, unbounded loops or lists, work inside hot paths, missing indexes for new query patterns.
6. **Error handling and observability**: swallowed exceptions, unhelpful error messages, missing logs on failure paths.
7. **Tests**: missing coverage for the changed behavior, tests that assert nothing, tests coupled to implementation details.
8. **Readability**: naming, duplication, dead code. Mention only when it hides a real risk or when it is severe.

The checklist in `references/checklist.md` covers the details of each category.

## 4. Verify before reporting

- Do not report a bug you have not traced. For each finding, be able to name the input or sequence that triggers it.
- If the project has a test or typecheck command and running it is safe, run it and include the result.
- Drop speculative findings. If something is uncertain, label it as a question instead of a defect.

## 5. Report

Start with a one-line verdict: **Approve**, **Approve with comments**, or **Request changes**.

Then list findings grouped by severity:

- **Blocker**: will cause a bug, data loss, or a security issue. Must fix before merge.
- **Major**: likely to cause problems soon or a significant maintainability cost.
- **Minor**: improvement that is worth doing but not required.
- **Nit**: optional polish. Keep these few.

Each finding follows this format:

```
[Major] src/orders/service.ts:142 - Race condition on stock decrement
Two concurrent requests can both read stock=1 and both succeed, which oversells the item.
Suggested fix: use a conditional update (`UPDATE ... SET stock = stock - 1 WHERE stock > 0`) and check the affected row count.
```

Finish with a short "What's good" note only if something in the change deserves it. Keep the whole report scannable. Cap the list at the 15 most important findings and say how many lower-priority items were omitted.

## Rules

- Review the change, not the whole codebase. Mention pre-existing problems separately and briefly.
- Do not request changes that only restate personal preference.
- Do not edit files while reviewing unless the user asks you to apply fixes.
