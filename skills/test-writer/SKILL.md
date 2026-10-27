---
name: Test Writer
description: "Write meaningful unit, integration, and regression tests for existing code, matching the project's own test framework, file layout, and conventions. Use when the user asks for tests, test coverage, edge case tests, or a regression test for a bug fix."
license: MIT
---

# Test Writer

Write tests that describe behavior and would fail if that behavior broke. Follow the project's existing setup. Do not introduce a new framework.

## 1. Discover the test setup

Find out how the project already tests:

- Read the manifest and scripts: `package.json`, `pyproject.toml`, `go.mod`, `Cargo.toml`, `Makefile`, `pom.xml`, and similar.
- Find existing test files with `glob` (for example `**/*.test.*`, `**/*_test.*`, `**/test_*.py`, `**/tests/**`).
- Note the framework, runner command, file naming, folder location, fixture and factory helpers, and mocking style.

Mirror what you find. If the project has no tests, ask which framework to use before adding one.

## 2. Understand the target

- Read the function, class, or module under test in full.
- Identify its public contract: inputs, outputs, side effects, errors it throws or returns.
- Identify its dependencies. Decide which are external boundaries to mock (network, database, filesystem, clock, randomness) and which are simple collaborators to use as they are.

Test through the public interface. Do not test private helpers unless they hold logic that cannot be reached otherwise.

## 3. Plan the cases

Write a short table of cases before writing any test code:

| Category | Examples |
| -------- | -------- |
| Happy path | Typical valid input produces the expected output |
| Boundaries | Empty, one item, max size, zero, negative, exact limits |
| Invalid input | Wrong type, missing field, malformed format |
| Errors | Dependency fails, timeout, permission denied, not found |
| State | Repeated calls, order of operations, idempotency |
| Regression | The exact input that caused a reported bug |
| Concurrency and storage | Parallel calls on shared state, interrupted or partial writes, leftover temp or lock files, file permissions, symlinks at predictable paths |

Add the concurrency and storage rows whenever the code touches shared state, files, or locks. Those bugs rarely show up in single-call tests.

Show the table to the user when the scope is large. Otherwise proceed.

## 4. Write the tests

- One behavior per test. The test name states the behavior: `returns 404 when the order does not exist`, not `test_get_order_2`.
- Arrange, act, assert. Keep each section short.
- Assert specific values. Avoid assertions that only check the code did not throw.
- No conditionals, loops, or complex logic inside tests. Use parameterized tests for repeated cases.
- Make tests deterministic. Inject or freeze time and randomness.
- Use the project's fixtures and factories. Do not hand-build large objects in every test.
- Keep each test independent. Tests must pass in any order.

## 5. Run and verify

1. Run only the new tests first, then the full suite.
2. If a test fails, decide whether the test or the code is wrong.
   - If the test has a mistake, fix the test.
   - If the code has a bug, report it with the failing input. Do not change production code unless the user asked for the fix.
3. Confirm that the tests can fail. For one key behavior, temporarily break the implementation, run the test, and restore the code. Report the result. Make sure the restore is exact.

## Anti-patterns to avoid

- Testing the mock instead of the code (asserting only that a mock was called, with no check on the result).
- Snapshot tests for logic that should be asserted explicitly.
- Sleeping for fixed durations instead of waiting on a condition.
- Shared mutable state between tests.
- Tests that re-implement the function under test to compute the expected value.
- Mocking the module under test.
- Excessive mocking that leaves nothing real to test. Prefer a real in-memory database or a local test server when the project already has one.

## Final report

Summarize what you added: the number of tests, the file paths, which behaviors are now covered, the test command used, and the result. List any behavior you deliberately did not test and why.
