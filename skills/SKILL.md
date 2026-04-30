---
name: create-a-pr-safely
description: Use when the user wants Claude to implement a JIRA ticket end-to-end (branch, code, commit, PR) following team conventions.
---
# Create a PR Safely (JIRA-driven)

This skill implements a JIRA ticket end-to-end with disciplined hygiene: correct base branch, correct branch naming, acceptance-criteria-driven implementation, verified locally, clean commits, and a PR title/body that follows a predefined format.

<HARD-GATES>
1) Do NOT push, open a PR, or modify remote branches until you have shown the user the exact plan (branch name, commit plan, verification plan) and they approve.
2) Do NOT force-push unless the user explicitly asks for it.
3) Do NOT commit secrets. If any secret-like content appears (tokens/keys/passwords), stop and ask the user how to proceed.
4) Do NOT implement beyond the ticket scope. If the ticket is ambiguous or missing acceptance criteria, stop and ask for clarification or require the user to paste the missing details.
</HARD-GATES>

## Required conventions (must be known)
Before starting, you must know or ask for:
- Base branch name (usually `main`)
- Branch naming pattern
- PR title pattern
- Commit message pattern (optional but recommended)

If the user does not provide patterns, use these defaults and ask them to confirm:

Branch naming pattern (default):
- `feature/{JIRA_ID}-{slug}`

PR title pattern (default):
- `{JIRA_ID}: {ticket_summary}`

Commit message pattern (default):
- `{JIRA_ID}: {short_summary}`

## Inputs to confirm
Ask only what you need:
- JIRA ticket ID (e.g. `ABC-123`)
- JIRA ticket URL (preferred) OR pasted ticket content (required if you cannot access JIRA)
- Any local setup constraints (env vars, services, feature flags)
- Verification standard (tests/lint/build command). If unknown, locate it.

## JIRA ticket loading rules
You must have the following ticket fields before coding:
- Title / summary
- Description
- Acceptance criteria (or Definition of Done)
- Any linked designs/specs
- Any dependencies or blocked-by notes

If you cannot access JIRA content directly, ask the user to paste these fields into the chat. Do not ask for or store JIRA API tokens unless the user explicitly wants to configure secure access.

## Execution flow (follow in order)

### 1) Read and restate the ticket
Summarize the ticket in your own words:
- What the user wants
- What is explicitly in scope
- What is explicitly out of scope (if stated)
- List each acceptance criterion as a checklist item

If any acceptance criterion is ambiguous, ask a clarifying question before proceeding.

### 2) Plan + user approval gate
Present a short plan that includes:
- The exact branch name you will create (must match pattern)
- What files/areas you expect to touch (best guess)
- The verification commands you will run
Wait for user approval.

### 3) Confirm repo state and sync base branch
- `git status` (must be clean or user-approved to proceed)
- Identify base branch and sync from origin
If base branch diverged locally, stop and ask whether to rebase or merge.

### 4) Create branch with the fixed pattern
Create the branch name using:
- `{JIRA_ID}` exactly (case preserved)
- `{slug}` derived from the ticket summary (lowercase, hyphens, no punctuation)

Example:
- Ticket: `ABC-123` “Add export button”
- Branch: `feature/ABC-123-add-export-button`

### 5) Implement strictly to acceptance criteria
Implement the smallest correct change set that satisfies all criteria.
As you work, maintain an explicit checklist and mark each acceptance criterion “done” only when:
- code is implemented
- it is testable
- it is verified locally (when possible)

If you discover missing requirements, stop and ask the user instead of guessing.

### 6) Verify locally (required)
Run the repo’s standard checks (tests/lint/build).
If checks fail, fix them before committing unless the user explicitly requests a WIP PR.

### 7) Diff review gate (required)
Before committing, review the diff for:
- unrelated changes
- accidental files
- debug logs
- dependency/lockfile changes not required by the ticket
- secrets

If the diff includes work not justified by the ticket, stop and ask whether to split it into another ticket/PR.

### 8) Commit cleanly with ticket ID
Prefer 1–3 commits max unless the user requests otherwise.
Commit message must include `{JIRA_ID}` and a meaningful summary.

### 9) Push safely
Push the branch to origin with upstream tracking.
No force-push.

### 10) Create PR with the predefined title and body format
PR title must match your team pattern, default:
- `{JIRA_ID}: {ticket_summary}`

PR body must include:
- JIRA link
- Summary of changes (what/why)
- Acceptance criteria checklist (copied from ticket, now checked off)
- Verification commands run and results
- Notes / follow-ups (if any)

After creating the PR, confirm CI started. If CI fails, switch to a CI-triage flow (identify failing workflow/job/step, extract exact error, reproduce locally if possible, fix minimally, push, re-check).
