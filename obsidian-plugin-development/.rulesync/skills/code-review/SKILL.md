---
name: code-review
description: >-
  Review Obsidian plugin code for correctness, security, maintainability,
  performance, tests, release readiness, accessibility, and desktop/mobile
  runtime risk.
targets:
  - "*"
---

# Obsidian Plugin Code Review

Use this skill when reviewing a pull request, auditing code, or checking whether a change is ready to ship in an Obsidian plugin.

## Review Priorities

1. Correctness: lifecycle, settings migration, command availability, editor behavior, file operations, and race conditions.
2. Security: unsafe HTML, eval, dynamic code execution, path handling, external data validation, and hardcoded secrets.
3. Obsidian API fit: use `this.app`, `requestUrl`, `registerEvent`, `registerDomEvent`, `registerInterval`, `vault.configDir`, and `fileManager.trashFile` where appropriate.
4. Mobile compatibility: `isDesktopOnly`, touch target size, mobile-safe regex, responsive layout, and `Platform` guards.
5. Accessibility: keyboard operation, focus state, icon labels, semantic text, and no color-only affordances.
6. Tests and verification: unit tests for pure logic plus real Obsidian runtime checks for UI, editor, or plugin lifecycle behavior.
7. Release readiness: `manifest.json`, `styles.css`, `versions.json`, version consistency, and BRAT or GitHub release payload expectations.

## Findings Format

Lead with blocking findings, each with file and line when available. State the exact failure mode and the smallest fix. Put skipped runtime verification or remaining risk after findings.

Do not claim runtime behavior works from static checks alone.
