---
name: task-management
description: "Use when creating or editing a task -- in TASKS.md, an external issue tracker, or anywhere. Not for read-only task lookups or simple status toggles."
---

# Task Management

Ensures relevant wiki memories are loaded whenever {{NAME}} creates or edits a task, so the assistant has full context (why it matters, who it connects to, what prior knowledge applies).

## External issue trackers are the source of truth for their issues

If an external issue tracker (e.g. Jira, Linear, GitHub Issues) is configured for this workspace, internalise this policy before any task action that touches it:

- When the task lives in the external tracker, the assistant **never** edits a local mirror of those issues directly. All mutations go through the tracker's own API or CLI (for Jira, that means `acli jira workitem create/edit` or the Atlassian MCP tools such as `createJiraIssue`, `editJiraIssue`, `transitionJiraIssue`, `addCommentToJiraIssue`).
- After any change in the external tracker, refresh the local mirror if one exists (e.g. a `make sync` target).
- Any local mirror of tracker issues is read-only outside of its sync tooling. Reads are fine; writes are not.
- Reading from local `.md` files is the fast path for context lookup; this is exactly why a mirror exists.
- `TASKS.md` remains local-first; the no-write policy is **only** for issues backed by an external tracker.

If no external tracker is configured, all tasks live in `TASKS.md` (sections: Active, Waiting On, Someday, Done) and there is no source-of-truth conflict to worry about.

## Step 1 — Identify the task

This step also applies to implicit task actions — e.g. converting a `TASKS.md` item into an external ticket, or a conversation that leads to "let's track this" even if the word "task" isn't used.

Extract the **task description** from the user's message — the subject, keywords, and any project/person names mentioned. This is the query text used for ranking.

## Step 2 — Rank memories

Read `memory/CLAUDE.md`. For each page listed, score its relevance to the task query by comparing the page name, summary, and any entity names against the task description. Use keyword overlap, entity matching, and semantic proximity.

Select the **top-k most relevant pages** (k = min(5, total pages)). Always include every page that shares an explicit entity name (person, project) with the task.

## Step 3 — Ask the user

Present the top-k pages using the `AskQuestion` tool with `allow_multiple: true`. Format:

- **id**: `select-memories`
- **prompt**: "I found these memories related to your task. Which ones should I load for context?"
- **options**: one per page, using the page name as label and the filename as id.

If there is only one relevant page, still ask — don't auto-load silently.

If no pages seem relevant (score is very low for all), skip this step and tell the user no related memories were found. Proceed directly to task management.

## Step 4 — Load and proceed

Read the selected memory pages. Use their content as context while creating or editing the task (in `TASKS.md`, an external tracker, or wherever the task is being tracked).

When writing or updating the task, leverage loaded context to:
- Write a richer task description that captures why the task matters.
- Add relevant cross-references if appropriate (e.g., project names, people).
- Flag connections the user might not have mentioned ("this relates to X from your notes on Y").
