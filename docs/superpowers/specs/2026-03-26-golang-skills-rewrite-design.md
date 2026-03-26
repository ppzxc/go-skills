# golang Skills Rewrite Design

## Context

The `golang` plugin provides three skills (`coder`, `reviewer`, `tester`) for Claude Code AI agents working with Go code. The current skills were written based on 3 books with a prose-heavy explanatory format. This rewrite upgrades both content quality and format to be AI-agent optimized.

## Goals

1. Rewrite all 3 skills with expanded source material (7 books + official docs)
2. Switch to Rule-based DO/DON'T format optimized for AI agent consumption
3. Target 500-600 lines per skill, higher information density than current ~670 lines
4. Maintain current skill scope boundaries (coder/reviewer/tester)

## Source Materials

| Abbrev | Title | Author | Year |
|--------|-------|--------|------|
| [GoBook] | The Go Programming Language | Donovan & Kernighan | 2015 |
| [Mistakes] | 100 Go Mistakes and How to Avoid Them | Harsanyi | 2022 |
| [Concurrency] | Concurrency in Go | Cox-Buday | 2017 |
| [LearningGo] | Learning Go, 2nd Edition | Bodner | 2024 |
| [LearnWithTests] | Learn Go with Tests | Chris James | ongoing |
| [CloudNative] | Cloud Native Go | Titmus | 2021 |
| [LetsGo] | Let's Go / Let's Go Further | Edwards | 2024 |
| [EffectiveGo] | Effective Go | Go Team | official |
| [CodeReview] | Go Code Review Comments | Go Wiki | official |

## Common Format

All skills share this structure:

```markdown
---
name: <skill-name>
description: "<trigger description>"
user_invocable: true
---

# <Title>

> Sources: [Abbrev] notation for references

## Section N: <Topic>

### DO
- **Rule**: One-line imperative rule [Source#N]
  ```go
  // Minimal example (3-5 lines)
  ```

### DON'T
- **Anti-pattern**: Pattern to avoid [Source#N]
  ```go
  // Counter-example (3-5 lines)
  ```

### WHEN
- Conditional rule: "In X situation, do Y" [Source#N]
```

### Format Rules
- Rules use imperative voice ("Use X", "Return Y", "Avoid Z")
- One code example per rule, max 5 lines
- Each rule tagged with source: `[Mistakes#42]`, `[GoBook$7]`, `[Concurrency$4]`
- Section ends with cross-reference to related skills
- Priority ordering: most important rules first, rare edge cases last

## Skill 1: coder

**Trigger**: Writing or refactoring Go production code
**Target size**: ~550 lines, ~70 rules

### Sections

| # | Topic | Sources | Rules |
|---|-------|---------|-------|
| 1 | Types & Data Structures | GoBook$4, Mistakes#18-28, LearningGo$5 | 8-10 |
| 2 | Interfaces | GoBook$7, Mistakes#5-9, LearningGo$7 | 6-8 |
| 3 | Error Handling | Mistakes#48-57, LearningGo$9, GoBook$5 | 8-10 |
| 4 | Functions & Methods | Mistakes#10-17, GoBook$5 | 5-7 |
| 5 | Concurrency | Concurrency$3-5, Mistakes#58-73, GoBook$8-9 | 10-12 |
| 6 | Context | Mistakes#74-77, LearningGo$14 | 5-6 |
| 7 | HTTP & Web | LetsGo, GoBook$7 | 6-8 |
| 8 | Service Design | CloudNative$4-5, LearningGo$15 | 5-7 |
| 9 | Resilience | CloudNative$5-6, Mistakes#78-81 | 5-6 |
| 10 | Performance | Mistakes#82-100, GoBook$11 | 6-8 |

### Section Details

**Section 1: Types & Data Structures**
- Struct field ordering (memory alignment)
- Slice initialization with known capacity (`make([]T, 0, n)`)
- Map initialization with size hint
- Prefer value types for small structs
- Use named types for clarity
- Generics: when to use, when to avoid
- Enum pattern with iota
- Zero value usefulness

**Section 2: Interfaces**
- Accept interfaces, return concrete types
- Keep interfaces small (1-3 methods)
- Define interfaces at consumer side, not producer
- Don't export interfaces for implementation
- Embedding for composition
- Type assertions and type switches

