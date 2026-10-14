---
name: create-pr
description: >-
  Create a draft pull request (Hootsuite title + repo template body) and keep the
  PR body accurate after follow-up commits, review fixes, version bumps, or linked
  PR changes. Use when opening a PR, ensuring PR descriptions are up to date, or
  after push when an open PR’s scope changed.
---

# Open a PR (draft by default)

**Default:** create **draft** PRs. Use `gh pr create --draft`.
Only omit `--draft` when the user explicitly asks for a ready-for-review / non-draft PR.

To mark a draft ready later: `gh pr ready` (after user asks).

**Before push:** **`git-pr-branch-hygiene`** (rebase) + **`pre-push-quality-gates`** + **`git-commit`** for commits.

## Pre-flight

- On a branch other than `main`, `master`, or `prod`. If not, create and switch to a feature branch.
- Review the diff against the repo default branch.
- Commit all relevant staged/unstaged work first (`git-commit` skill).
- `gh` installed and authenticated.

## Title

Start with the Jira issue key (e.g. `TSG-1405`). Resolve from (in order): branch name, latest commit message, or ask the user. Do not invent a key. Separator after the key may be space or colon.

Keep the title ≤ 80 characters.

## Body — honour an existing PR template

Check the repo for a pull-request template (first match wins):

1. `.github/PULL_REQUEST_TEMPLATE.md`
2. `PULL_REQUEST_TEMPLATE.md` (repo root)
3. `docs/PULL_REQUEST_TEMPLATE.md`
4. `.github/PULL_REQUEST_TEMPLATE/*.md` — if multiple, ask which to use.

Case-insensitive names count.

When a template exists:

- Use it as the body. Do not use `gh pr create --fill` — it overrides the template with commit messages.
- Fill sections from the diff and context. Leave checkboxes unchecked unless you have direct evidence.
- Replace placeholders like `<Insert Jira ID>` with the resolved key.
- Preserve headings, ordering, and HTML comments.

## Body — fallback (no template)

```
<TLDR — no more than 2 sentences>

## Description
- 1–3 bullet points explaining what's changing
```

## Create the PR

```bash
git push -u origin HEAD   # if no upstream or unpushed commits
gh pr create --draft --base <default-branch> --title "<title>" --body "$(cat <<'EOF'
<body>
EOF
)"
```

If a PR already exists for this branch, use it (`gh pr view HEAD`) and refresh the body (next section). Do not recreate.

## Keep the PR body current (required)

The PR **body** is the reviewer-facing source of truth — not commit messages, Bugbot footers, or chat.

**Update when** the branch’s meaningful scope changes: new behavior/tests/deps, review follow-ups, linked PR or publish-order changes, or stale text (wrong versions, removed APIs, abandoned approach). Applies after push even if the user only asked to push, fix CI, or “ensure descriptions are up to date” (refresh **all** related open PRs they mean).

**Skip** only when the user explicitly says not to edit the PR description.

**How**

1. Read: `gh pr view --json number,title,body`.
2. Scope: `git log origin/<default>..HEAD`, `git diff origin/<default>...HEAD`, repo PR template.
3. Rewrite Summary/Description, dependency notes, and test-plan bullets for the **current** diff. Cross-link related PRs when merge order matters.
4. Apply: `gh pr edit <number> --body "$(cat <<'EOF' … EOF)"`.

## Finishing up

- Include the PR link as markdown, e.g. `[PR #123: Title](https://github.com/owner/repo/pull/123)`.
- Note in the response that the PR was opened as a **draft** (when newly created).
- Open the PR in the browser when appropriate (`open` / `gh pr view --web`).
