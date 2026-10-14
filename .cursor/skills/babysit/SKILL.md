---
name: babysit
description: >-
  Keep a PR merge-ready by triaging comments, resolving clear conflicts, fixing CI,
  and looping on PR-side Bugbot review comments until resolved. Use when babysitting
  a PR, merge-ready, green CI, Bugbot comments, or Cursor Bugbot check.
---
# Babysit PR

Your job is to get this PR to a merge-ready state.

Check PR status, comments, and latest CI and resolve any issues until the PR is ready to merge.

1. **Merge conflicts:** Intelligently resolve any merge conflicts, preserving the intent and correctness of changes on your branch and the base branch. If intents conflict, abort the merge and ask for clarification.
2. **Comments:** Review active unresolved comments (including Bugbot) and resolve change requests / bug reports where valid. When fetching GitHub comments, filter out resolved threads first. Read only each comment body and the minimum location/URL needed to act on it; do not read the entire JSON output or other unnecessary payload data. Carefully validate issues reported by Bugbot and only take action on those that are valid; explain when you disagree or are unsure.
3. **CI:** Fix CI issues caused by changes within this PR's scope. Never change CI checks/workflows just to make failures pass, or make unrelated code changes; if that would be required, report back instead. For merge-blocking failures that seem unrelated to this PR, check whether the branch is behind the base branch and merge latest changes, since another PR may have fixed them. Push scoped fixes and re-watch CI until mergeable + green + comments triaged.

## Bugbot addon (PR-side loop)

After step 2–3 for a given cycle — or whenever unresolved **Bugbot** review threads block merge — run the **Bugbot addon** in [`bugbot-addon.md`](bugbot-addon.md):

- Fetch unresolved Bugbot PR review threads
- Validate each finding; fix valid ones; reply on false positives
- Commit, push, re-trigger with `bugbot run`, poll, repeat until no open valid Bugbot threads or `bugbotMaxRounds` (default 8)

Then return to the main loop (CI + human comments) until merge-ready.

For **local** Bugbot before push, use **review-bugbot** or (in kairos repos) **review-feedback** — the addon covers **PR-side** Bugbot after push.

Remote PR inspection: **remote-pr-review**. Push gates: **pre-push-quality-gates**, **git-ship**.
