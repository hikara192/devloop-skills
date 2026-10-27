---
name: Conventional Commits
description: "Write commit messages and split changes into atomic commits that follow the Conventional Commits 1.0.0 specification. Use when the user asks to commit, write a commit message, clean up or split commits, or check whether a message follows the convention."
license: MIT
---

# Conventional Commits

Write commit messages that are readable in `git log`, parseable by release tooling, and explain why a change exists.

## Format

```
<type>(<optional scope>)<optional !>: <description>

<optional body>

<optional footers>
```

## Types

| Type | Use for | Release effect |
| ---- | ------- | -------------- |
| `feat` | A new user-visible capability | minor |
| `fix` | A bug fix | patch |
| `perf` | A change that improves performance | patch |
| `refactor` | Restructuring without behavior change | none |
| `docs` | Documentation only | none |
| `test` | Adding or fixing tests only | none |
| `style` | Formatting, whitespace, semicolons | none |
| `build` | Build system or dependency changes | none |
| `ci` | CI configuration | none |
| `chore` | Maintenance that fits no other type | none |
| `revert` | Reverts an earlier commit | depends on the reverted change |

Breaking changes add `!` after the type or scope, or a `BREAKING CHANGE:` footer. Both trigger a major release.

## Workflow

1. **Inspect the changes.** Run `git status` and `git diff --staged`. If nothing is staged, run `git diff` and decide what belongs together.
2. **Split if needed.** If the changes mix concerns (a feature plus an unrelated refactor plus a formatting pass), propose separate commits. Stage each group with `git add <paths>` or `git add -p`, and commit them one at a time.
3. **Pick the type.** Use the most specific type that fits. A bug fix that also adds a test is still `fix`.
4. **Write the description.** Use the imperative mood, lowercase first letter, no trailing period, and keep the first line under 72 characters.
5. **Write the body when the why is not obvious.** Explain motivation and trade-offs, not a restatement of the diff. Wrap lines at 72 characters.
6. **Add footers.** Reference issues with `Refs: #123` or `Closes #123`. Document breaking changes with `BREAKING CHANGE: <what breaks and how to migrate>`.
7. **Show the message before committing** unless the user explicitly asked you to commit without review.

For multi-line messages, use a heredoc so formatting survives:

```sh
git commit -F - <<'EOF'
fix(auth): reject expired refresh tokens

Refresh tokens were accepted for 24h after expiry because the check
compared timestamps in local time instead of UTC.

Closes #482
EOF
```

## Examples

Good:

```
feat(export): add CSV export for invoices
fix(cart): prevent negative quantities on double submit
refactor(db): extract connection pool into its own module
feat(api)!: rename /users endpoint to /accounts

BREAKING CHANGE: clients must update the base path. The old path returns 410 until v3.
```

Bad, and why:

| Message | Problem | Better |
| ------- | ------- | ------ |
| `fixed stuff` | No type, no meaning | `fix(cart): correct total when coupon is applied` |
| `Update files` | Says nothing about the change | Describe the behavior that changed |
| `feat: Added new login page.` | Past tense, capital letter, period | `feat(auth): add login page` |
| `WIP` | Not a meaningful unit of work | Squash or finish before committing |
| `fix: bug` in one commit with a refactor and a feature | Mixed concerns | Split into three commits |

## Checking an existing message

When asked to check a message, verify:

- The type is one of the allowed types.
- The header is under 72 characters and follows the format.
- A blank line separates the header from the body.
- Breaking changes are marked with `!` or a footer.
- The body explains why, not only what.
