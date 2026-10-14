---
name: pre-push-quality-gates
description: >-
  Mandatory before any git push. Run repo format/lint/check so CI does not fail on
  avoidable gates. Use when the user says push, ship, push to remote, babysit CI,
  create/update a PR, or whenever the agent will run git push — any stack with lint,
  check, or verify targets. Triggered by git-ship rule; does not require create-pr.
---

# Pre-push quality gates

## When this applies

**Default push path:** any time you will run `git push` — whether the user said "push", asked to fix CI and push, or you are shipping a small one-line fix.

The user does **not** need to say "run lint" or "run pre-push-quality-gates". The **`git-ship`** rule requires this skill first.

Also applies when opening/updating a PR or babysitting CI — same gates before each push.

**Passing tests is not enough.** Formatters (e.g. `gofmt`, scalafmt) do not replace linters (e.g. golangci-lint `goconst`, ESLint). Run both when the repo defines them.

**Do not push** until applicable gates below succeed, or you have told the user why local lint could not run and they accept pushing anyway.

## Required workflow (before push)

1. **Discover** what CI likely runs (in order):
   - `Makefile`: `check`, `lint`, `verify-*`, `fmt` / `format`
   - `package.json` scripts: `lint`, `check`, `typecheck`, `validate`
   - Python: `pyproject.toml` / `tox.ini` / `Makefile` lint targets
   - `go.mod` → follow **go-pr-gofmt** (includes `make lint` when present)
   - `build.sbt` / Scala → follow **scala-pr-scalafmt**

2. **Run** the narrowest target that matches CI when obvious:
   - Prefer `make check` only if that is what Jenkins runs end-to-end; otherwise prefer `make lint` (faster) plus format skills.
   - Scope to changed packages when the repo documents a scoped command; otherwise run the repo default.

3. **Fix** failures, stage any auto-format diffs, re-run until clean.

4. **Push** only after step 3 (or explicit user acknowledgment when local lint is blocked — see below).

5. **After push:** if the branch has an open PR and scope changed, refresh the body per **`create-pr`** (“Keep the PR body current”).

## Language skills (do not skip)

| Stack | Skill |
|-------|--------|
| Go | **go-pr-gofmt** — `make fmt` then **`make lint`** when the Makefile has it |
| Scala / sbt | **scala-pr-scalafmt** |

For other stacks, use discovered `npm run lint`, `make lint`, etc. from step 1.

## When local lint fails (toolchain / env)

If `make lint` (or equivalent) errors due to **local toolchain mismatch** (e.g. golangci-lint built for a different Go version):

1. Try the repo’s documented CI path if obvious (`make check`, Docker lint, `./bin/golangci-lint`, Jenkinsfile command).
2. If still blocked, **tell the user** what failed locally and that Jenkins may catch issues — do **not** silently skip lint and push.
3. Prefer fixing or running lint in a matching environment over hoping CI passes.

## Integration

See **`git-ship`** for full order. This skill is **lint/format only** — rebase is **`git-pr-branch-hygiene`**; PR body is **`create-pr`** after push.

Report in chat which commands ran and whether they passed (one line is enough), **before** `git push`.
