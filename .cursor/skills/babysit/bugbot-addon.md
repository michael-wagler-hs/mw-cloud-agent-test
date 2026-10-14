# Babysit — Bugbot addon

PR-side Cursor Bugbot loop. Run during **babysit** after merge conflicts, human comments, and CI are handled — or in parallel when Bugbot is the only blocker.

Complements local Bugbot (`review-bugbot` / `review-feedback`). This addon watches **GitHub PR review comments** from Bugbot and fixes until threads are resolved or you hit the round cap.

## Defaults

- `bugbotMaxRounds` = `8` (fix → push → re-fetch cycles per babysit session)
- `bugbotPollSeconds` = `90` (wait between push and re-fetch when Bugbot check still pending)
- `bugbotAuthors` = treat as Bugbot when review-comment author login matches `cursor-bugbot`, `bugbot`, or `cursor[bot]` (case-insensitive)

## Step 1 — Locate PR

Same as core babysit / **remote-pr-review**: `gh pr view --json number,url,headRefOid` for current branch, or parse user-supplied PR URL.

Record `head_sha` before each fix cycle (`git rev-parse HEAD` after commits).

## Step 2 — Fetch unresolved Bugbot threads

Use `gh`. Read **minimum fields only** (author, body, path, line, thread id/URL, resolved flag) — not full raw JSON dumps.

Sources:

1. **PR review comments** (line-anchored) — primary Bugbot surface
2. **PR reviews** whose body looks like a Bugbot summary — secondary; still triage like review comments

Filter:

- **Keep** unresolved threads where the root (or sole) comment author matches `bugbotAuthors`
- **Drop** resolved threads
- **Drop** threads you already addressed this session (track by thread id + `head_sha` when fixed)

If zero unresolved Bugbot threads → addon done for this cycle.

## Step 3 — Triage each finding

For each unresolved Bugbot comment, **before editing**:

1. Open the cited file at the line (or nearest hunk if line drifted)
2. Decide: **valid** | **false positive** | **uncertain**

Rules (same spirit as core babysit step 2):

- **Valid** → fix at root cause; scope minimal to the finding
- **False positive** → do **not** change code; reply on thread with reasoning; leave unresolved (or resolve only if team convention is to resolve dismissed bot threads)
- **Uncertain** → reply asking for human confirmation; do not guess-merge

Never bulk-apply Bugbot suggestions without reading the code.

## Step 4 — Fix, verify, commit, push

For all **valid** findings in the round:

1. Apply fixes (tests when behavior changes)
2. Run scoped local verification (repo test/lint targets for touched paths; **pre-push-quality-gates** before push)
3. Commit with Jira key from branch (**git-commit-jira-prefix**)
4. Push to PR branch
5. Reply on each fixed thread: short note + commit SHA; **resolve thread** via `gh`/GraphQL

## Step 5 — Re-trigger PR Bugbot

After push, best-effort re-review:

```bash
gh pr comment <number> --body "bugbot run"
```

Non-blocking if `gh` fails. Bugbot may also run from the `Cursor Bugbot` check on its own — do not spam `bugbot run` every round; once per push is enough.

## Step 6 — Poll and loop

1. Re-fetch unresolved Bugbot threads (Step 2)
2. Optionally watch `Cursor Bugbot` check: `gh pr checks --watch` until complete or timeout
3. If new unresolved Bugbot comments on **current** `head_sha` → increment round; go to Step 3
4. If `bugbotMaxRounds` reached with open valid findings → stop; report residuals and URLs
5. If only false positives / uncertain remain → stop; report what you replied

```
fetch Bugbot threads
    → valid findings? → fix → push → bugbot run → poll → (round++)
    → none / only dismissed → done
```

## Step 7 — Report

One-line status plus table when findings existed:

| Thread | Location | Disposition |
|--------|----------|-------------|
| URL | `path:line` | fixed / false-positive / deferred / uncertain |

Include: rounds used, final `head_sha`, CI status, unresolved Bugbot thread URLs.

## What not to do

- Do not loop forever — respect `bugbotMaxRounds`
- Do not resolve threads without a fix or a written false-positive reply
- Do not change CI/workflows to silence Bugbot
- Do not push without **pre-push-quality-gates** (or user-approved skip)
- Do not treat Bugbot issue comments as automatically valid — validate each finding
