---
name: Git Release
description: "Prepare a versioned release of a git repository. Determines the semver bump from commits since the last tag, generates a changelog and release notes, updates version files, creates an annotated tag, and publishes a GitHub release with gh. Use when the user asks to cut a release, bump a version, write a changelog, or publish a release."
license: MIT
---

# Git Release

Ship a release in a predictable, reviewable way. Every step that changes shared state (commit, tag, push, publish) waits for explicit user approval.

## 1. Preflight checks

Run these and stop if any check fails:

```sh
git status --porcelain          # must be empty
git branch --show-current       # must be the default branch (usually main)
git fetch --tags origin
git describe --tags --abbrev=0 2>/dev/null   # last release, empty if none
```

- Dirty working tree: ask the user to commit or stash first.
- Not on the default branch: ask whether to release from it anyway.
- Local branch behind origin: pull before continuing.
- No tags yet: treat the whole history as the first release. Suggest `0.1.0` unless the project is already stable, in which case suggest `1.0.0`.

## 2. Collect changes

```sh
git log <last-tag>..HEAD --pretty=format:'%h %s'
git log <last-tag>..HEAD --pretty=format:'%B' | grep -i 'BREAKING CHANGE'
```

If there are no commits since the last tag, stop. There is nothing to release.

Read commit subjects and bodies. Use the Conventional Commits prefixes (`feat`, `fix`, `perf`, `!`) when present. For commits that do not follow the convention, read the diff summary (`git show --stat <sha>`) and classify them yourself.

## 3. Choose the version

Apply the rules in `references/semver-rules.md`:

- Any breaking change: major bump (minor bump while the version is `0.x`).
- Any new feature: minor bump.
- Only fixes, performance, and internal changes: patch bump.

Present the proposed version and the reason, for example: "3 features and 2 fixes since v1.4.2, so this is **v1.5.0**. Proceed?" Wait for confirmation. The user can override the version.

## 4. Write the changelog

Group entries under these headings, omitting empty groups:

1. Breaking changes
2. Features
3. Fixes
4. Performance
5. Other changes

Each entry is one line, written for users rather than developers, and links the short SHA or PR number when available. Prepend the new section to `CHANGELOG.md`. Create the file if it does not exist, using the [Keep a Changelog](https://keepachangelog.com) layout.

Keep the release notes text in a temporary file. You will pass it to `gh` in step 6.

## 5. Bump version files

Detect version declarations instead of guessing:

| File | Field |
| ---- | ----- |
| `package.json` | `"version"` |
| `pyproject.toml` | `version` under `[project]` or `[tool.poetry]` |
| `Cargo.toml` | `version` under `[package]` |
| `setup.cfg`, `VERSION`, `version.txt` | version string |

Show the diff of the version changes before committing.

## 6. Commit, tag, and publish

After the user approves the diff:

```sh
git add CHANGELOG.md <version files>
git commit -m "chore(release): vX.Y.Z"
git tag -a vX.Y.Z -m "vX.Y.Z"
```

Ask for approval before pushing. Then:

```sh
git push origin <default-branch>
git push origin vX.Y.Z
gh release create vX.Y.Z --title "vX.Y.Z" --notes-file <notes-file>
```

Add `--draft` if the user wants to review the release on GitHub first. For pre-releases, use a suffix such as `vX.Y.Z-rc.1` and add `--prerelease`.

## Safety rules

- Never force-push, rewrite published tags, or delete a tag without an explicit request.
- Never publish a release the user has not approved in this conversation.
- If a step fails halfway, report the exact state (committed? tagged? pushed?) and the command that would undo it. Do not guess.
- Do not include secrets or local paths in the changelog.
