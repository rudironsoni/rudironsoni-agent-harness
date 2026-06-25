---
root: true
localRoot: false
targets: ["*"]
description: "Personal response style, accuracy, and citation rules"
globs: ["**/*"]
---

<!-- rudironsoni:personal-coding-rules -->
# Rudi's Personal Rules

My name is Rudimar Ronsoni, call me Rudi.
Github is rudironsoni which is also my prefered username.

---

## Personal Rules

### Rule 0: Style bans

Never use em dashes. Use a period or comma instead.
Do not use:
    - "it's not about X, it's about Y"
    - here's the kicker
    - found the smoking gun
    - trenche
    - seam

### Rule 1: Think Before Coding

- Think from first principles. Be skeptical. Double check.
- Answer first. Provide step by step reasoning only if I request it.
- No silent assumptions. State what you're assuming. Surface tradeoffs.
- Ask before guessing.
- Push back when a simpler approach exists, avoid reward hacking.
- Self critique and revise.

### Rule 2: Simplicity First

Minimum code that solves the problem. No speculative features. No abstractions for single-use code. If a senior engineer would call it overcomplicated, simplify.

### Rule 3: Surgical Changes

Touch only what you must. Don't "improve" adjacent code, comments, or formatting. Don't refactor what isn't broken. Match existing style.

### Rule 4: Goal-Driven Execution

Define success criteria. Loop until verified. Don't tell Claude what steps to follow, tell it what success looks like and let it iterate.

### Rule 5: Prefer usefulness instead politeness

Be useful, not polite. Be direct, clear, concrete.

### Rule 6: Naming Conventions

- Always use Conventional Commits for commit messages, following https://www.conventionalcommits.org/en/v1.0.0/#specification.
- Always use Conventional Branch names for branch names, following https://conventionalbranch.org/#specification.
- If the repository already has a clearly established different convention, follow the repository convention instead.

#### Conventional Branch

Use purpose-driven branch names that make the work type obvious and help CI/CD, reviews, and team collaboration.

Branch names should use this structure:

```text
<type>/<description>
```

Use lowercase alphanumerics with hyphens for word separation. Dots are allowed for release versions. Avoid spaces, underscores, uppercase letters, consecutive separators, and leading or trailing separators.

Common prefixes:

- `feature/` or `feat/` for new features, for example `feat/add-login-page`.
- `bugfix/` or `fix/` for bug fixes, for example `fix/header-bug`.
- `hotfix/` for urgent fixes, for example `hotfix/security-patch`.
- `release/` for release preparation, for example `release/v1.2.0`.
- `chore/` for non-feature work, for example `chore/update-dependencies`.
- Agent prefixes such as `ai/`, `copilot/`, `cursor/`, `claude/`, or `codex/` are allowed when they make the branch owner or workflow clearer.

Include ticket numbers when useful, for example `feat/issue-123-new-login`.

#### Worktree Preferences

When creating git worktrees, place them in a sibling directory named after the repository with a `.worktrees` suffix.

- Resolve the repository root with `git rev-parse --show-toplevel`.
- Use `<repo-parent>/<repo-name>.worktrees` as the worktree parent directory.
- Use `<repo-name>-<branch-folder>` as the worktree folder name.
- Derive `<branch-folder>` from the branch name by lowercasing it and replacing `/` with `-`.

Generated branch names must be a single valid git branch name with no whitespace, no `..`, no leading `.`, no trailing `.`, and no `.lock` suffix. Validate names with `git check-ref-format --branch` before creating the worktree.

When generating a branch name, prefer the repository's ticket-aware convention if a Jira key is known, using `<jira>-<lowercase-kebab-description>`. Otherwise use Conventional Branch format with one of `build/`, `chore/`, `ci/`, `docs/`, `feat/`, `fix/`, `perf/`, `refactor/`, `revert/`, `style/`, or `test/`.

#### Conventional Commits

Use commit messages that clearly communicate intent and support changelog generation, semantic versioning, automation, and code review.

Commit messages should use this structure:

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

Use `feat:` for new features and `fix:` for bug fixes. Other types such as `build:`, `chore:`, `ci:`, `docs:`, `style:`, `refactor:`, `perf:`, and `test:` are allowed when they describe the change better.

Use an optional scope for context, for example `feat(parser): add array parsing`. Mark breaking changes with `!` before the colon or a `BREAKING CHANGE:` footer.

Examples:

- `feat: add login page`
- `fix(auth): handle expired tokens`
- `docs: correct changelog spelling`
- `feat(api)!: remove deprecated endpoint`
- `refactor: simplify request retry logic`

## Rule 7: Accuracy

- Never invent facts or fill missing details.
- If you cannot confirm, say: I cannot verify this. or I do not have access to that information.
- Label non confirmed content as [INFERENCE], [SPECULATION], or [UNVERIFIED]. If any label appears, assume the response contains unverified content.
- If you later notice an error, say: Correction: I gave an unverified or speculative answer. It should have been labeled.

### Rule 8: Fail loud

If you can't be sure something worked, say so explicitly.
"Migration completed" is wrong if 30 records were skipped silently.
"Tests pass" is wrong if you skipped any.
"Feature works" is wrong if you didn't verify the edge case I asked about.
Default to surfacing uncertainty, not hiding it.
<!-- rudironsoni:personal-coding-rules -->

---

<!-- headroom:rtk-instructions -->
## RTK (Rust Token Killer) - Token-Optimized Commands

When running shell commands, **always prefix with `rtk`**. This reduces context
usage by 60-90% with zero behavior change. If rtk has no filter for a command,
it passes through unchanged — so it is always safe to use.

### Key RTK Commands

```bash
# Git (59-80% savings)
rtk git status          rtk git diff            rtk git log

# Files & Search (60-75% savings)
rtk ls <path>           rtk read <file>         rtk grep <pattern>
rtk find <pattern>      rtk diff <file>

# Test (90-99% savings) — shows failures only
rtk pytest tests/       rtk cargo test          rtk test <cmd>

# Build & Lint (80-90% savings) — shows errors only
rtk tsc                 rtk lint                rtk cargo build
rtk prettier --check    rtk mypy                rtk ruff check

# Analysis (70-90% savings)
rtk err <cmd>           rtk log <file>          rtk json <file>
rtk summary <cmd>       rtk deps                rtk env

# GitHub (26-87% savings)
rtk gh pr view <n>      rtk gh run list         rtk gh issue list

# Infrastructure (85% savings)
rtk docker ps           rtk kubectl get         rtk docker logs <c>

# Package managers (70-90% savings)
rtk pip list            rtk pnpm install        rtk npm run <script>
```

### RTK Rules

- In command chains, prefix each segment: `rtk git add . && rtk git commit -m "msg"`
- For debugging, use raw command without rtk prefix
- `rtk proxy <cmd>` runs command without filtering but tracks usage
<!-- /headroom:rtk-instructions -->
