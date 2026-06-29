# Agent Harness

## How to Work in This Repository

This repository is a collection of separate RuleSync workspaces. Each workspace is a distinct package with its own purpose, `rulesync.jsonc`, `.rulesync/` source tree, generated outputs, and validation boundary.

Do not treat the repository root as one RuleSync package. Do not copy rules, skills, commands, hooks, or generated files between workspaces unless the user explicitly asks for a cross-package migration.

## Workspace Boundaries

| Path | Purpose | Edit Boundary |
|---|---|---|
| `coding-agents/` | Shared coding-agent rules and skills for engineering workflows. | Keep general coding-agent behavior here. |
| `obsidian-plugin-development/` | Agent configuration for Obsidian plugin development. | Keep Obsidian-plugin-specific conventions here. |
| `personal-assistant/` | Personal assistant and second-brain template. | Keep assistant memory, setup, task, and multi-agent template behavior here. |

If a request names one workspace, work only in that workspace unless inspection proves a root-level file must change. If a request is ambiguous, inspect the workspace docs and `rulesync.jsonc` files before choosing a package.

## Source Of Truth

For every workspace, `.rulesync/` and `rulesync.jsonc` are the durable source of truth.

Edit source files such as:

- `.rulesync/rules/`
- `.rulesync/commands/`
- `.rulesync/skills/`
- `.rulesync/mcp.json`
- `.rulesync/permissions.json`
- `.rulesync/hooks.json`
- `rulesync.jsonc`
- `rulesync.lock`, when declarative sources are installed

Generated files are outputs. Do not hand-edit generated target files as the primary fix. Instead, update the owning `.rulesync/` source and regenerate from that workspace.

## RuleSync Workflow

Run RuleSync from the workspace directory, not from the repository root:

```bash
cd personal-assistant
rulesync generate
rulesync generate --check
```

For workspaces with declarative skill sources, use:

```bash
rulesync install
```

Only run `rulesync install --update` when the user explicitly wants dependency/source updates.

### Install A Package Into Another Repository

This repository contains multiple RuleSync packages, each aimed at a particular
use case. Choose the package intentionally before installing it into another
repository:

- `coding-agents/`: shared coding-agent behavior. This package is currently configured as global.
- `obsidian-plugin-development/`: Obsidian plugin development repositories.
- `personal-assistant/`: personal assistant and second-brain repositories.

To install a repo-local package into another repository, run RuleSync from the
selected package in this harness and point `--output-roots` at the target repo:

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

## Generated Output Discipline

Keep source and generated output changes together when they belong to the same logical change. Do not mix unrelated generated churn into a focused commit.

Before committing or summarizing, check:

```bash
git status --short
```

Call out generated output separately from source edits in summaries. Do not claim RuleSync validation unless `rulesync generate --check` was actually run.

## Git Naming Conventions

Use Conventional Commits for commit messages and Conventional Branch names for branches unless an edited downstream repository has a clearly established different convention.

- Commit messages follow `type(scope): description`, with `feat:` for features, `fix:` for bug fixes, and types such as `docs:`, `chore:`, `refactor:`, `test:`, `ci:`, and `build:` when appropriate.
- Branch names follow `<type>/<description>`, using lowercase alphanumerics and hyphens, for example `feat/add-login-page`, `fix/header-bug`, `chore/update-dependencies`, or `release/v1.2.0`.
- Mark breaking commits with `!` before the colon or a `BREAKING CHANGE:` footer.

## Worktree Preferences

When creating git worktrees, place them in a sibling directory named after the repository with a `.worktrees` suffix.

- Resolve the repository root with `git rev-parse --show-toplevel`.
- Use `<repo-parent>/<repo-name>.worktrees` as the worktree parent directory.
- Use `<repo-name>-<branch-folder>` as the worktree folder name.
- Derive `<branch-folder>` from the branch name by lowercasing it and replacing `/` with `-`.

Generated branch names must be a single valid git branch name with no whitespace, no `..`, no leading `.`, no trailing `.`, and no `.lock` suffix. Validate names with `git check-ref-format --branch` before creating the worktree.

When generating a branch name, prefer the repository's ticket-aware convention if a Jira key is known, using `<jira>-<lowercase-kebab-description>`. Otherwise use Conventional Branch format with one of `build/`, `chore/`, `ci/`, `docs/`, `feat/`, `fix/`, `perf/`, `refactor/`, `revert/`, `style/`, or `test/`.

## Root Files

Root-level files describe repository-wide policy:

- `README.md`: public overview and maintainer workflow.
- `.gitignore`: local noise, secrets, and cache ignore rules.
- `AGENTS.md`: agent instructions for this repository.

Root files should not contain package-specific behavior unless it explains how the separate workspaces are organized.

## Personal Assistant Convention

`personal-assistant/` intentionally uses `AGENTS.md` as its neutral navigation convention. `CLAUDE.md` may still appear there as a generated Claude Code target file. Do not reintroduce Claude-first source naming in `.rulesync/`.

## Safety

This repository is intended to stay public-safe.

- Do not commit tokens, credentials, account-specific secrets, or local machine paths.
- Use environment variables or local credential stores for MCP credentials.
- Keep local runtime files in ignored paths.
- Preserve user changes you did not make; never reset or revert unrelated work without explicit permission.