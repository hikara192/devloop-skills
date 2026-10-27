# OpenCode Skills

A collection of production-ready skills for [OpenCode](https://opencode.ai). Each skill gives the agent a focused, repeatable workflow for one engineering task.

| Skill | What it does |
| ----- | ------------ |
| [`code-review`](skills/code-review) | Reviews diffs, branches, and PRs. Prioritizes correctness, security, and data integrity. Reports findings by severity with `file:line` references. |
| [`git-release`](skills/git-release) | Picks the semver bump from commits, writes the changelog, bumps version files, tags, and publishes a GitHub release. Every shared-state step needs approval. |
| [`test-writer`](skills/test-writer) | Writes behavior-focused tests that match the project's existing framework and conventions, then verifies they can actually fail. |
| [`conventional-commits`](skills/conventional-commits) | Writes Conventional Commits 1.0.0 messages and splits mixed changes into atomic commits. |
| [`security-audit`](skills/security-audit) | Finds secrets, injection, broken access control, SSRF, and vulnerable dependencies. Reports only verified issues, with a fix for each. |

## Installation

### Option 1: Copy into your global skills directory

```sh
git clone https://github.com/hikara192/opencode-skills.git
cp -r opencode-skills/skills/* ~/.config/opencode/skills/
```

### Option 2: Reference the folder from your config

Add the path to `opencode.json` or `opencode.jsonc`. This keeps the skills in sync with `git pull`:

```jsonc
{
  "$schema": "https://opencode.ai/config.json",
  "skills": ["/absolute/path/to/opencode-skills/skills"],
}
```

Config changes apply without a restart. Run `/reload` in OpenCode if a skill does not appear right away.

### Using a skill

Let the agent choose a skill from the task description, or load one explicitly by mentioning it:

```
@code-review review the changes on this branch
@conventional-commits commit the staged changes
```

## Structure

```
skills/
├── index.json                  # HTTP catalog index
├── code-review/
│   ├── SKILL.md
│   └── references/checklist.md
├── git-release/
│   ├── SKILL.md
│   └── references/semver-rules.md
├── test-writer/
│   └── SKILL.md
├── conventional-commits/
│   └── SKILL.md
└── security-audit/
    ├── SKILL.md
    └── references/owasp-checklist.md
```

Each skill is one directory with a `SKILL.md` file. The frontmatter `description` tells the agent when to use it. Supporting material lives in `references/` and is read only when the skill needs it, which keeps the main instructions short.

## Contributing

1. Keep each skill focused on one task and describe the trigger conditions clearly in `description`.
2. Put long checklists, tables, or examples in `references/`.
3. Make every step that changes shared state (pushes, publishes, deletes) require explicit confirmation.
4. Bump `version` in `skills/index.json` for any skill you change.

## License

[MIT](LICENSE)
