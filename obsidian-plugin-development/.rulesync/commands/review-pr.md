---
description: "Review a pull request"
targets: ["*"]
---

target_pr = $ARGUMENTS

If target_pr is not provided, use the PR of the current branch.

Execute the following in parallel:

1. Check code quality, style consistency, and fit with existing architecture.
2. Review test coverage and whether desktop and mobile Obsidian runtime paths were verified.
3. Verify documentation, manifest, release artifact, and `versions.json` updates when relevant.
4. Check for potential bugs, security issues, Obsidian API misuse, accessibility regressions, and mobile incompatibilities.

Prioritize blocking correctness and security findings first. Do not treat build or unit test results as proof that Obsidian UI/runtime behavior works.
