---
name: git-pr-branch-hygiene
description: >-
  Sync local default with upstream before branching; rebase feature branches onto
  latest default before push or PR. Use when branching, rebasing, pushing a feature
  branch, opening a PR, or running create-pr / babysit.
---

# Git PR branch hygiene

Git history only — PR title/body rules live in **`create-pr`**.

Resolve the repo **default branch** (`master`, `main`, or configured default) before the commands below.

## Before creating a feature branch

Local default must match **upstream** before you branch:

```bash
git fetch origin
git switch <default-branch>
git pull --ff-only origin <default-branch>
```

Then create the branch from that tip (e.g. `git switch -c <ticket-branch>`).

If you need a fresh branch while already on a feature branch, update default first, then branch from it — not from a stale local default.

## Before push or PR

Rebase the feature branch onto the **latest** default:

```bash
git fetch origin
git switch <feature-branch>
git rebase origin/<default-branch>
```

- Resolve conflicts, continue the rebase, and run relevant tests.
- After a successful rebase, update the remote with `git push --force-with-lease` (history was rewritten).
- If the user did not ask for a rebase and push is rejected for other reasons, do not force-push; stop and report.

Do not open or refresh a PR from a branch that is behind a stale picture of default — fetch and rebase (or merge, if the repo uses merge) first.

When **creating** a new PR, follow **`create-pr`** (draft by default). After push, if an open PR’s scope changed, refresh its body there too — not in this skill.

## With other workflows

- **pre-push-quality-gates**: after rebase, on the **final** branch state, before push.
- **Jira commit prefix rule**: stage → commit (with ticket key) → push — never push uncommitted work.
- **git-ship** rule: orchestrates hygiene → gates → push → **`create-pr`** body refresh.
