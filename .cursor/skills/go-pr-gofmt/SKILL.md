---
name: go-pr-gofmt
description: >-
  Enforces gofmt (and repo format targets) before committing, pushing, or opening
  Go pull requests. Use when editing Go code, working in a Go module repo, babysitting
  a Go PR, running create-pr, or before any git push (pre-push-quality-gates / git-ship rule)
  on a Go project.
---

# Go PR — gofmt before submit

## When this applies

Any **Go module** work that ends in **commit**, **push**, or **PR** — including babysitting CI, `/create-pr`, and hand-off to review.

**Do not** push or open a PR until Go files are gofmt-clean **and** repo lint passes when present.

`gofmt` alone does **not** satisfy this skill — golangci-lint rules (e.g. `goconst`, `wrapcheck`) require **`make lint`** or equivalent.

## Required workflow (before commit / push / PR)

1. **Format** changed `.go` files (prefer repo targets when present):

   ```bash
   make fmt
   ```

   If there is no `fmt` Make target, format files touched on this branch vs the PR base (replace `<base>` with `master`, `main`, or the repo default):

   ```bash
   git diff --name-only origin/<base>...HEAD -- '*.go' | while read -r f; do
     [ -n "$f" ] && gofmt -w "$f"
   done
   ```

   If the diff set is empty (e.g. before first commit), format packages you edited:

   ```bash
   gofmt -w path/to/changed/...
   ```

2. **Verify** the same checks CI uses when obvious from the repo `Makefile`:

   ```bash
   make lint
   ```

   Or, if only formatting is needed and the repo documents it:

   ```bash
   gofmt -l $(git diff --name-only origin/<base>...HEAD -- '*.go')
   ```

   `gofmt -l` must produce **no output** before push.

3. **Stage** any files reformatted by gofmt together with functional changes (format-only diffs are expected).

4. **Commit → push → PR** only after steps 1–2 succeed.

## Discovery order

1. Repo `Makefile` (`fmt`, `lint`, `check` targets) — prefer these when present.
2. `go.mod` at repo root — confirms a Go module; use `gofmt` on changed paths.
3. Ask the user how CI runs format/lint checks only if neither is obvious.

## Failure handling

- If `gofmt -l` lists files: run step 1 again, re-check, do not push.
- If CI fails on `gofmt` / `gci` / golangci formatting after a green local run: reproduce with `make lint` on the PR branch tip, fix, push again.

## Integration with other workflows

- **pre-push-quality-gates**: parent workflow for any push; this skill is the Go-specific part.
- **create-pr / babysit PR**: run this skill **after** code changes and tests, **before** `git push` and `gh pr create --draft`.
- **git-pr-branch-hygiene**: rebasing does not replace gofmt or lint; run both on the final branch state before push.

## Local lint blocked

If `make lint` fails due to a local Go/golangci version mismatch, follow **pre-push-quality-gates** (try Docker/`check`, warn the user — do not silently skip and push).
