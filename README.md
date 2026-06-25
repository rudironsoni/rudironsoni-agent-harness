# rudironsoni-agent-harness

RuleSync-backed agent configuration workspaces.

This repository keeps reusable agent rules, skills, hooks, permissions, MCP
configuration, and template projects in source-controlled RuleSync layouts.
Each top-level workspace owns its own `rulesync.jsonc` and `.rulesync/` source
tree; generated agent files are outputs.

## Workspaces

| Path | Purpose | Current RuleSync targets |
|---|---|---|
| `coding-agents/` | Shared coding-agent rules and skills for day-to-day engineering agents. | `agentsmd`, `agentsskills`, `cursor`, `claudecode`, `opencode`, `copilotcli`, `codexcli` |
| `obsidian-plugin-development/` | Agent configuration for Obsidian plugin development work. | `copilot`, `cursor`, `claudecode`, `codexcli` |
| `personal-assistant/` | RuleSync-first personal assistant and second-brain template. | `claudecode`, `cursor`, `copilotcli`, `codexcli`, `antigravity-cli`, `factorydroid`, `pi`, `opencode` |

## Source Of Truth

Edit RuleSync source files, then regenerate target outputs from the workspace
directory that owns the change.

Source files usually live under:

- `.rulesync/rules/`
- `.rulesync/commands/`
- `.rulesync/skills/`
- `.rulesync/mcp.json`
- `.rulesync/permissions.json`
- `.rulesync/hooks.json`
- `rulesync.jsonc`
- `rulesync.lock` when a workspace uses declarative sources

Do not hand-edit generated target files unless the goal is to inspect a
generator result. Put durable behavior changes back into `.rulesync/`.

## Maintainer Workflow

Install RuleSync if it is not already available:

```bash
npm install -g rulesync
```

For a workspace with declarative skill sources, install or refresh sources from
that workspace directory:

```bash
cd coding-agents
rulesync install
```

Regenerate agent files from a workspace:

```bash
cd personal-assistant
rulesync generate
```

Check that generated files are up to date:

```bash
rulesync generate --check
```

Commit source changes and generated outputs together when they belong to the
same logical change. Keep unrelated generated churn out of focused commits.

Use Conventional Commits for commit messages and Conventional Branch names for
branches unless the target repository has a clearly established different
convention.

- Commit messages follow `type(scope): description`, for example
  `docs: update agent workflow` or `feat(rules): add naming convention`.
- Branch names follow `<type>/<description>`, for example
  `feat/add-login-page`, `fix/header-bug`, `chore/update-dependencies`, or
  `release/v1.2.0`.
- Mark breaking commits with `!` before the colon or a `BREAKING CHANGE:`
  footer.

For git worktrees, keep worktree directories outside the repository in a sibling
`<repo-name>.worktrees/` directory. Name each worktree
`<repo-name>-<branch-folder>`, where `<branch-folder>` is the branch name lowercased
with `/` replaced by `-`. Generated branch names should use a Jira-prefixed
lowercase kebab-case name when a Jira key is known, otherwise use Conventional
Branch format. Always validate branch names with `git check-ref-format --branch`
before creating a worktree.

### Install A Package Into Another Repository

This repository contains multiple RuleSync packages. Each package is aimed at a
particular use case, so choose the package intentionally before installing it
into another repository:

- `coding-agents/`: shared coding-agent behavior. This package is currently
  configured as global.
- `obsidian-plugin-development/`: Obsidian plugin development repositories.
- `personal-assistant/`: personal assistant and second-brain repositories.

To install a repo-local package into another repository, run RuleSync from the
selected package in this harness and write generated files into the target repo:

```bash
HARNESS=/path/to/src/rudironsoni/rudironsoni-agent-harness
PACKAGE=personal-assistant
TARGET=/path/to/target-repo

cd "$HARNESS/$PACKAGE"

rulesync generate --check \
  --input-root .rulesync \
  --output-roots "$TARGET"

rulesync generate \
  --input-root .rulesync \
  --output-roots "$TARGET"
```

The target repo does not need its own `rulesync.jsonc` for this flow. The config
comes from the selected harness package. Packages with `"global": true`, such as
`coding-agents/` today, should not be used for repo-local installs as-is.

## Public-Safe Boundary

This repo is intended to stay public-safe.

- Do not commit real tokens, credentials, account-specific secrets, or local
  machine paths.
- MCP credentials should be referenced through environment variables or local
  credential stores.
- Local-only runtime files belong in ignored files, not in `.rulesync/`.
- Lockfiles are acceptable when they record source refs or integrity metadata,
  not secrets.

## Validation

Before publishing a workspace change:

```bash
rulesync generate --check
```

For `personal-assistant/`, also verify that `memory/` navigation uses
`AGENTS.md` and that `CLAUDE.md` appears only as the generated Claude Code
entrypoint.
