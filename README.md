# rudironsoni-agent-harness

Personal AI-agent configuration powered by [Rulesync](https://github.com/dyoshikawa/rulesync).

This repository is the source of truth for shared agent rules, skills, permissions, hooks, and MCP server definitions. Rulesync reads the files in `.rulesync/` and renders tool-specific configuration for the configured agent targets.

The repo is public-safe by design. Secrets and account-specific credentials are expected to come from environment variables or local credential stores, not from committed files.

## What This Manages

- Response style, accuracy, and execution rules shared across agents.
- Local and curated skills exposed to compatible agent tools.
- MCP server definitions for supported clients.
- Permission defaults for shell, edit, and read operations.
- Hook configuration for tools that support pre-tool-use hooks.
- Lockfiles for reproducible skill source resolution.

## Repository Layout

```text
.
+-- rulesync.jsonc              # Rulesync targets, global settings, and skill sources
+-- rulesync.lock               # Locked refs and integrity hashes for declarative skill sources
+-- .aiignore                   # AI-tool ignore rules
+-- .rulesync/
    +-- rules/                  # Source rules rendered into target agent configs
    +-- skills/                 # Local skills plus fetched curated skills
    +-- mcp.json                # MCP server definitions
    +-- permissions.json        # Tool permission policy
    +-- hooks.json              # Agent hook configuration
```

Generated target files are outputs. Treat `rulesync.jsonc` and `.rulesync/` as the source files to edit.

## Configured Targets

`rulesync.jsonc` currently renders configuration for:

- `agentsmd`
- `agentsskills`
- `cursor`
- `claudecode`
- `opencode`
- `copilotcli`
- `codexcli`

The project is configured with `global: true`, so Rulesync generates user-scope configuration where the selected target supports it.

## Maintainer Workflow

Install Rulesync if it is not already available:

```bash
npm install -g rulesync
```

Install declared skill sources using the lockfile:

```bash
rulesync install
```

Regenerate agent configuration:

```bash
rulesync generate
```

Check whether generated files are up to date:

```bash
rulesync generate --check --targets "*" --features "*"
```

Update locked skill refs intentionally:

```bash
rulesync install --update
```

Commit source changes and lockfile updates together when they belong to the same change.

## Skills

Declarative skill sources are listed in `rulesync.jsonc` under `sources`. `rulesync install` resolves those sources and records the exact refs in `rulesync.lock`.

Skill precedence:

- `.rulesync/skills/<name>/` contains local skills and should be committed.
- `.rulesync/skills/.curated/<name>/` contains fetched skills from declared sources.
- Local skills win when a local and curated skill share the same name.

## MCP Servers and Credentials

MCP server definitions live in `.rulesync/mcp.json`. Credentials are referenced through environment variables, for example:

- `ATLASSIAN_MCP_SERVER_API_TOKEN`
- `GITHUB_MCP_SERVER_API_TOKEN`

Do not commit real tokens, credentials, or machine-local secrets. Keep local-only values in your shell, password manager, or ignored credential files.

## Validation

Before publishing changes, run:

```bash
rulesync install --frozen
rulesync generate --check --targets "*" --features "*"
```

If Rulesync is not installed, at minimum review the changed source files and confirm that `rulesync.jsonc` remains valid JSONC.
