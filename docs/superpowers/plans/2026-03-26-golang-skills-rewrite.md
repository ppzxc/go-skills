# golang Skills Rewrite Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rewrite 3 Go skills (coder, reviewer, tester) with Rule-based DO/DON'T format optimized for AI agents, sourced from 7 books + official Go docs.

**Architecture:** Each skill is a standalone SKILL.md file in `plugins/golang/skills/<name>/`. All skills share common format (DO/DON'T/WHEN sections with source tags). Skills are independent but cross-reference each other via `/golang:<skill>` slash commands.

**Tech Stack:** Markdown (SKILL.md files), Claude Code plugin system

**Spec:** `docs/superpowers/specs/2026-03-26-golang-skills-rewrite-design.md`

---

## Common Format Reference

All 3 skills use this template:

```markdown
---
name: <skill-name>
description: "<trigger>"
user_invocable: true
---

# <Title>

> Sources: [GoBook] The Go Programming Language, [Mistakes] 100 Go Mistakes, [Concurrency] Concurrency in Go, [LearningGo] Learning Go 2e, [LearnWithTests] Learn Go with Tests, [CloudNative] Cloud Native Go, [LetsGo] Let's Go, [EffectiveGo] Effective Go, [CodeReview] Go Code Review Comments

## Section N: <Topic>

### DO
- **Rule name**: Imperative rule [Source#N]
  ```go
  // 3-5 line example
  ```

### DON'T
- **Anti-pattern name**: What to avoid [Source#N]
  ```go
  // 3-5 line counter-example
  ```

### WHEN
- Conditional: "In X situation, do Y" [Source#N]
```

Format rules:
- Imperative voice ("Use X", "Return Y")
- One code example per rule, max 5 lines
- Source tag on every rule
- Priority ordering: important rules first
- Cross-reference related skills at section end

---

## Task 1: Write coder SKILL.md

**Files:**
- Rewrite: `plugins/golang/skills/coder/SKILL.md`

**Target:** ~550 lines, ~70 rules, 10 sections

- [ ] **Step 1: Write the complete coder SKILL.md**

Rewrite `plugins/golang/skills/coder/SKILL.md` with the following content. The file must follow the Common Format Reference above.

**Frontmatter:**
```yaml
---
name: coder
description: "Use when writing or refactoring Go production code — types, interfaces, error handling, concurrency, context, HTTP, service design, resilience, and performance patterns."
user_invocable: true
---
```

**Section 1: Types & Data Structures** (~8 rules)

DO rules:
- **Order struct fields by alignment**: Place larger types first (int64, pointers) then smaller (bool, byte) to minimize padding [Mistakes#91]
  ```go
  type Event struct {
      Timestamp int64  // 8 bytes
      ID        int32  // 4 bytes
      Active    bool   // 1 byte
  }
  ```
- **Pre-allocate slices with known capacity**: Use `make([]T, 0, n)` when size is known [Mistakes#22]
  ```go
  results := make([]Result, 0, len(input))
  ```
- **Provide map size hints**: Use `make(map[K]V, n)` for known sizes [Mistakes#27]
- **Use named types for domain concepts**: Create distinct types for type safety [GoBook#4]
  ```go
  type UserID int64
  type Email string
  ```
- **Design for useful zero values**: Ensure zero value is valid initial state [EffectiveGo]
  ```go
  var mu sync.Mutex // ready to use, no init needed
  ```
- **Use iota for enums**: Start with unknown/invalid zero value [Mistakes#21]
  ```go
  type Status int
  const (
      StatusUnknown Status = iota
      StatusActive
      StatusInactive
  )
  ```

DON'T rules:
- **Don't use generics prematurely**: Only when same logic applies to multiple concrete types [Mistakes#9]
- **Don't embed types just for convenience**: Embed only when the "is-a" relationship is intentional [Mistakes#10]

**Section 2: Interfaces** (~7 rules)

DO rules:
- **Accept interfaces, return structs**: Functions take interfaces as params, return concrete types [Mistakes#5, GoBook#7]
  ```go
  func Process(r io.Reader) (*Result, error) { ... }
  ```
- **Keep interfaces small**: 1-3 methods. Compose larger interfaces from smaller ones [EffectiveGo]
  ```go
  type ReadCloser interface {
      io.Reader
      io.Closer
  }
  ```
- **Define interfaces at consumer side**: Consumers declare what they need, producers satisfy implicitly [Mistakes#6]
- **Use type assertions with ok pattern**: Always use comma-ok to avoid panics [GoBook#7]
  ```go
  if v, ok := val.(Stringer); ok { ... }
  ```

DON'T rules:
- **Don't export interfaces for implementation**: Export only when consumers in other packages need them [Mistakes#7]
- **Don't create interfaces with 5+ methods**: Split into smaller focused interfaces [CodeReview]
- **Don't use interface{}/any as lazy typing**: Use concrete types or generics when type is known [Mistakes#8]

**Section 3: Error Handling** (~9 rules)

DO rules:
- **Always check returned errors**: Never discard with `_` unless explicitly justified [Mistakes#48]
- **Wrap errors with context**: Use `%w` for wrappable, `%v` for opaque [Mistakes#49]
  ```go
  return fmt.Errorf("fetch user %d: %w", id, err)
  ```
- **Use sentinel errors for expected conditions**: Package-level `var` for errors callers check [Mistakes#50]
  ```go
  var ErrNotFound = errors.New("not found")
  ```
- **Use custom error types for rich context**: When callers need structured info [Mistakes#51]
  ```go
  type ValidationError struct {
      Field   string
      Message string
  }
  func (e *ValidationError) Error() string { ... }
  ```
- **Match errors with errors.Is/errors.As**: Never compare with `==` for wrapped errors [Mistakes#52]
- **Handle errors once**: Either log or return, never both [Mistakes#53]

DON'T rules:
- **Don't panic for expected errors**: Panic only for programmer bugs (impossible states) [GoBook#5]
- **Don't start error messages with capital or end with punctuation**: Errors compose [CodeReview]
- **Don't ignore errors in deferred Close()**: Use named return or helper [Mistakes#54]
  ```go
  defer func() { err = errors.Join(err, f.Close()) }()
  ```

**Section 4: Functions & Methods** (~6 rules)

DO rules:
- **Use functional options for complex config**: When constructor has 3+ optional params [Mistakes#11]
  ```go
  func NewServer(addr string, opts ...Option) *Server { ... }
  ```
- **Use defer for resource cleanup**: Immediately after acquiring resource [GoBook#5]
- **Be consistent with receiver types**: All methods on a type use same receiver (pointer or value) [CodeReview]

DON'T rules:
- **Don't use named returns for short functions**: Only for documenting return values in complex signatures [Mistakes#15]
- **Don't exceed 4-5 parameters**: Group into struct or use options pattern [Mistakes#12]
- **Don't use naked returns**: Always specify what you're returning [Mistakes#16]

**Section 5: Concurrency** (~11 rules)

DO rules:
- **Give every goroutine a clear exit path**: Use context, done channel, or WaitGroup [Concurrency#4, Mistakes#58]
  ```go
  go func() {
      select {
      case <-ctx.Done():
          return
      case msg := <-ch:
          process(msg)
      }
  }()
  ```
- **Use errgroup for concurrent tasks with error handling**: Replaces manual WaitGroup + error collection [Mistakes#67]
  ```go
  g, ctx := errgroup.WithContext(ctx)
  g.Go(func() error { return task1(ctx) })
  g.Go(func() error { return task2(ctx) })
  return g.Wait()
  ```
- **Use mutex for shared state, channel for communication**: Choose based on purpose [Concurrency#3]
- **Specify channel direction in signatures**: `<-chan T` for receive, `chan<- T` for send [GoBook#8]
  ```go
  func consume(ch <-chan Event) { ... }
  ```
- **Use sync.Once for one-time initialization**: Thread-safe lazy init [Mistakes#72]
- **Use select with context for all blocking operations**: Prevent goroutine leaks [Concurrency#4]

DON'T rules:
- **Don't start goroutines in init()**: No lifecycle control possible [Mistakes#60]
- **Don't capture loop variables by reference in goroutines**: Pass as parameter [Mistakes#63]
- **Don't copy sync primitives**: Pass mutex, WaitGroup, etc. by pointer [Mistakes#73]
- **Don't mix mutex and channel for the same data**: Pick one synchronization strategy [Concurrency#3]
- **Don't use unbuffered channels without guaranteed receiver**: Blocks sender forever [Mistakes#65]

> See `/golang:reviewer` Section 2 for concurrency review checklist. See `/golang:tester` Section 8 for race detection.

**Section 6: Context** (~5 rules)

DO rules:
- **Pass context as first parameter named ctx**: Convention throughout stdlib [EffectiveGo]
  ```go
  func FetchUser(ctx context.Context, id int64) (*User, error) { ... }
  ```
- **Use context.WithTimeout for external calls**: Always bound network/IO operations [Mistakes#74]
  ```go
  ctx, cancel := context.WithTimeout(ctx, 5*time.Second)
  defer cancel()
  ```
- **Respect cancellation in loops**: Check ctx.Done() or ctx.Err() in long iterations [Mistakes#76]

DON'T rules:
- **Don't store context in structs**: Pass explicitly through call chain [Mistakes#75]
- **Don't use context.Value for control flow**: Only for request-scoped metadata (trace ID, auth) [Mistakes#77]

**Section 7: HTTP & Web** (~7 rules)

DO rules:
- **Use Go 1.22+ ServeMux patterns**: Method-aware routing with `mux.HandleFunc("GET /api/users", handler)` [LetsGo]
  ```go
  mux := http.NewServeMux()
  mux.HandleFunc("GET /api/users/{id}", getUser)
  ```
- **Use middleware for cross-cutting concerns**: Logging, auth, recovery, CORS [LetsGo]
  ```go
  func logging(next http.Handler) http.Handler {
      return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
          slog.Info("request", "method", r.Method, "path", r.URL.Path)
          next.ServeHTTP(w, r)
      })
  }
  ```
- **Decode JSON with DisallowUnknownFields**: Reject unexpected input [LetsGo]
  ```go
  dec := json.NewDecoder(r.Body)
  dec.DisallowUnknownFields()
  ```
- **Set timeouts on http.Server**: ReadTimeout, WriteTimeout, IdleTimeout [LetsGo]
  ```go
  srv := &http.Server{Addr: ":8080", Handler: mux, ReadTimeout: 5 * time.Second, WriteTimeout: 10 * time.Second}
  ```

DON'T rules:
- **Don't use default http.Client**: Always set timeout [Mistakes#78]
  ```go
  client := &http.Client{Timeout: 10 * time.Second}
  ```
- **Don't write response body after error status**: Write header and body together [LetsGo]
- **Don't read request body without size limit**: Use `http.MaxBytesReader` [LetsGo]

**Section 8: Service Design** (~6 rules)

DO rules:
- **Use constructor injection for dependencies**: Pass interfaces through constructors [CloudNative#4]
  ```go
  func NewUserService(repo UserRepository, logger *slog.Logger) *UserService {
      return &UserService{repo: repo, logger: logger}
  }
  ```
- **Use slog for structured logging**: Key-value pairs, log levels [LearningGo#15]
  ```go
  slog.Info("user created", "user_id", user.ID, "email", user.Email)
  ```
- **Load config from environment**: Use os.Getenv or struct-based config loader [CloudNative#5]
- **Separate transport, business, and storage layers**: Each layer has clear interface [CloudNative#4]

DON'T rules:
- **Don't use global state for dependencies**: No package-level DB connections or loggers [Mistakes#1]
- **Don't log sensitive data**: Never log passwords, tokens, PII [LetsGo]

**Section 9: Resilience** (~5 rules)

DO rules:
- **Implement exponential backoff with jitter for retries**: Prevent thundering herd [CloudNative#5]
  ```go
  delay := baseDelay * time.Duration(math.Pow(2, float64(attempt)))
  jitter := time.Duration(rand.Int63n(int64(delay / 2)))
  time.Sleep(delay + jitter)
  ```
- **Handle OS signals for graceful shutdown**: Drain connections before exit [CloudNative#6]
  ```go
  ctx, stop := signal.NotifyContext(context.Background(), syscall.SIGINT, syscall.SIGTERM)
  defer stop()
  ```
- **Use circuit breaker for external dependencies**: Fail fast when downstream is unhealthy [CloudNative#5]

DON'T rules:
- **Don't retry without backoff**: Fixed-interval retry causes cascading failures [CloudNative#5]
- **Don't ignore shutdown context**: Always pass shutdown context to server.Shutdown(ctx) [CloudNative#6]

**Section 10: Performance** (~7 rules)

DO rules:
- **Use strings.Builder for loop concatenation**: O(n) instead of O(n^2) [Mistakes#92]
  ```go
  var b strings.Builder
  for _, s := range parts { b.WriteString(s) }
  ```
- **Pre-allocate when size is known**: Slices, maps, builders [Mistakes#22]
- **Profile before optimizing**: Use pprof to identify actual bottlenecks [GoBook#11]
  ```go
  import _ "net/http/pprof" // in main, behind admin port
  ```
- **Use sync.Pool for high-churn temporary objects**: Reduce GC pressure [Mistakes#96]

DON'T rules:
- **Don't optimize without measurement**: Premature optimization wastes effort [GoBook#11]
- **Don't use pointer receivers for small read-only structs**: Value receivers can be stack-allocated [Mistakes#95]
- **Don't defer in tight loops**: Defer runs at function return, not loop iteration [Mistakes#93]

- [ ] **Step 2: Verify coder SKILL.md**

Run:
```bash
wc -l plugins/golang/skills/coder/SKILL.md
head -5 plugins/golang/skills/coder/SKILL.md
grep -c "^### DO\|^### DON'T\|^### WHEN" plugins/golang/skills/coder/SKILL.md
grep "/golang:" plugins/golang/skills/coder/SKILL.md
```

Expected:
- Line count: 500-600
- Frontmatter: `name: coder`
- 10+ DO/DON'T/WHEN section headers
- Cross-references use `/golang:reviewer` and `/golang:tester`

- [ ] **Step 3: Commit**

```bash
git add plugins/golang/skills/coder/SKILL.md
git commit -m "feat(golang:coder): rewrite with rule-based DO/DON'T format

Rewrite coder skill based on 7 books + Go official docs.
Switch from prose-heavy to AI-optimized rule-based format.
~70 rules across 10 sections."
```

---

## Task 2: Write reviewer SKILL.md

**Files:**
- Rewrite: `plugins/golang/skills/reviewer/SKILL.md`

**Target:** ~500 lines, ~50 rules, 7 sections

- [ ] **Step 1: Write the complete reviewer SKILL.md**

Rewrite `plugins/golang/skills/reviewer/SKILL.md` with the following content.

**Frontmatter:**
```yaml
---
name: reviewer
description: "Use when reviewing Go code in pull requests or auditing Go codebases — checklists for error handling, concurrency safety, naming conventions, API design, performance pitfalls, security, and package structure."
user_invocable: true
---
```

**Section 1: Error Handling Review** (~9 rules)

CHECK rules (things that must be present):
- **Every error return is checked**: No `_` for error returns without explicit comment [Mistakes#48]
  ```go
  // BAD
  val, _ := strconv.Atoi(s)
  // GOOD
  val, err := strconv.Atoi(s)
  if err != nil { return fmt.Errorf("parse age: %w", err) }
  ```
- **Errors wrapped with context**: Bare `return err` should add wrapping context [Mistakes#49]
- **Consistent wrapping strategy**: Either `%w` (matchable) or `%v` (opaque) per module boundary [Mistakes#50]
- **Sentinel errors are package-level vars**: Not created inline in functions [Mistakes#50]
  ```go
  // GOOD: package level
  var ErrNotFound = errors.New("not found")
  // BAD: inline
  func Get() error { return errors.New("not found") }
  ```
- **Error messages lowercase, no punctuation**: For composability in error chains [CodeReview]

FLAG rules (code smells to raise):
- **errors.New inside loops**: Should be package-level var [Mistakes#50]
- **panic in library code**: Only acceptable in truly impossible states [GoBook#5]
- **Log AND return same error**: Handle once — log or propagate, not both [Mistakes#53]
- **Ignoring deferred Close() errors**: Use errors.Join or named returns [Mistakes#54]

> See `/golang:coder` Section 3 for error handling patterns.

**Section 2: Concurrency Safety** (~11 rules)

CHECK rules:
- **Every goroutine has exit path**: Context cancellation, done channel, or WaitGroup [Concurrency#4]
- **Shared state has synchronization**: Mutex or channel, documented which [Concurrency#3]
- **WaitGroup.Add before goroutine launch**: Not inside the goroutine [Mistakes#66]
  ```go
  // GOOD
  wg.Add(1)
  go func() { defer wg.Done(); work() }()
  // BAD
  go func() { wg.Add(1); defer wg.Done(); work() }()
  ```
- **defer mu.Unlock() immediately after Lock()**: No code between Lock and defer Unlock [Mistakes#70]
  ```go
  mu.Lock()
  defer mu.Unlock()
  ```
- **Channel direction in function parameters**: Restrict to send or receive [GoBook#8]

FLAG rules:
- **Unbuffered channel without guaranteed receiver**: Blocks sender, potential deadlock [Mistakes#65]
- **sync primitives copied**: Mutex, WaitGroup, Cond must pass by pointer [Mistakes#73]
- **Closure captures loop variable in goroutine**: Pass as parameter instead [Mistakes#63]
- **Mixing mutex and channel for same data**: Pick one synchronization approach [Concurrency#3]
- **No select with context in blocking goroutine**: Missing cancellation path [Concurrency#4]
- **Goroutine launched without clear ownership**: Who waits for it? Who cancels it? [Concurrency#4]

> See `/golang:tester` Section 8 for race detection with `go test -race`.

**Section 3: Naming & Style** (~7 rules)

CHECK rules:
- **MixedCaps, no underscores**: `getUser` not `get_user` [EffectiveGo]
- **Short names for narrow scope**: `i` for loop index, `r` for reader in 3-line func [CodeReview]
- **Acronyms all-caps**: `HTTP`, `URL`, `ID`, `API` — not `Http`, `Url`, `Id` [CodeReview]
- **Package names lowercase, single word**: `http` not `httpUtils` or `http_utils` [EffectiveGo]
- **Single-method interfaces use -er suffix**: `Reader`, `Writer`, `Closer` [EffectiveGo]
- **Exported names have doc comments**: Starting with the name itself [CodeReview]

FLAG rules:
- **Name stuttering**: `http.HTTPServer` should be `http.Server` [CodeReview]

**Section 4: API Design** (~7 rules)

CHECK rules:
- **Interfaces at consumer side**: Consumer declares, producer satisfies [Mistakes#6]
- **Accept interfaces, return structs**: Flexible inputs, concrete outputs [Mistakes#5]
- **Options pattern for 3+ optional params**: Functional options or config struct [Mistakes#11]
- **context.Context as first parameter**: Standard Go convention [EffectiveGo]
- **error as last return value**: Go convention for all fallible functions [EffectiveGo]

FLAG rules:
- **Interface with 5+ methods**: Too wide, split into focused interfaces [CodeReview]
- **Exported interface with single implementation**: Unnecessary abstraction [Mistakes#7]

> See `/golang:coder` Section 2 for interface design patterns.

**Section 5: Performance Pitfalls** (~7 rules)

FLAG rules:
- **String concatenation in loops**: Use `strings.Builder` [Mistakes#92]
  ```go
  // BAD
  s := ""
  for _, part := range parts { s += part }
  // GOOD
  var b strings.Builder
  for _, part := range parts { b.WriteString(part) }
  ```
- **Slice append without pre-allocation**: Use `make([]T, 0, n)` when size known [Mistakes#22]
- **Map without size hint**: Use `make(map[K]V, n)` for known sizes [Mistakes#27]
- **Unnecessary pointer for small read-only structs**: Value receivers allow stack allocation [Mistakes#95]
- **string/[]byte conversion in hot path**: Causes allocation, reuse buffer [Mistakes#92]
- **defer in tight loops**: Defers run at function exit, not iteration end [Mistakes#93]
- **Range loop copies large values**: Use index or pointer for large structs [Mistakes#30]
  ```go
  // BAD: copies LargeStruct each iteration
  for _, v := range largeSlice { use(v) }
  // GOOD: avoids copy
  for i := range largeSlice { use(&largeSlice[i]) }
  ```

> See `/golang:tester` Section 6 for benchmarking to measure actual impact.

**Section 6: Security** (~5 rules)

CHECK rules:
- **User input validated at boundary**: Validate and sanitize before business logic [LetsGo]
- **SQL uses parameterized queries**: Never string concatenation for SQL [LetsGo]
  ```go
  // GOOD
  db.QueryRow("SELECT name FROM users WHERE id = $1", id)
  // BAD
  db.QueryRow("SELECT name FROM users WHERE id = " + id)
  ```
- **File paths sanitized**: Use `filepath.Clean`, check for `..` traversal [LetsGo]
- **TLS for external connections**: Validate certificates for production [CloudNative#6]
- **Secrets not in source or logs**: Use environment variables, secret managers [LetsGo]

**Section 7: Package & Project Structure** (~5 rules)

CHECK rules:
- **No circular package dependencies**: Packages form a DAG [EffectiveGo]
- **internal/ for private packages**: Not importable outside module [Mistakes#3]
- **One package per directory**: Go requirement, don't fight it [GoBook#10]
- **cmd/ for entry points**: Standard Go layout for multi-binary projects [Mistakes#4]

FLAG rules:
- **Side-effect import without comment**: Blank imports need justification [CodeReview]
  ```go
  import _ "image/png" // register PNG decoder
  ```

- [ ] **Step 2: Verify reviewer SKILL.md**

Run:
```bash
wc -l plugins/golang/skills/reviewer/SKILL.md
head -5 plugins/golang/skills/reviewer/SKILL.md
grep -c "^### CHECK\|^### FLAG" plugins/golang/skills/reviewer/SKILL.md
grep "/golang:" plugins/golang/skills/reviewer/SKILL.md
```

Expected:
- Line count: 400-550
- Frontmatter: `name: reviewer`
- 7+ CHECK/FLAG section headers
- Cross-references to `/golang:coder` and `/golang:tester`

- [ ] **Step 3: Commit**

```bash
git add plugins/golang/skills/reviewer/SKILL.md
git commit -m "feat(golang:reviewer): rewrite with CHECK/FLAG review format

Rewrite reviewer skill based on 7 books + Go official docs.
Switch from bare checklist to structured CHECK/FLAG format.
~50 rules across 7 sections."
```

---

## Task 3: Write tester SKILL.md

**Files:**
- Rewrite: `plugins/golang/skills/tester/SKILL.md`

**Target:** ~500 lines, ~45 rules, 8 sections

- [ ] **Step 1: Write the complete tester SKILL.md**

Rewrite `plugins/golang/skills/tester/SKILL.md` with the following content.

**Frontmatter:**
```yaml
---
name: tester
description: "Use when writing, debugging, or improving Go tests — table-driven tests, benchmarks, fuzzing, mocks, integration tests, race detection, and coverage."
user_invocable: true
---
```

**Section 1: Test Structure** (~7 rules)

DO rules:
- **Place test files next to source**: `foo_test.go` beside `foo.go` [LearnWithTests]
- **Name tests descriptively**: `TestFunctionName_Scenario` with `t.Run` subtests [GoBook#11]
  ```go
  func TestParseAge_ValidInput(t *testing.T) { ... }
  func TestParseAge_NegativeNumber(t *testing.T) { ... }
  ```
- **Use TestMain for package-level setup/teardown**: One per package [LearningGo#15]
  ```go
  func TestMain(m *testing.M) {
      setup()
      code := m.Run()
      teardown()
      os.Exit(code)
  }
  ```
- **Use build tags for integration tests**: `//go:build integration` [LearningGo#15]
- **Use t.Parallel() for independent tests**: Speed up test suites [LearnWithTests]
  ```go
  func TestFoo(t *testing.T) {
      t.Parallel()
      // ...
  }
  ```

DON'T rules:
- **Don't test private functions directly**: Test through public API; use `_test` package for blackbox [LearnWithTests]
- **Don't put test logic in TestMain**: Only setup/teardown, actual tests go in Test functions [LearningGo#15]

**Section 2: Table-Driven Tests** (~6 rules)

DO rules:
- **Use name field for clear subtest output**: Identifies failing case in output [GoBook#11]
  ```go
  tests := []struct {
      name     string
      input    string
      expected int
      wantErr  bool
  }{
      {name: "valid number", input: "42", expected: 42},
      {name: "negative",     input: "-1", expected: -1},
      {name: "invalid",      input: "abc", wantErr: true},
  }
  for _, tc := range tests {
      t.Run(tc.name, func(t *testing.T) { ... })
  }
  ```
- **Cover happy path, edge cases, and error cases**: Minimum three categories [LearnWithTests]
- **Keep test cases declarative**: Input and expected output, no logic in test struct [Mistakes#82]

DON'T rules:
- **Don't put assertions inside test case struct**: Assert in the loop body [Mistakes#83]
- **Don't share mutable state between subtests**: Each t.Run gets own setup [LearnWithTests]
- **Don't skip error case tests**: Test that wrong input produces correct error [Mistakes#84]

**Section 3: Assertions & Helpers** (~6 rules)

DO rules:
- **Mark helpers with t.Helper()**: Correct line numbers in failure output [LearningGo#15]
  ```go
  func assertStatus(t *testing.T, got, want int) {
      t.Helper()
      if got != want { t.Errorf("status = %d, want %d", got, want) }
  }
  ```
- **Use t.Cleanup() for teardown in helpers**: Runs after test, not at function return [LearningGo#15]
  ```go
  func createTempDB(t *testing.T) *DB {
      t.Helper()
      db := openDB()
      t.Cleanup(func() { db.Close() })
      return db
  }
  ```
- **Use require for fatal checks, assert for non-fatal**: testify convention [LearnWithTests]
  ```go
  require.NoError(t, err) // stops test on failure
  assert.Equal(t, expected, got) // continues on failure
  ```
- **Use cmp.Diff for struct comparison**: Readable diff output [LearningGo#15]
  ```go
  if diff := cmp.Diff(want, got); diff != "" { t.Errorf("mismatch (-want +got):\n%s", diff) }
  ```

DON'T rules:
- **Don't use t.Fatal in goroutines**: Causes panic; use t.Error and return [GoBook#11]
- **Don't assert on string representations**: Use typed comparisons [LearnWithTests]

**Section 4: Mocks & Test Doubles** (~7 rules)

DO rules:
- **Define narrow interfaces at test site**: Only methods the test needs [LearnWithTests]
  ```go
  // In test file, not production code
  type userGetter interface {
      GetUser(ctx context.Context, id int64) (*User, error)
  }
  ```
- **Use httptest for HTTP testing**: Server and ResponseRecorder [LearnWithTests]
  ```go
  srv := httptest.NewServer(handler)
  defer srv.Close()
  resp, err := http.Get(srv.URL + "/api/users")
  ```
- **Prefer fakes over mock frameworks**: Simple struct implementing interface [LearnWithTests]
  ```go
  type fakeRepo struct{ users map[int64]*User }
  func (f *fakeRepo) GetUser(_ context.Context, id int64) (*User, error) {
      u, ok := f.users[id]; if !ok { return nil, ErrNotFound }; return u, nil
  }
  ```
- **Use httptest.NewRecorder for handler tests**: Test handlers without network [LearnWithTests]
  ```go
  rec := httptest.NewRecorder()
  req := httptest.NewRequest("GET", "/api/users/1", nil)
  handler.ServeHTTP(rec, req)
  assert.Equal(t, 200, rec.Code)
  ```

DON'T rules:
- **Don't mock types you don't own**: Wrap external types in your own interface [LearnWithTests]
- **Don't verify mock call counts by default**: Test behavior (outputs), not implementation (calls) [LearnWithTests]
- **Don't generate mocks for simple interfaces**: Hand-write fakes for 1-2 method interfaces [LearningGo#15]

> See `/golang:coder` Section 2 for interface design patterns used in mocks.

**Section 5: Integration Tests** (~5 rules)

DO rules:
- **Separate with build tags**: `//go:build integration` at file top [LearningGo#15]
  ```go
  //go:build integration

  package repo_test

  func TestUserRepo_Create(t *testing.T) { ... }
  ```
- **Use testcontainers-go for dependencies**: Spin up real DB/Redis in tests [CloudNative]
  ```go
  container, err := postgres.Run(ctx, "postgres:16")
  t.Cleanup(func() { container.Terminate(ctx) })
  connStr, _ := container.ConnectionString(ctx)
  ```
- **Use testing.Short() for fast mode**: Skip slow tests with `go test -short` [GoBook#11]
  ```go
  if testing.Short() { t.Skip("skipping integration test") }
  ```

DON'T rules:
- **Don't share database state between tests**: Each test sets up and tears down its data [CloudNative]
- **Don't use file-based separation for test types**: Build tags are the Go-idiomatic way [LearningGo#15]

**Section 6: Benchmarks** (~6 rules)

DO rules:
- **Name benchmarks descriptively**: `BenchmarkFunctionName_Scenario` [GoBook#11]
- **Call b.ResetTimer() after setup**: Don't measure initialization [GoBook#11]
  ```go
  func BenchmarkSort(b *testing.B) {
      data := generateData(10000)
      b.ResetTimer()
      for i := 0; i < b.N; i++ { sort.Ints(data) }
  }
  ```
- **Use b.ReportAllocs()**: Track allocations per operation [Mistakes#89]
- **Prevent compiler eliminating results**: Assign to package-level sink variable [Mistakes#90]
  ```go
  var sink int
  func BenchmarkFoo(b *testing.B) {
      for i := 0; i < b.N; i++ { sink = Foo() }
  }
  ```

DON'T rules:
- **Don't benchmark without benchstat**: Single run is noise, use `benchstat old.txt new.txt` [GoBook#11]
- **Don't benchmark in -race mode**: Race detector adds overhead, skews results [Mistakes#91]

> See `/golang:reviewer` Section 5 for performance review checklist.

**Section 7: Fuzzing** (~5 rules)

DO rules:
- **Target parsing and validation functions**: Where invalid input causes crashes [LearningGo#15]
  ```go
  func FuzzParseJSON(f *testing.F) {
      f.Add([]byte(`{"name":"test"}`))
      f.Fuzz(func(t *testing.T, data []byte) {
          var v map[string]any
          _ = json.Unmarshal(data, &v) // should not panic
      })
  }
  ```
- **Add seed corpus for known edge cases**: f.Add() with representative inputs [LearningGo#15]
- **Add crash inputs to testdata/corpus**: Regression prevention [Go fuzzing docs]

DON'T rules:
- **Don't fuzz functions with external dependencies**: Fuzz pure logic only [LearningGo#15]
- **Don't run fuzzing in CI without time limit**: Use `-fuzztime 30s` [Go fuzzing docs]

**Section 8: Race Detection & Coverage** (~5 rules)

DO rules:
- **Run -race in CI always**: `go test -race ./...` [Concurrency#6]
- **Use -count=1 to bypass test cache**: Fresh runs for race detection [Mistakes#85]
  ```bash
  go test -race -count=1 ./...
  ```
- **Generate coverage profiles**: `go test -coverprofile=coverage.out ./...` [GoBook#11]

DON'T rules:
- **Don't chase 100% coverage**: Focus on business logic, skip generated/trivial code [Mistakes#88]
- **Don't ignore race detector findings**: Every race is a real bug, not a false positive [Concurrency#6]

> See `/golang:reviewer` Section 2 for concurrency safety review checklist.

- [ ] **Step 2: Verify tester SKILL.md**

Run:
```bash
wc -l plugins/golang/skills/tester/SKILL.md
head -5 plugins/golang/skills/tester/SKILL.md
grep -c "^### DO\|^### DON'T" plugins/golang/skills/tester/SKILL.md
grep "/golang:" plugins/golang/skills/tester/SKILL.md
```

Expected:
- Line count: 400-550
- Frontmatter: `name: tester`
- 8+ DO/DON'T section headers
- Cross-references to `/golang:coder` and `/golang:reviewer`

- [ ] **Step 3: Commit**

```bash
git add plugins/golang/skills/tester/SKILL.md
git commit -m "feat(golang:tester): rewrite with rule-based DO/DON'T format

Rewrite tester skill based on 7 books + Go official docs.
Switch from prose-heavy to AI-optimized rule-based format.
~45 rules across 8 sections."
```

---

## Task 4: Cross-Reference Verification & Final Commit

**Files:**
- Verify: `plugins/golang/skills/coder/SKILL.md`
- Verify: `plugins/golang/skills/reviewer/SKILL.md`
- Verify: `plugins/golang/skills/tester/SKILL.md`

- [ ] **Step 1: Verify all cross-references are valid**

Run:
```bash
grep -n "/golang:" plugins/golang/skills/*/SKILL.md
```

Verify each reference points to a section that exists in the target skill:
- `/golang:reviewer` Section 2 → reviewer has Section 2: Concurrency Safety ✓
- `/golang:tester` Section 8 → tester has Section 8: Race Detection ✓
- `/golang:coder` Section 3 → coder has Section 3: Error Handling ✓
- `/golang:coder` Section 2 → coder has Section 2: Interfaces ✓
- `/golang:tester` Section 6 → tester has Section 6: Benchmarks ✓
- `/golang:reviewer` Section 5 → reviewer has Section 5: Performance Pitfalls ✓

- [ ] **Step 2: Verify line counts and format**

Run:
```bash
echo "=== Line counts ==="
wc -l plugins/golang/skills/*/SKILL.md

echo "=== Frontmatter names ==="
head -3 plugins/golang/skills/*/SKILL.md

echo "=== Rule count per file ==="
for f in plugins/golang/skills/*/SKILL.md; do
    echo "$f:"
    grep -c "^\- \*\*" "$f"
done
```

Expected:
- Each file: 400-600 lines
- Names: `coder`, `reviewer`, `tester`
- Rule count: coder ~70, reviewer ~50, tester ~45

- [ ] **Step 3: Final summary commit (if needed)**

If any fixes were made during verification:
```bash
git add plugins/golang/skills/*/SKILL.md
git commit -m "fix(golang): fix cross-references and formatting in skills"
```
