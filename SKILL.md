---
name: golang-lint
description: Lints Go code with golangci-lint, fixes reported issues, and keeps the project's .golangci.yml config consistent. Use whenever writing, editing, or reviewing Go (.go) files, before committing Go changes, or when the user asks to lint, format, or clean up Go code.
---

# Golang Lint

Use this skill any time you write or modify Go code in this project, or when asked to lint,
fix warnings, or clean up Go files. The goal is zero `golangci-lint` findings on changed code,
without silencing checks unless there's no reasonable fix.

## Key Principles

- Never introduce new `golangci-lint` findings. Run the linter after edits, not just at the end.
- Fix the underlying issue rather than suppressing it. Only use `//nolint` as a last resort,
  and always include a reason (see Nolint policy below).
- Keep the repo's `.golangci.yml` (or `.golangci.yaml`) as the single source of truth. Don't
  pass ad hoc `--enable`/`--disable` flags that diverge from it unless the user asks for that.
- Prefer `golangci-lint run --fix` for auto-fixable issues, then handle the rest by hand.

## Workflow

1. **Check for a `make lint` target first.** Look for a `Makefile` (or `makefile`,
   `GNUmakefile`) at the repo root and grep it for a `lint:` target — e.g.
   `grep -E '^lint:' Makefile` or `make -n lint` to preview what it runs. Many Go repos wrap
   `golangci-lint` in `make lint` to pin the version, pass a specific config path, or lint
   only certain modules/directories. If a `lint` target exists, use `make lint` (and
   `make lint-fix` / `make fmt` if those targets also exist) instead of invoking
   `golangci-lint` directly, so you match the project's actual CI behavior.
   - If `make -n lint` shows the target just shells out to `golangci-lint run ...` with no
     extra logic, it's fine to also call `golangci-lint` directly for faster iteration on a
     single package — but do a final `make lint` before considering the work done.
   - If the target does more (codegen, multiple modules, Docker, a pinned installer script),
     always go through `make lint` — replicating it by hand risks missing a step.
2. Confirm the config: check for `.golangci.yml` / `.golangci.yaml` / `.golangci.toml` /
   `.golangci.json` at the repo root (golangci-lint searches upward from the linted path).
   If none exists, see `templates/.golangci.yml` for a reasonable v2 starting point rather
   than inventing one from memory.
3. After editing Go files, run the linter scoped to what changed:
   - Via make: `make lint`
   - Whole module directly: `golangci-lint run ./...`
   - Only files newer than a base ref (fast, good for iterative work):
     `golangci-lint run --new-from-rev=HEAD ./...`
4. Run formatters as a separate step — check for a `make fmt` target first; otherwise
   `golangci-lint fmt ./...` (v2) handles `gofmt`/`goimports`-style formatting, while `run`
   handles the actual lint checks.
5. Fix every reported issue:
   - Auto-fixable: `make lint-fix` if that target exists, otherwise `golangci-lint run --fix ./...`
   - Everything else: edit the code to address the finding directly.
6. Re-run `make lint` (or `golangci-lint run ./...` if there's no make target) until it reports
   no issues on the touched files.
7. If a finding is a genuine false positive, use a scoped `//nolint:<linter>` comment with a
   reason (see below) instead of disabling the linter project-wide.

## Nolint policy

- Always scope to the specific linter: `//nolint:errcheck // reason` — never a bare `//nolint`
  that suppresses everything on the line.
- Always include a `// reason` explaining why suppression is correct here, not just "todo" or
  "fix later".
- Never add a linter to the `linters.exclude` (or legacy `disable`) list in `.golangci.yml` to
  silence a single finding — fix the code or use a scoped `//nolint` instead. Only touch the
  project-wide config when the user explicitly asks to change linting policy.

## Config format notes (golangci-lint v2)

- Config files are `.golangci.yml`, `.golangci.yaml`, `.golangci.toml`, or `.golangci.json` at
  the repo root, or nearest ancestor of the linted path.
- v2 config requires a top-level `version: "2"` key and has separate `linters:` and
  `formatters:` sections (formatting linters like `gofmt`/`goimports` moved out of `linters`
  into `formatters` in v2).
- A v1 config (no `version` key, or `version: "1"`) will fail to load on the v2 binary. If the
  repo still has a v1 config, run `golangci-lint migrate` to convert it rather than hand-editing —
  don't do this without confirming with the user first, since it rewrites the config file.
- Check which config file is actually being used with `golangci-lint run -v` (or
  `golangci-lint config verify` / `golangci-lint config path` if available in the installed
  version) if linting behavior seems unexpected.

## Useful commands

- `make -n lint` — preview what the project's `lint` make target actually runs, without running it.
- `make lint` — the project's own lint entry point, if one exists; prefer this over calling
  `golangci-lint` directly.
- `golangci-lint run ./...` — lint the whole module.
- `golangci-lint run --fix ./...` — lint and auto-fix what can be fixed.
- `golangci-lint run --new-from-rev=<rev> ./...` — lint only code changed since `<rev>`.
- `golangci-lint fmt ./...` — apply configured formatters.
- `golangci-lint linters` — list enabled/disabled linters for the current config.
- `golangci-lint config verify` — validate the config file against the schema.
- `golangci-lint migrate` — convert a v1 config file to v2.

See `checklists/pre-commit-checklist.md` for the checklist to run through before considering
Go changes done, and `templates/.golangci.yml` for a starting config when a project has none.
