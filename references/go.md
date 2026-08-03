# Go

## Framework
Standard library `testing` package by default. Use `testify` (`assert`/`require`) only if
it's already an import in the project's existing tests or `go.mod`.

## File placement & naming
- `<file>_test.go` in the same package/directory as `<file>.go` — this is fixed by Go
  convention, not a style choice.
- Test function names: `TestXxx(t *testing.T)`.

## Style
- Table-driven tests are the Go-idiomatic default for anything with multiple input/output
  cases — use a `[]struct{ name string; input ...; want ... }` slice with `t.Run(tt.name,
  ...)` subtests, rather than separate `TestXxxCase1`, `TestXxxCase2` functions.
- Use `t.Errorf`/`t.Fatalf` with clear "got X, want Y" messages if not using testify; use
  `require.Equal`/`assert.Equal` if testify is present (use `require` for setup failures that
  should stop the test, `assert` for the actual checks so all failures in one test are
  reported).

## Mocking
- Prefer designing tests around small interfaces the code already depends on (Go's implicit
  interfaces make this cheap) rather than a heavy mocking framework.
- If the project already uses `gomock`/`mockery`/`testify/mock`, match that instead of hand-
  rolling fakes.
- For HTTP, use `httptest.NewServer` / `httptest.NewRecorder` rather than mocking the client.

## Running
```
go test ./path/to/package/...
go test -run TestXxx ./path/to/package/... -v
go test -cover ./path/to/package/...
```

## Common pitfalls to avoid
- Don't forget `t.Parallel()` consideration if the existing suite uses it — but don't add it
  blindly if tests share mutable state.
- Watch for goroutine leaks / unclosed channels in concurrent code under test.
