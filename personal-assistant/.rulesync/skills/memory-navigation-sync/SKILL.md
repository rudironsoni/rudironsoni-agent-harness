---
name: memory-navigation-sync
description: >-
  Use on every markdown write under memory/ -- create, rename, move, or delete,
  at any depth. Patches ancestor AGENTS.md navigation. Not for files outside
  memory/.
targets:
  - '*'
---
# Memory AGENTS.md sync

Whenever a markdown file under `memory/` changes, the `AGENTS.md` files above it must stay coherent. This skill keeps that hierarchy honest: every time you create, rewrite, rename, move, or delete a memory page, walk up the directory chain and patch each ancestor `AGENTS.md`.

The [`agent-navigation-template`](../agent-navigation-template/SKILL.md) skill will also fire because we are editing `AGENTS.md` files — that is expected. This skill says **what** to change at the content level; `agent-navigation-template` ensures the structural spine stays intact.

## Step 1 — Identify the chain

Skip read-only operations, non-markdown files (e.g. `.obsidian/` configs), and anything outside `memory/`. If multiple memory pages change in one turn, run this skill once at the end of the turn and patch every affected ancestor in a single pass.

For each changed page `P`:

1. Walk up from `P`'s directory toward `memory/`.
2. Collect every `AGENTS.md` between `P`'s directory and `memory/` (inclusive of both). The workspace root `AGENTS.md` is **not** in scope — it lists top-level workspace folders only and does not need updating when a single memory page changes.
3. Order the chain from closest-to-`P` (the immediate parent) to furthest (`memory/AGENTS.md`).

The template's `memory/` is **flat**: every page is a sibling of `memory/AGENTS.md`, so for almost every page the chain is length 1 — `memory/AGENTS.md` only. A leaf like `memory/foo.md` has just `memory/AGENTS.md` above it. The multi-level walk-up below still applies if you grow `memory/` into nested sub-folders (each with its own `AGENTS.md`); for the default flat wiki, Step 2 is usually all you need.

## Step 2 — Update the immediate parent

The immediate parent `AGENTS.md` is where the changed page is listed by name. In the default flat wiki this is always `memory/AGENTS.md`. Pick the right section by inspecting which catalog sections exist in that file:

- `## Projects` — pages of type `project` (ongoing efforts, initiatives, areas of work).
- `## Concepts` — pages of type `concept` or `synthesis` (themes, frameworks, mental models, analyses).
- `## Preferences` — pages of type `preference` (durable preferences, conventions, standing instructions).
- `## Deep-Dive Navigation` — links to child `AGENTS.md` files. Only relevant when the changed entity is itself a navigation node (a sub-folder with its own `AGENTS.md`), which is rare in the flat wiki.

Match the section to the page's frontmatter `type` field. Apply the right operation:

| Change type | What to do in the immediate parent |
|-------------|-------------------------------------|
| New page    | Insert a new row into the relevant table with the page's relative link, a tight one-line description grounded in the page contents, and keywords that match the page. |
| Updated page | Compare the existing row's description and keywords to the page's current purpose. If the core purpose or scope has materially shifted (new topic, expanded scope, different framing), rewrite the row. If the edit is minor (typo, small clarification, one new bullet), leave the row alone. |
| Renamed page | Update the link target. If the human-readable label was a slug of the old filename, refresh it. Check any deep-link anchors elsewhere in the file. |
| Moved page | Remove the row from the old parent (if the page moved across directories) and insert it in the new parent. Both parents are in scope for the sweep. In a flat wiki a "move" is usually just a section change (e.g. a page reclassified from `Concepts` to `Projects`) within the same `memory/AGENTS.md`. |
| Deleted page | Remove the row entirely. If a custom prose paragraph elsewhere in the file referenced the deleted page, update or remove that reference too. |

Read enough of the changed page to write an accurate description and pick keywords. Mirror the tone of the existing rows in the same table — they are usually one sentence ending in a period, with keywords as comma-separated lowercase tokens.

## Step 3 — Walk further up

This step only applies if `memory/` has nested sub-folders with their own `AGENTS.md` files. In the default flat wiki the chain ends at `memory/AGENTS.md` and you can skip ahead to Step 4.

For deeper hierarchies, the work in distant ancestors is lighter. You are **not** adding a row per leaf page in distant ancestors — that is precisely why intermediate `AGENTS.md` files exist. You are checking whether the ancestor's *description of the next step down* has drifted.

For each remaining ancestor in the chain, in order:

1. Find the row in the ancestor's `## Deep-Dive Navigation` table that points to the next step down the chain — e.g. `[sub-area/AGENTS.md](sub-area/AGENTS.md)` inside `memory/AGENTS.md`.
2. Ask: does the changed page meaningfully shift the character of that subfolder (a new domain emerging, a new theme family, a removed pillar)?
3. If yes, refresh that row's description and keywords to reflect the broader picture.
4. If no, leave it alone.

Most leaf-page additions stop at the immediate parent. Only structural shifts in the folder's overall shape ripple further up.

## Protections

- **Never reorder** existing rows or sections. Insertions go at the bottom of the relevant table unless an obvious alphabetical or thematic ordering is already in place — match the existing order if so.
- **Never rewrite** custom prose, operating rules, what-belongs-here sections, suggested-shape sections, or any non-table content unless the user asked you to.
- **Preserve table column headers exactly** — downstream tooling and the `agent-navigation-template` skill rely on `Path | Description | Keywords`.
- **Use relative links** rooted at the `AGENTS.md` file being edited (`foo.md` from `memory/AGENTS.md`, not absolute paths, not `[[wikilinks]]`).
- **Standard Markdown only** — `[label](relative/path.md#anchor)`. Section anchors are `#kebab-case-of-the-heading`.

## Step 4 — Verify links

Editing, moving, deleting, or renaming memory pages and rewriting nav rows is exactly when links break, so after patching every affected ancestor `AGENTS.md`, run the link checker to confirm nothing broke.

1. **Run the check.** Prefer `make check-links`. If no makefile exists yet (pre-`/setup`), fall back to the direct command from the repo root: `lychee --offline --include-fragments --config lychee.toml './**/*.md'`.
2. **Best-effort only.** `lychee` is an optional dependency. If the command is not found, note that the link check was skipped (`brew install lychee` to enable it) and **do not** block the memory write on a missing tool.
3. **Fix what this change caused.** If the check reports broken links, fix the ones introduced by this sweep — a nav row pointing at a moved/renamed/deleted page, or a stale relative link — and re-run. For broken links you cannot confidently attribute or fix, report them rather than guessing.

## Step 5 — Report

After the sweep, briefly tell the user which ancestors you touched and what changed — e.g. *"Added a row for `memory/foo.md` to `memory/AGENTS.md` under Projects."* If you walked deeper ancestors, mention the ones you considered but left alone, so the reasoning is visible without forcing a diff review. Fold in the link-check outcome — clean, fixed (and what), skipped (lychee not installed), or unresolved links you flagged for the user.
