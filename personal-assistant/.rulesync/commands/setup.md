---
targets:
  - '*'
description: >
  Interactive, agent-guided setup for this Personal Assistant. Walks the user
  through identity (name/email/GitHub), Google Workspace authentication in an
  isolated config dir, and makefile generation. Runs deterministic scripts from
  .rulesync/scripts/setup/ for everything that can be automated, and pauses to
  confirm anything that requires the user (OAuth flows, etc). Use this
  immediately after cloning the template.
claudecode:
  name: setup
---
# /setup - stand up this Personal Assistant

You are orchestrating a one-time setup flow that turns a fresh template clone into a personalized, running assistant. The user just cloned the template repo and started an agent session in it. Your job is to interview them, run the scripts under `.rulesync/scripts/setup/`, and at the end leave them with a working `make run` command plus clear next steps.

## Principles

- **Favor determinism.** Every side effect goes through one of the scripts in `.rulesync/scripts/setup/`. Do not hand-edit files to replicate what a script can do. Reserve direct file edits for things the scripts do not cover.
- **Verify after each side effect.** If a script runs, read back the file(s) it touched and confirm the change took. If verification fails, abort.
- **Track progress.** Use the active agent's native todo/progress mechanism when available; otherwise keep a concise checklist in the conversation.
- **Ask one question at a time** unless a group of questions is genuinely atomic, such as name and email.
- **Idempotency.** If setup is re-run, detect completed state and offer to skip or re-run each step individually. Do not blindly redo everything.

## Error Handling

On any non-zero exit from a setup script, stop the flow. Report to the user:

1. Which step failed, such as `generate-makefile.sh`.
2. The exact stderr the script emitted.
3. The most likely fix based on the script's error message.
4. Instruction to re-run setup after fixing.

Do not attempt automatic retry. Do not skip past the failing step. The user decides how to proceed.

## Flow

### 1. Greet and scope

Briefly explain what this command does. Warn that some steps, such as Google Workspace OAuth, happen outside the chat and you will pause for the user to complete them in another terminal or app.

Start a progress checklist with these items:

- Identity: collect name, email, optional GitHub handle
- Apply placeholders
- Init local settings
- Google Workspace auth (optional, isolated config dir)
- GitHub account pin for `gh auth token` (optional)
- Agent launch command
- Generate makefile
- Fresh git history (optional)
- Print next steps

### 2. Identity

Ask for:

- Name
- Email
- GitHub username (optional; drives PR search in the `daily-briefing` skill; accept empty)

### 3. Apply placeholders

Run `.rulesync/scripts/setup/apply-placeholders.sh "<name>" "<email>" "<gh_handle_or_empty>"`.

Verify by reading the first 12 lines of `AGENTS.md` and confirming `{{NAME}}` and `{{EMAIL}}` are both gone. Do not show the content back to the user unless they ask.

### 4. Init local settings

Run `.rulesync/scripts/setup/init-settings-local.sh`. No verification needed; the script's output is self-explanatory.

### 5. Google Workspace (optional)

Ask whether to enable Google Workspace integration for Gmail, Calendar, and Drive access via the `gws` CLI. If no, skip this step entirely.

If yes:

1. Check `gws` is installed: `command -v gws`. If not, print the install command (`npm install -g @googleworkspace/cli` plus Node and `gcloud` prerequisites) and abort this step with a message that the user should install `gws` then re-run setup.
2. Ask the user for a config dir name. Suggest `gws-<nick>` where `<nick>` is the repo basename, for example `gws-personal-assistant`. They can override.
3. Run `.rulesync/scripts/setup/ensure-gws-config-dir.sh <dir_name>`. Relay the `gws auth setup` command the script prints to the user. Tell them to run it in a separate terminal, complete the OAuth flow with the Google account this assistant should act as, and come back.
4. Wait for the user to confirm OAuth is done.
5. Run `.rulesync/scripts/setup/verify-gws.sh "$HOME/.config/<dir_name>"`. Show the authenticated email and ask the user to confirm it is the right account.
6. Remember `<dir_name>` for the makefile generation step. If the user skipped this whole section, treat it as empty.

### 6. GitHub account pin (optional)

Only applicable if the user provided a GitHub handle in step 2 and is running multiple `gh` accounts. Detect this by running `gh auth status` and counting the number of logged-in accounts.

- If 0 or 1 `gh` accounts: skip. The default `gh` auth is fine.
- If 2+ accounts: show the user the list and ask which one this assistant should use. `make run` will bake `gh auth token --user <chosen>` into the environment so the assistant's `gh` calls act as that user regardless of which account is active globally.

Remember the chosen user for the makefile generation step. If skipped, treat as empty.

### 7. Agent launch command

Ask which command should run the assistant from `make run`. Recommend the command for the agent the user is currently using:

| Agent | Suggested command |
|---|---|
| Claude Code | `claude` |
| Cursor | `cursor-agent` |
| Copilot CLI | `copilot` |
| Codex | `codex` |
| Antigravity CLI | `antigravity` |
| Factory Droid | `droid` |
| Pi | `pi` |
| OpenCode | `opencode` |

Accept the user's override verbatim. Remember this value for makefile generation.

### 8. Generate the makefile

Run `.rulesync/scripts/setup/generate-makefile.sh <nick> <gws_config_dir_or_empty> <gh_user_or_empty> <agent_command>`. If the script refuses because a `makefile` already exists, show the existing makefile to the user and ask whether to overwrite. If yes, re-run with `--force` as the fifth argument.

The `nick` is used only as a label in the makefile. Use the repo basename.

Read the generated makefile and show it to the user so they understand what `make run` will do.

### 9. Fresh git history (optional)

Ask: "Wipe template git history and re-init? (recommended unless you are customizing this repo as a fork)." If yes, run `.rulesync/scripts/setup/fresh-git-init.sh`. Warn that this is destructive before running.

### 10. Next steps

Print a final summary block covering:

- How to launch: `make run`
- How to test the daily briefing: `ask "what's on my plate today?"`

End by marking the progress checklist complete. The user can exit the session and run `make run` to start their configured assistant.

## Notes for the agent

- The scripts are located relative to the repo root. Always invoke them as `.rulesync/scripts/setup/<name>.sh`.
- The scripts use positional args, not env vars. Quote all args that could contain spaces.
- Do not read the scripts' source unless debugging; trust their exit codes and stdout/stderr.
- If the user asks "what exactly is this going to do?", describe the flow above and the scripts involved, but do not overwhelm them with implementation details.
