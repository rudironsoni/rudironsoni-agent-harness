---
root: true
localRoot: false
targets: ["*"]
description: "Personal response style, accuracy, citation rules"
globs: ["**/*"]
---

<!-- rudironsoni:personal-coding-rules -->
# Rudi's Personal Rules

My name is Rudimar Ronsoni. Call me Rudi. GitHub is `rudironsoni`, also my preferred username.

## Conventions

- Never use em dashes. Use periods or commas instead.
- Do not use: "it's not about X, it's about Y", "here's kicker", "found smoking gun", "trenche", or "seam".
- Answer first. Provide step-by-step reasoning only if requested.
- State assumptions, surface tradeoffs, and ask before guessing.
- Be useful, direct, clear, and concrete.
- Never invent facts to fill missing details.
- If you cannot confirm something, say: `I cannot verify this.` or `I do not access that information.`
- Label non-confirmed content with `[INFERENCE]`, `[SPECULATION]`, or `[UNVERIFIED]`.
- If you later notice an error, say: `[CORRECTION] I previously gave an unverified or speculative answer. It should have been labeled, and here is the corrected version.`

## Rules

Agents MUST follow these execution rules:

- Think from first principles. Be skeptical. Double check.
- Push back when a simpler approach exists. Avoid reward hacking.
- Self-critique and revise before presenting results.
- Use the minimum code that solves the problem. Do not add speculative features or single-use abstractions.
- Touch only what is necessary. Do not improve adjacent code, comments, formatting, or structure.
- Do not refactor what is not broken. Match existing style.
- Define success criteria before changing code, then loop until verified.
- Do not say work is complete unless it was actually verified. State skipped checks, uncertainty, or partial verification.
- For agent delegation, describe success criteria instead of prescribing every step.

## Naming

- Use repository conventions when they are clearly established.
- Otherwise use Conventional Commits for commits and Conventional Branches for branches.

### Branches

- Use `<type>/<description>`.
- Use lowercase kebab-case after the slash.
- Allowed types: `build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test`.
- Also allowed when useful: `feature`, `bugfix`, `hotfix`, `release`, `ai`, `copilot`, `cursor`, `claude`, `codex`.
- Include ticket numbers when useful.
- If a Jira key is known, prefer `<jira>-<lowercase-kebab-description>`.

```text
<type>/<description>
```

### Commits

- Use `<type>[optional scope]: <description>`.
- Prefer `feat:` for features and `fix:` for bugs.
- Allowed types: `build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test`.
- Use scopes when they clarify context, for example `fix(auth): handle expired token`.
- Mark breaking changes with `!` before the colon or a `BREAKING CHANGE:` footer.

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

### Worktrees

- Place worktrees in a sibling directory named after the repository with a `.worktrees` suffix.
- Resolve the repository root with `git rev-parse --show-toplevel`.
- Use `<repo-parent>/<repo-name>.worktrees` as the worktree parent directory.
- Use `<repo-name>-<branch-folder>` as the worktree folder name.
- Derive `<branch-folder>` from the branch name by lowercasing it and replacing `/` with `-`.
- Generated branch names must be valid single git branch names with no whitespace, no `..`, no leading `.`, no trailing `.`, and no `.lock` suffix.
- Validate generated branch names with `git check-ref-format --branch` before creating the worktree.

<!-- rudironsoni:personal-coding-rules -->

---

<!-- headroom:rtk-instructions -->
## RTK (Rust Token Killer)

When running shell commands, always prefix each command segment with `rtk`. It reduces context usage and passes through unchanged when no filter exists.

## Key RTK Commands

```bash
rtk git status          rtk git diff            rtk git log
rtk ls <path>           rtk read <file>         rtk grep <pattern>
rtk find <pattern>      rtk diff <file>
rtk pytest tests/       rtk cargo test          rtk test <cmd>
rtk tsc                 rtk lint                rtk cargo build
rtk prettier --check    rtk mypy                rtk ruff check
rtk err <cmd>           rtk log <file>          rtk json <file>
rtk summary <cmd>       rtk deps                rtk env
rtk gh pr view <n>      rtk gh run list         rtk gh issue list
rtk docker ps           rtk kubectl get         rtk docker logs <c>
rtk pip list            rtk pnpm install        rtk npm run <script>
```

## RTK Rules

- In command chains, prefix each segment: `rtk git add . && rtk git commit -m "msg"`.
- For debugging, use the raw command without `rtk`.
- Use `rtk proxy <cmd>` to run a command without filtering while still tracking usage.

<!-- /headroom:rtk-instructions -->
