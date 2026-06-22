# Personal Assistant

A RuleSync-first personal assistant and second-brain template.

The durable source of agent behavior lives in `.rulesync/`. Tool-specific files
for Claude Code, Cursor, Copilot CLI, Codex, Antigravity CLI, Factory Droid, Pi,
and OpenCode are generated from that source.

## What Ships

- Local task tracking in `TASKS.md`
- Durable memory under `memory/`
- Agent navigation files rooted at `AGENTS.md`
- Shared commands, skills, MCP config, hooks, permissions, and rules in `.rulesync/`
- Generated tool-specific config for the supported agents

## Supported Agents

| Agent | RuleSync target | Generated surface |
|---|---|---|
| Claude Code | `claudecode` | `CLAUDE.md`, `.claude/` |
| Cursor | `cursor` | `.cursor/`, `.cursorignore` |
| Copilot CLI | `copilotcli` | `.copilot/mcp-config.json`, `.github/copilot-instructions.md`, `.github/hooks/`, `.github/skills/` |
| Codex | `codexcli` | `AGENTS.md`, `.codex/`, `.agents/skills/` |
| Antigravity CLI | `antigravity-cli` | `.agents/`, `.geminiignore` |
| Factory Droid | `factorydroid` | `AGENTS.md`, `.factory/` |
| Pi | `pi` | `.pi/prompts/`, `.pi/skills/` |
| OpenCode | `opencode` | `opencode.jsonc`, `.opencode/` |

Some targets do not support every RuleSync feature. For example, `pi` does not
support MCP or hooks, and `copilotcli`, `codexcli`, and `antigravity-cli` do not
support generated command files. RuleSync handles those skips during generation.

## Source Of Truth

Edit these files by hand:

- `.rulesync/rules/personal-assistant.md`
- `.rulesync/commands/*.md`
- `.rulesync/skills/**`
- `.rulesync/mcp.json`
- `.rulesync/hooks.json`
- `.rulesync/permissions.json`
- `.rulesync/scripts/setup/*`
- `memory/**/AGENTS.md`
- `TASKS.md`

Do not hand-edit generated agent outputs. After changing `.rulesync/`, run:

```bash
rulesync generate
rulesync generate --check
```

## Setup

Clone the template, open it with your agent of choice, and run the generated
setup command or prompt for that agent. The setup flow interviews you, applies
placeholders, configures optional Google Workspace access, generates a local
`makefile`, and optionally resets template git history.

The setup scripts live in `.rulesync/scripts/setup/` so every supported agent
uses the same deterministic implementation.

The setup flow asks for:

| Input | Used for |
|---|---|
| Name | Replaces `{{NAME}}` in agent instructions and skills |
| Email | Replaces `{{EMAIL}}` in agent instructions |
| GitHub username | Optional PR/search context for daily briefing |
| Google Workspace config dir | Optional isolated `gws` auth profile |
| GitHub account pin | Optional `gh auth token --user <name>` selection |
| Agent launch command | Command that `make run` should execute, such as `claude`, `codex`, or `opencode` |

## Google Workspace Notes

The `daily-briefing`, `wiki-ingest`, and `google-workspace-cli` skills use
[`gws`](https://googleworkspace-cli.mintlify.app) for Google APIs. The setup flow
can create an isolated config directory such as `$HOME/.config/gws-personal-assistant`
and bake it into the generated `makefile`.

If your GCP project owner and OAuth sign-in account differ, make sure the signed-in
Workspace account has API access to the GCP project and is listed as a test user
while the OAuth app is in testing mode.

To switch the signed-in account later:

```bash
GOOGLE_WORKSPACE_CLI_CONFIG_DIR="$HOME/.config/gws-<nick>" gws auth login
rm ~/.config/gws-<nick>/token_cache.json
```

Only remove the token cache if API calls still use the old account.

## Layout

```text
.
├── AGENTS.md                    generated neutral entrypoint for agents that use it
├── CLAUDE.md                    generated Claude Code entrypoint
├── TASKS.md                     local task list
├── memory/
│   └── AGENTS.md                wiki hub and navigation root
├── .rulesync/                   editable source of agent config
│   ├── rules/
│   ├── commands/
│   ├── skills/
│   └── scripts/setup/
├── .claude/                     generated Claude Code files
├── .cursor/                     generated Cursor files
├── .codex/                      generated Codex files
├── .copilot/                    generated Copilot CLI MCP config
├── .github/                     generated Copilot CLI files
├── .agents/                     generated Codex/Antigravity shared files
├── .factory/                    generated Factory Droid files
├── .pi/                         generated Pi files
└── .opencode/                   generated OpenCode files
```

## Skills And Commands

| Name | Purpose |
|---|---|
| `daily-briefing` | Builds agenda and task briefings from local memory and optional external tools |
| `wiki` | Ingests source notes into durable memory |
| `task-management` | Maintains `TASKS.md` |
| `agent-navigation-template` | Enforces Root/Intermediate/Leaf `AGENTS.md` navigation files |
| `memory-navigation-sync` | Keeps ancestor `AGENTS.md` navigation in sync on memory writes |
| `google-workspace-cli` | Uses `gws` for Gmail, Calendar, Drive, Docs, Sheets, and Slides |
| `project-context` | Summarizes project context and constraints |

| Command | Purpose |
|---|---|
| `setup` | One-time bootstrap for identity, `gws`, makefile, and optional git reset |
| `daily-briefing` | Build an agenda/task brief |
| `wiki-ingest` | Convert raw source documents into `memory/` pages |
| `weekly-done-cleanup` | Review and clean completed tasks |
| `compact-session` | Produce a concise handoff/resume note |
| `grill-me` | Challenge a plan against the workspace docs |
