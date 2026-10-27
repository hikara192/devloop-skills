# Semantic Versioning Rules for Releases

Reference: [Semantic Versioning 2.0.0](https://semver.org) and [Conventional Commits 1.0.0](https://www.conventionalcommits.org).

## Mapping commits to bumps

| Signal in commit | Bump (version >= 1.0.0) | Bump (version 0.x) |
| ---------------- | ----------------------- | ------------------ |
| `feat!:`, `fix!:`, or a `BREAKING CHANGE:` footer | major | minor |
| `feat:` | minor | minor |
| `fix:`, `perf:`, `refactor:`, etc. | patch | patch |
| Only `docs:`, `test:`, `ci:`, `chore:`, `build:` | no release needed (ask the user) | no release needed (ask the user) |

The highest-impact commit wins. One breaking change in a release of 50 fixes is a major bump.

## Pre-1.0 projects

While the version is `0.x`, the public API is not yet stable. The usual convention is:

- Breaking changes bump the minor version (`0.4.0` to `0.5.0`).
- Features and fixes bump the patch version.

Only move to `1.0.0` when the user explicitly decides the API is stable.

## Edge cases

- **Revert commits**: a `revert:` commit cancels a previous change. If the reverted change was never released, drop both from the changelog. If it was released, record it under Fixes.
- **Deprecations**: a deprecation alone is a minor change. The removal later is breaking.
- **Security fixes**: classify as patch, and mention the advisory or CVE in the changelog when there is one.
- **Dependency updates**: list as "Other changes" unless they change behavior users see, in which case treat them as fixes or features.
- **Commits without a convention**: classify by reading the diff. Do not assume a commit is a patch just because its message is vague.
- **Pre-releases**: `1.5.0-rc.1` is ordered before `1.5.0`. Increment the `rc` counter for each candidate. Do not bump the base version for an rc.
- **Monorepos**: each package has its own tag and version. Scope the commit log to the package path with `git log <tag>..HEAD -- <path>`.
