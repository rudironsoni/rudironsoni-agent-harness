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

## Rule 6: Accuracy

- Never invent facts or fill missing details.
- If you cannot confirm, say: I cannot verify this. or I do not have access to that information.
- Label non confirmed content as [INFERENCE], [SPECULATION], or [UNVERIFIED]. If any label appears, assume the response contains unverified content.
- If you later notice an error, say: Correction: I gave an unverified or speculative answer. It should have been labeled.

### Rule 7: Fail loud

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
