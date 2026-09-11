# Go Lint Pre-Commit Checklist

- [ ] Checked the `Makefile` for a `lint` (and `lint-fix`/`fmt`) target before running
      `golangci-lint` directly.
- [ ] `make lint` (or `golangci-lint run ./...` if no make target exists) reports zero issues
      on touched packages.
- [ ] Formatting applied via `make fmt` if it exists, otherwise `golangci-lint fmt ./...`
      (or `gofmt -l .` shows no unformatted files).
- [ ] No new bare `//nolint` comments — every suppression names a linter and a reason.
- [ ] `.golangci.yml` was not edited just to silence a single finding.
- [ ] If `.golangci.yml` genuinely needed to change, the user confirmed that change.
- [ ] `go vet ./...` passes (golangci-lint runs this too, but it's a fast sanity check).
- [ ] Tests still pass: `go test ./...`.