**Section 3: Error Handling**
- Always check returned errors
- Wrap with context using `fmt.Errorf("...: %w", err)`
- Use sentinel errors for expected conditions
- Custom error types for rich context
- `errors.Is` / `errors.As` for matching
- Never use panic for control flow
- Handle errors once (don't log and return)
- Provide actionable error messages

**Section 4: Functions & Methods**
- Functional options pattern for config
- Avoid named return values except for documentation
- Use defer for cleanup
- Pointer vs value receivers: consistency rule
- Limit function parameters (max 4-5)

**Section 5: Concurrency**
- Every goroutine must have a clear exit path
- Use errgroup for concurrent tasks with error handling
- Mutex for shared state, channel for communication
- Never start goroutines in init()
- Use sync.WaitGroup for fire-and-forget
- Channel direction in signatures
- Select with default for non-blocking
- Goroutine leak prevention patterns
- Avoid closure variable capture pitfalls
- sync.Once for lazy initialization

**Section 6: Context**
- Pass context as first parameter
- Never store context in structs
- Use context.WithTimeout for external calls
- context.Value only for request-scoped data
- Respect context cancellation in loops

**Section 7: HTTP & Web**
- Handler pattern: `func(w http.ResponseWriter, r *http.Request)`
- Middleware chaining
- Use http.NewServeMux (Go 1.22+ patterns)
- Structured JSON response helpers
- Proper status code usage
- Request validation at boundary

**Section 8: Service Design**
- Constructor injection for dependencies
- Environment-based configuration
- Structured logging with slog
- Health check endpoints
- Clean separation of transport/business/storage

**Section 9: Resilience**
- Exponential backoff with jitter for retries
- Graceful shutdown with signal handling
- Circuit breaker for external dependencies
- Timeout all external calls
- Connection pool management

**Section 10: Performance**
- strings.Builder for concatenation
- Pre-allocate slices and maps
- Avoid unnecessary interface conversions
- sync.Pool for high-churn allocations
- Profile before optimizing (pprof)
- Escape analysis awareness

## Skill 2: reviewer

**Trigger**: Reviewing Go code in pull requests or auditing Go codebases
**Target size**: ~500 lines, ~50 rules

### Sections

| # | Topic | Sources | Rules |
|---|-------|---------|-------|
| 1 | Error Handling | Mistakes#48-57, CodeReview | 8-10 |
| 2 | Concurrency Safety | Concurrency$3-5, Mistakes#58-73 | 10-12 |
| 3 | Naming & Style | EffectiveGo, CodeReview, Mistakes#1-4 | 6-8 |
| 4 | API Design | GoBook$7, Mistakes#5-9, LearningGo$7 | 6-8 |
| 5 | Performance Pitfalls | Mistakes#82-100, GoBook$11 | 6-8 |
| 6 | Security | LetsGo, OWASP Go | 5-6 |
| 7 | Package & Project Structure | EffectiveGo, Mistakes#1-4 | 5-6 |

### Section Details

**Section 1: Error Handling Review**
- Check: no ignored errors (unchecked returns)
- Check: errors wrapped with context before returning
- Check: consistent error wrapping strategy (%w vs %v)
- Check: sentinel errors are package-level vars
- Check: error messages don't start with capital or end with punctuation
- Check: no panic in library code
- Flag: errors.New in loops (should be package-level)
- Flag: bare `return err` without context

**Section 2: Concurrency Safety**
- Check: every goroutine has exit mechanism
- Check: shared state protected by mutex OR channel
- Check: no goroutine leaks (blocked channel, missing done signal)
- Check: WaitGroup.Add before goroutine launch
- Check: defer mu.Unlock() immediately after Lock()
- Check: channel direction in function signatures
- Check: no race conditions in closure captures
- Flag: unbuffered channel without guaranteed receiver
- Flag: sync primitives copied (must pass by pointer)
- Flag: mixing sync.Mutex with channel for same data

**Section 3: Naming & Style**
- Check: MixedCaps (not underscores) for multi-word names
- Check: short variable names for narrow scope
- Check: acronyms all-caps (URL, HTTP, ID)
- Check: package names lowercase, single word
- Check: interface names: -er suffix for single method
- Check: exported names have doc comments
- Flag: stuttering (http.HTTPServer -> http.Server)

**Section 4: API Design**
- Check: interfaces defined at consumer, not provider
- Check: functions accept interfaces, return structs
- Check: options pattern for 3+ optional params
- Check: context.Context as first parameter
- Check: error as last return value
- Flag: interface with 5+ methods (too wide)
- Flag: exported interface with single implementation

**Section 5: Performance Pitfalls**
- Flag: string concatenation in loops (use strings.Builder)
- Flag: slice append without pre-allocation in known-size loops
- Flag: map without size hint for known sizes
- Flag: unnecessary pointer for small structs
- Flag: conversion between string and []byte in hot path
- Flag: defer in tight loops

**Section 6: Security**
- Check: user input validated at boundary
- Check: SQL queries use parameterized statements
- Check: file paths sanitized (filepath.Clean, no path traversal)
- Check: TLS configured for external connections
- Check: secrets not in source code or logs

**Section 7: Package & Project Structure**
- Check: no circular dependencies
- Check: internal/ for private packages
- Check: one package per directory
- Check: cmd/ for entry points
- Flag: package imports only for side effects without comment

## Skill 3: tester

**Trigger**: Writing, debugging, or improving Go tests
**Target size**: ~500 lines, ~45 rules

### Sections

| # | Topic | Sources | Rules |
|---|-------|---------|-------|
| 1 | Test Structure | LearnWithTests, LearningGo$15, GoBook$11 | 6-8 |
| 2 | Table-Driven Tests | Mistakes#82-84, LearnWithTests, GoBook$11 | 5-7 |
| 3 | Assertions & Helpers | LearningGo$15, LearnWithTests | 5-6 |
| 4 | Mocks & Test Doubles | LearnWithTests, LearningGo$15 | 6-8 |
| 5 | Integration Tests | Mistakes#84, LearningGo$15, CloudNative | 5-6 |
| 6 | Benchmarks | GoBook$11, Mistakes#89-92 | 5-6 |
| 7 | Fuzzing | LearningGo$15, Go fuzzing docs | 4-5 |
| 8 | Race Detection & Coverage | Concurrency$6, Mistakes#85-88 | 4-5 |

### Section Details

**Section 1: Test Structure**
- File naming: `foo_test.go` next to `foo.go`
- Test naming: `TestFunctionName_Scenario`
- Use `TestMain` for setup/teardown
- Build tags for integration tests: `//go:build integration`
- Test in same package for whitebox, `_test` package for blackbox
- Use `t.Parallel()` for independent tests

**Section 2: Table-Driven Tests**
- Name field in test case struct for clear `t.Run` output
- Cover happy path, edge cases, error cases
- Use `t.Run(tc.name, ...)` for subtests
- Keep test cases declarative (input/expected)
- Avoid logic in test case definitions

**Section 3: Assertions & Helpers**
- Use `t.Helper()` in all test helper functions
- Use `t.Cleanup()` for teardown (not defer in helpers)
- testify: `require` for fatal, `assert` for non-fatal
- Compare structs with `cmp.Diff` for readable diffs
- Use `t.Fatal` / `t.Error` correctly (stop vs continue)

**Section 4: Mocks & Test Doubles**
- Define interfaces at test site for narrow mocking
- Use `httptest.NewServer` for HTTP testing
- Fake implementations over mock frameworks when simple
- Generate mocks with mockgen for complex interfaces
- Use `httptest.NewRecorder` for handler testing
- Verify behavior, not implementation

**Section 5: Integration Tests**
- Separate with build tags, not file naming
- Use testcontainers-go for database/service dependencies
- `testing.Short()` to skip in fast mode
- Test database migrations in integration tests
- Clean up test data after each test

**Section 6: Benchmarks**
- Function naming: `BenchmarkFunctionName`
- Use `b.ResetTimer()` after setup
- Use `b.ReportAllocs()` for allocation tracking
- Avoid compiler optimization elimination (`sink` pattern)
- Compare with `benchstat` for statistical significance

**Section 7: Fuzzing**
- Function naming: `FuzzFunctionName`
- Add seed corpus for known edge cases
- Target parsing/validation functions
- Run with `-fuzz` flag, time-limited
- Regression: add crash inputs to corpus

**Section 8: Race Detection & Coverage**
- Always run `go test -race ./...` in CI
- Accept 5-10x slowdown for race detection
- Minimum coverage target: 80% for business logic
- Use `-coverprofile` + `go tool cover -html`
- Don't chase 100% coverage on generated/trivial code

## Cross-References Between Skills

| From | To | When |
|------|----|------|
| coder Section 5 (Concurrency) | reviewer Section 2 | Review concurrent code |
| coder Section 5 (Concurrency) | tester Section 8 | Race detection |
| coder Section 3 (Error Handling) | reviewer Section 1 | Review error handling |
| coder Section 2 (Interfaces) | tester Section 4 | Mock design |
| reviewer Section 5 (Performance) | tester Section 6 | Benchmark verification |

## Implementation Notes

- Each skill is a single SKILL.md file in `plugins/golang/skills/<name>/`
- Skills are standalone — AI agent loads only the relevant skill, not all three
- Cross-references use `/golang:<skill>` format
- Source abbreviations are defined once at the top of each skill
