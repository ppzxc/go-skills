---
name: reviewer
description: "Use when reviewing Go code in pull requests or auditing Go codebases — checklists for error handling, concurrency safety, naming conventions, API design, performance pitfalls, security, and package structure."
user_invocable: true
---

# Go Reviewer

CHECK/FLAG review checklist for production Go. Every rule cites a canonical source.

**Sources:** [GoBook] The Go Programming Language, [Mistakes] 100 Go Mistakes and How to Avoid Them, [Concurrency] Concurrency in Go, [LearningGo] Learning Go 2nd Ed, [CloudNative] Cloud Native Go, [LetsGo] Let's Go / Let's Go Further, [EffectiveGo] Effective Go, [CodeReview] Go Code Review Comments

---

## Section 1: Error Handling Review

### CHECK (things that must be present/correct)

- **Verify every error return is checked**: No unhandled error values from function calls [Mistakes#48]
  ```go
  // GOOD
  f, err := os.Open(path)
  if err != nil {
      return fmt.Errorf("open config: %w", err)
  }
  ```

- **Verify errors are wrapped with context**: Bare returns lose call-site information [Mistakes#49]
  ```go
  // GOOD
  row, err := db.QueryRow(ctx, query, id)
  if err != nil {
      return fmt.Errorf("get user %d: %w", id, err)
  }
  ```

- **Verify consistent wrapping strategy (%w vs %v)**: Use %w to preserve the chain, %v only to intentionally break it [Mistakes#50]
  ```go
  // GOOD — deliberate choice documented
  return fmt.Errorf("validate input: %w", err)   // callers can errors.Is
  return fmt.Errorf("internal: %v", err)          // hide implementation detail
  ```

- **Verify sentinel errors are package-level vars**: Sentinels declared inside functions are unreachable to callers [LearningGo#9]
  ```go
  // GOOD
  var ErrNotFound = errors.New("not found")
  var ErrConflict = errors.New("conflict")
  ```

- **Verify error messages are lowercase with no punctuation**: Error strings compose inside wrapping chains [CodeReview]
  ```go
  // GOOD
  errors.New("connection refused")
  fmt.Errorf("parse config: %w", err)
  ```

### FLAG (code smells to raise)

- **Flag errors.New inside loops**: Allocates a new error value every iteration [Mistakes#48]
  ```go
  // BAD — allocates on every iteration
  for _, v := range items {
      if v < 0 { return errors.New("negative value") }
  }
  ```

- **Flag panic in library code**: Libraries must return errors, not crash the caller [EffectiveGo]
  ```go
  // BAD — library function should not panic
  func Parse(data []byte) Config {
      if len(data) == 0 { panic("empty data") }
  }
  ```

- **Flag logging AND returning the same error**: Produces duplicate noise in logs [Mistakes#49]
  ```go
  // BAD — caller will also log the returned error
  if err != nil {
      log.Printf("failed: %v", err)
      return fmt.Errorf("operation failed: %w", err)
  }
  ```

- **Flag ignoring deferred Close() errors**: Write-path Close can fail and lose data [Mistakes#50]
  ```go
  // BAD — file write may be incomplete
  defer f.Close()
  // GOOD — check error on write path
  defer func() { closeErr = f.Close() }()
  ```

> Cross-ref: See `/golang:coder` Section 3 for full error handling patterns.

---

## Section 2: Concurrency Safety

### CHECK (things that must be present/correct)

- **Verify every goroutine has an exit path**: Context cancellation, channel close, or explicit signal [Concurrency#4]
  ```go
  // GOOD — goroutine exits when ctx is cancelled
  go func() {
      for { select { case <-ctx.Done(): return
          case msg := <-ch: process(msg) } }
  }()
  ```

- **Verify shared state has synchronization**: Every read/write to shared mutable data protected [Mistakes#58]
  ```go
  // GOOD
  var mu sync.Mutex
  mu.Lock()
  counter++
  mu.Unlock()
  ```

- **Verify WaitGroup.Add is called before goroutine launch**: Add inside goroutine races with Wait [Concurrency#3]
  ```go
  // GOOD
  var wg sync.WaitGroup
  wg.Add(len(items))
  for _, item := range items {
      go func(it Item) { defer wg.Done(); process(it) }(item)
  }
  wg.Wait()
  ```

- **Verify defer mu.Unlock() immediately after Lock()**: Gap between Lock and defer risks missing unlock on panic [Mistakes#69]
  ```go
  // GOOD
  mu.Lock()
  defer mu.Unlock()
  return shared.value
  ```

- **Verify channel direction in function parameters**: Restrict to send-only or receive-only where possible [GoBook#8]
  ```go
  // GOOD — direction enforced by compiler
  func produce(out chan<- int) { out <- 42 }
  func consume(in <-chan int)  { v := <-in }
  ```

### FLAG (code smells to raise)

- **Flag unbuffered channel without guaranteed receiver**: Sender blocks forever if no goroutine reads [Concurrency#3]
  ```go
  // BAD — if nobody reads ch, this goroutine leaks
  ch := make(chan int)
  go func() { ch <- expensiveCalc() }()
  ```

- **Flag sync primitives copied by value**: Copying a mutex or WaitGroup breaks internal state [Mistakes#73]
  ```go
  // BAD — mu is copied, lock is meaningless
  type Cache struct{ mu sync.Mutex; data map[string]string }
  func (c Cache) Get(k string) string { c.mu.Lock(); defer c.mu.Unlock(); return c.data[k] }
  // Use pointer receiver: func (c *Cache) Get(...)
  ```

- **Flag closure capturing loop variable in goroutine**: Variable is shared across iterations in Go < 1.22 [Mistakes#63]
  ```go
  // BAD — all goroutines see last value of v (Go < 1.22)
  for _, v := range items {
      go func() { process(v) }()
  }
  ```

- **Flag mixing mutex and channel for same data**: Pick one synchronization mechanism per data path [Concurrency#4]
  ```go
  // BAD — two mechanisms guarding the same counter
  mu.Lock()
  counter++
  mu.Unlock()
  ch <- counter // also sending through channel
  ```

- **Flag blocking goroutine without select on context**: Goroutine cannot be cancelled [Concurrency#5]
  ```go
  // BAD — no way to cancel this goroutine
  go func() {
      result := <-longRunningCh
      process(result)
  }()
  ```

- **Flag goroutine launched without clear ownership**: Caller must know who stops it [Concurrency#4]
  ```go
  // BAD — fire and forget, no shutdown mechanism
  func StartProcessor() {
      go func() { for { processNext() } }()
  }
  ```

> Cross-ref: See `/golang:tester` Section 8 for race detection testing patterns.

---

## Section 3: Naming & Style

### CHECK (things that must be present/correct)

- **Verify MixedCaps with no underscores**: Go convention is MixedCaps or mixedCaps, never snake_case [CodeReview]
  ```go
  // GOOD
  type UserAccount struct{}
  var maxRetryCount int
  func parseRequestBody() {}
  ```

- **Verify short names for narrow scope**: Single-letter or abbreviated names for local variables with small scope [EffectiveGo]
  ```go
  // GOOD — short scope, short name
  for i, v := range items { use(i, v) }
  func (s *Server) handleReq(w http.ResponseWriter, r *http.Request) {}
  ```

- **Verify acronyms are all-caps**: HTTP, URL, ID, API, SQL — not Http, Url, Id [CodeReview]
  ```go
  // GOOD
  type HTTPClient struct{}
  var userID int
  func ServeHTTP(w http.ResponseWriter, r *http.Request) {}
  ```

- **Verify package names are lowercase single words**: No underscores, no mixedCaps in package names [EffectiveGo]
  ```go
  // GOOD
  package user
  package httputil
  package testhelper
  ```

- **Verify single-method interfaces use -er suffix**: Reader, Writer, Closer, Formatter [EffectiveGo]
  ```go
  // GOOD
  type Validator interface { Validate() error }
  type Renderer interface  { Render(w io.Writer) error }
  ```

- **Verify exported names have doc comments**: Every exported type, function, const, var needs a comment [CodeReview]
  ```go
  // GOOD
  // UserStore persists user records to the database.
  type UserStore struct{}

  // ErrNotFound is returned when the requested resource does not exist.
  var ErrNotFound = errors.New("not found")
  ```

### FLAG (code smells to raise)

- **Flag name stuttering**: Package name should not repeat in exported identifiers [CodeReview]
  ```go
  // BAD — callers write http.HTTPServer
  package http
  type HTTPServer struct{}
  // GOOD — callers write http.Server
  type Server struct{}
  ```

---

## Section 4: API Design

### CHECK (things that must be present/correct)

- **Verify interfaces are defined at consumer side**: Consumer declares what it needs, not producer [Mistakes#5]
  ```go
  // GOOD — handler package defines only what it needs
  package handler
  type UserFinder interface { FindByID(ctx context.Context, id int) (User, error) }
  ```

- **Verify functions accept interfaces, return structs**: Accept narrow behavior, return concrete types [EffectiveGo]
  ```go
  // GOOD
  func NewLogger(w io.Writer) *Logger {
      return &Logger{out: w}
  }
  ```

- **Verify functional options pattern for 3+ optional params**: Forward-compatible, self-documenting [Mistakes#11]
  ```go
  // GOOD
  type Option func(*Server)
  func WithPort(p int) Option  { return func(s *Server) { s.port = p } }
  func WithTLS(cfg *tls.Config) Option { return func(s *Server) { s.tls = cfg } }
  func NewServer(opts ...Option) *Server { s := &Server{}; for _, o := range opts { o(s) }; return s }
  ```

- **Verify context.Context is the first parameter**: Standard convention for all I/O-bound functions [CodeReview]
  ```go
  // GOOD
  func (s *Store) GetUser(ctx context.Context, id int) (User, error) {
      return s.db.QueryRow(ctx, query, id)
  }
  ```

- **Verify error is the last return value**: Callers expect error in the final position [CodeReview]
  ```go
  // GOOD
  func ReadConfig(path string) (Config, error) {
      data, err := os.ReadFile(path)
      return parseConfig(data), err
  }
  ```

### FLAG (code smells to raise)

- **Flag interface with 5+ methods**: Large interfaces reduce substitutability and testability [Mistakes#5]
  ```go
  // BAD — too many methods, hard to mock
  type UserService interface {
      Create(ctx context.Context, u User) error
      Update(ctx context.Context, u User) error
      Delete(ctx context.Context, id int) error
      Get(ctx context.Context, id int) (User, error)
      List(ctx context.Context) ([]User, error)
      Search(ctx context.Context, q string) ([]User, error)
  }
  ```

- **Flag exported interface with single implementation**: Premature abstraction, test with concrete type instead [Mistakes#6]
  ```go
  // BAD — only one implementation exists, interface adds indirection
  type Notifier interface { Send(msg string) error }
  type EmailNotifier struct{}
  func (e *EmailNotifier) Send(msg string) error { /* ... */ }
  ```

> Cross-ref: See `/golang:coder` Section 2 for full interface and API patterns.

---

## Section 5: Performance Pitfalls

### FLAG (code smells to raise)

- **Flag string concatenation in loops**: Use strings.Builder for O(n) instead of O(n²) [Mistakes#39]
  ```go
  // BAD — quadratic allocation
  var s string
  for _, v := range items {
      s += v.String()
  }
  // GOOD: var b strings.Builder; for _, v := range items { b.WriteString(v.String()) }
  ```

- **Flag slice append without pre-allocation**: Known-size slices should use make with capacity [Mistakes#21]
  ```go
  // BAD — repeated reallocation
  var result []User
  for _, row := range rows {
      result = append(result, toUser(row))
  }
  // GOOD: result := make([]User, 0, len(rows))
  ```

- **Flag map without size hint**: Large maps benefit from initial capacity [Mistakes#27]
  ```go
  // BAD — grows incrementally, triggers rehashing
  m := make(map[string]int)
  for _, item := range thousandItems {
      m[item.Key] = item.Value
  }
  // GOOD: m := make(map[string]int, len(thousandItems))
  ```

- **Flag unnecessary pointer for small read-only structs**: Pointers add indirection and GC pressure for no benefit [Mistakes#95]
  ```go
  // BAD — 16-byte struct does not benefit from pointer
  type Point struct{ X, Y float64 }
  func distance(a, b *Point) float64 { /* ... */ }
  // GOOD: func distance(a, b Point) float64 { /* ... */ }
  ```

- **Flag string/[]byte conversion in hot path**: Each conversion allocates a copy [Mistakes#46]
  ```go
  // BAD — allocates on every call in the loop
  for _, s := range data {
      hash.Write([]byte(s))
  }
  ```

- **Flag defer in tight loops**: Defers stack until function returns, not iteration end [Mistakes#47]
  ```go
  // BAD — all defers stack until function exits
  for _, path := range files {
      f, _ := os.Open(path)
      defer f.Close()
      process(f)
  }
  // GOOD: wrap body in func() { f, _ := os.Open(path); defer f.Close(); process(f) }()
  ```

- **Flag range loop copying large values**: Range copies the element; use index or pointer [Mistakes#30]
  ```go
  // BAD — copies 4KB struct on every iteration
  type BigStruct struct{ data [4096]byte }
  for _, item := range bigSlice {
      process(item)
  }
  // GOOD: for i := range bigSlice { process(&bigSlice[i]) }
  ```

> Cross-ref: See `/golang:tester` Section 6 for benchmarking these patterns.

---

## Section 6: Security

### CHECK (things that must be present/correct)

- **Verify user input is validated at boundary**: Validate and sanitize at the entry point, trust internally [LetsGo#11]
  ```go
  // GOOD — validate at handler, pass clean data inward
  func (h *Handler) Create(w http.ResponseWriter, r *http.Request) {
      var input CreateRequest
      if err := json.NewDecoder(r.Body).Decode(&input); err != nil { /* 400 */ }
      if err := input.Validate(); err != nil { /* 422 */ }
      h.service.Create(r.Context(), input.ToDomain())
  }
  ```

- **Verify SQL uses parameterized queries**: Never interpolate user input into query strings [LetsGo#4]
  ```go
  // GOOD — parameterized, safe from injection
  row := db.QueryRowContext(ctx, "SELECT name FROM users WHERE id = $1", userID)
  ```

- **Verify file paths are sanitized with filepath.Clean**: Prevent path traversal attacks [GoBook#1]
  ```go
  // GOOD — normalize and restrict to base directory
  clean := filepath.Clean(userInput)
  full := filepath.Join(baseDir, clean)
  if !strings.HasPrefix(full, baseDir) { return ErrInvalidPath }
  ```

- **Verify TLS for external connections**: All outbound traffic to external services must use TLS [CloudNative#8]
  ```go
  // GOOD
  srv := &http.Server{
      TLSConfig: &tls.Config{MinVersion: tls.VersionTLS12},
  }
  ```

- **Verify secrets are not in source or logs**: No hardcoded tokens, passwords, or API keys [CloudNative#8]
  ```go
  // GOOD — read from environment
  apiKey := os.Getenv("API_KEY")
  if apiKey == "" { return errors.New("API_KEY not set") }
  // Never: log.Printf("connecting with key: %s", apiKey)
  ```

---

## Section 7: Package & Project Structure

### CHECK (things that must be present/correct)

- **Verify no circular dependencies**: Package A imports B and B imports A will not compile [GoBook#10]
  ```go
  // GOOD — one-way dependency, use interfaces to invert
  // package handler imports package service
  // package service does NOT import package handler
  // handler defines its own interface for what it needs from service
  ```

- **Verify internal/ is used for private packages**: Packages under internal/ are inaccessible outside the module [GoBook#10]
  ```go
  // GOOD project layout
  // myapp/
  //   cmd/myapp/main.go
  //   internal/user/store.go    ← private to module
  //   internal/order/service.go ← private to module
  //   pkg/httputil/response.go  ← public API (if needed)
  ```

- **Verify one package per directory**: Multiple packages in one directory will not compile [EffectiveGo]
  ```go
  // GOOD
  // internal/user/store.go     → package user
  // internal/user/handler.go   → package user
  // internal/order/service.go  → package order
  ```

- **Verify cmd/ is used for entry points**: Each binary gets its own subdirectory under cmd/ [CloudNative#2]
  ```go
  // GOOD
  // cmd/api/main.go      ← HTTP server binary
  // cmd/worker/main.go   ← background worker binary
  // cmd/migrate/main.go  ← database migration binary
  ```

### FLAG (code smells to raise)

- **Flag side-effect import without comment**: Blank imports must explain why the side effect is needed [CodeReview]
  ```go
  // BAD — no explanation for blank import
  import _ "net/http/pprof"

  // GOOD — comment explains the side effect
  import _ "net/http/pprof" // Register pprof HTTP handlers
  ```

---

## Quick Reference

| Area | CHECK | FLAG |
|---|---|---|
| Errors | Checked, wrapped, lowercase | errors.New in loop, panic in lib, log+return |
| Concurrency | Exit path, sync, WaitGroup.Add order | Unbuffered no reader, copied sync, loop var |
| Naming | MixedCaps, acronyms, -er suffix | Stuttering |
| API Design | Consumer interfaces, options pattern | 5+ method interface, single impl |
| Performance | — | String concat loop, no pre-alloc, defer loop |
| Security | Input validated, parameterized SQL, TLS | — |
| Structure | No cycles, internal/, cmd/ | Uncommented blank import |
