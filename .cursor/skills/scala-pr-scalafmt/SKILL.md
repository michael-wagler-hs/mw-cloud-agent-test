---
name: scala-pr-scalafmt
description: >-
  Enforces scalafmt before committing, pushing, or opening Scala/sbt pull requests.
  Use when editing Scala code, working in an sbt repo, babysitting a Scala PR, running
  create-pr, or before any git push that will trigger Jenkins scalafmtCheck on a Scala project.
---

# Scala PR — scalafmt before submit

## When this applies

Any **Scala / sbt** repository work that ends in **commit**, **push**, or **PR** — including babysitting CI, `/create-pr`, and hand-off to review.

**Do not** push or open a PR until formatting is applied and the format check passes.

## Required workflow (before commit / push / PR)

1. **Format** changed code (whole project unless the repo documents a narrower target):

   ```bash
   make scalafmt
   ```

   If there is no `scalafmt` Make target, use:

   ```bash
   sbt scalafmtAll scalafmtSbt
   ```

   For a focused change in one sbt subproject, you may format only that module (e.g. `sbt "apiCore/scalafmt" "apiCore/Test/scalafmt"`), then still run the **check** step below at repo scope unless CI only checks that module.

2. **Verify** the same checks CI uses:

   ```bash
   make test
   ```

   Or, if you only need format verification without full tests:

   ```bash
   sbt scalafmtCheckAll scalafmtSbtCheck
   ```

3. **Stage** any files reformatted by scalafmt together with your functional changes (formatting-only diffs are expected).

4. **Commit → push → PR** only after step 2 succeeds.

## Discovery order

1. Repo `Makefile` (`scalafmt`, `test` targets) — prefer these when present.
2. `build.sbt` / `.scalafmt.conf` if Make targets are missing.
3. Ask the user how CI runs format checks only if neither is obvious.

## Failure handling

- If `scalafmtCheck` fails: run the format command again, re-run check, do not push.
- If CI fails on `scalafmt` / `scalafmtCheckAll` after a green local run: reproduce with `make test` on the PR branch tip, fix, push again.

## Integration with other workflows

- **create-pr / babysit PR**: run this skill **after** code changes and tests, **before** `git push` and `gh pr create --draft`.
- **git-pr-branch-hygiene**: rebasing/updating default branch does not replace scalafmt; run format + check on the final branch state before push.
