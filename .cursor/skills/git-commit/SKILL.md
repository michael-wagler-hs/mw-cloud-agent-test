---
name: git-commit
description: Create commits with Hootsuite-style messages — Jira issue key first, then a short description. Analyzes staged changes, generates the message, and runs git commit. Use when the user asks to commit, write a commit message, or create a commit.
---

# Commit Messages

## When to Apply

- User asks to commit staged changes
- User wants a commit message or to create a commit
- User asks to review or summarize what will be committed (then offer to commit)

## Format (one line)

Hootsuite commits **must start with the Jira issue key**, followed by a short description:

```
ISSUE-KEY Description
```

- **`ISSUE-KEY`** (required): Jira issue key at the **start** of the line — uppercase project key, hyphen, digits (e.g. `DE-4915`, `SRCH-1016`).
- **`Description`** (required): short summary of the change. Capitalized, imperative mood, no trailing period. Aim for ~50 characters or less; up to ~70 is acceptable if shortening would lose meaning.

That's the whole standard. Teams may layer additional conventions on top (e.g. a trailing `[deployment-target]` tag, a leading `[scope]`, or Conventional Commits-style `type(scope):`) via their own repo-level skill or rule. Do **not** apply those team-specific conventions here unless the repo's own skill/rule asks for them.

### Resolving the Jira key (required before `git commit`)

1. Prefer an issue key already in context: user message, PR title/body, branch name (e.g. `feature/DE-4915-oauth-proxy`), or linked ticket in the conversation.
2. If the key is **not** known from context, **ask the user for the Jira issue key** and wait — do **not** invent a placeholder ticket or omit the key.

## Examples

```
DE-4915 Route refresh tokens to legacy hydra
SRCH-1016 Correct timezone in dates
SCDP-2626 Add knowledge graph prototype
TSG-1405 Essential plugin scaffold
PROJ-99 Add commit skill jira prefix
```

## Workflow

1. **Determine the Jira issue key** (see above). If unknown, ask the user and stop until you have it.
2. Run `git diff --staged` and `git status` to see staged changes.
3. Write a single line: `ISSUE-KEY Description` (capitalized, imperative mood, no trailing period).
4. Run `git commit -m "<message>"` to create the commit.

## Rules

- **Jira key first** on the line; then a short capitalized description.
- One line only; no body or footer unless the user explicitly requests it.
- No period at end of the description.
- Actually run `git commit`; do not only suggest the message.
- Only commit when instructed; do not keep committing subsequent work unless explicitly told.
- Optional: ask if the user would like to push (only if not on main).
