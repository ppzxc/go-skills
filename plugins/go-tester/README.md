# go-tester

A Go testing patterns reference skill for Claude Code.

## Contents

1. **Table-Driven Tests** — t.Run, struct slices, parallel subtests
2. **Test Helpers** — t.Helper, t.Cleanup, t.Parallel
3. **Mocks** — manual mock via interface, function-field pattern
4. **Data Race Detection** — go test -race
5. **Benchmarks** — b.ResetTimer, b.ReportAllocs, sub-benchmarks, benchstat
6. **Fuzzing** — testing.F, seed corpus (Go 1.18+)
7. **Integration Testing** — testcontainers-go, build tags
8. **httptest** — handler and middleware testing
9. **Golden File Testing** — testdata/ convention, -update flag
10. **Test Lifecycle** — TestMain, testing.TB helpers
11. **Coverage & Profiling** — -cover, -cpuprofile, -memprofile

## Installation

```bash
/plugin marketplace add ppzxc/go-plugins
/plugin install go-tester
```

## Usage

```
/go-tester
```

Also activates automatically when writing, debugging, or reviewing Go tests.
