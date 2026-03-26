---
name: tester
description: "Use when writing, debugging, or improving Go tests — table-driven tests, benchmarks, fuzzing, mocks, integration tests, race detection, and coverage."
user_invocable: true
---

# Go Tester

Rule-based reference for writing Go tests. Every rule cites a canonical source.

Sources: [GoBook] The Go Programming Language (Donovan & Kernighan), [Mistakes] 100 Go Mistakes and How to Avoid Them (Harsanyi), [Concurrency] Concurrency in Go (Cox-Buday), [LearningGo] Learning Go, 2nd Edition (Bodner), [LearnWithTests] Learn Go with Tests (Chris James), [CloudNative] Cloud Native Go (Titmus), [EffectiveGo] Effective Go (Go Team)

> See also: `/golang:coder` for interface design patterns. `/golang:reviewer` for code review checklists.

---

## Section 1: Test Structure

### DO

- **Place test files next to source**: Keep `foo_test.go` beside `foo.go` in the same directory [GoBook#11]
  ```go
  // mypackage/
  //   user.go
  //   user_test.go    <- same directory
  //   handler.go
  //   handler_test.go <- same directory
  ```

- **Name tests TestFunctionName_Scenario**: Use t.Run for subtests with descriptive names [LearningGo#13]
  ```go
  func TestParseConfig_EmptyInput(t *testing.T) {
      _, err := ParseConfig("")
      require.Error(t, err)
  }
  ```

- **Use TestMain for package-level setup/teardown**: Run shared setup once before all tests [GoBook#11]
  ```go
  func TestMain(m *testing.M) {
      db := setupTestDB()
      code := m.Run()
      db.Close()
      os.Exit(code)
  }
  ```

- **Use build tags for integration tests**: Separate slow tests from fast unit tests [CloudNative#12]
  ```go
  //go:build integration

  package myservice_test
  ```

- **Use t.Parallel for independent tests**: Speed up test suites by running independent tests concurrently [Mistakes#82]
  ```go
  func TestUserService_Create(t *testing.T) {
      t.Parallel()
      // test logic with no shared mutable state
  }
  ```

- **Use _test package for black-box testing**: Test the public API, not internals [LearningGo#13]
  ```go
  package mypackage_test // black-box: only public API visible

  import "myproject/mypackage"
  ```

### DON'T

- **Don't test private functions directly**: Use black-box tests via the `_test` package to validate behavior, not implementation [LearningGo#13]
  ```go
  // BAD: package mypackage (white-box test of internal helper)
  func Test_parseInternal(t *testing.T) {
      result := parseInternal("data") // unexported
  }
  ```

- **Don't put test logic in TestMain**: TestMain is for setup/teardown only; keep assertions in test functions [GoBook#11]
  ```go
  // BAD: assertions inside TestMain
  func TestMain(m *testing.M) {
      result := compute()
      if result != 42 { log.Fatal("wrong") } // don't do this
      os.Exit(m.Run())
  }
  ```

---

## Section 2: Table-Driven Tests

### DO

- **Use name field for clear subtest output**: Every table entry gets a descriptive name for `go test -v` output [LearnWithTests#4]
  ```go
  tests := []struct {
      name  string
      input string
      want  int
  }{
      {name: "empty string", input: "", want: 0},
      {name: "single word", input: "hello", want: 5},
  }
  ```

- **Cover happy path, edge cases, and error cases**: Include at least one success, one boundary, and one error case [Mistakes#83]
  ```go
  {name: "valid email", input: "a@b.com", wantErr: false},
  {name: "empty input", input: "", wantErr: true},
  {name: "missing @", input: "invalid", wantErr: true},
  ```

- **Keep test cases declarative**: Define input and expected output; keep assertion logic outside the struct [LearningGo#13]
  ```go
  tests := []struct {
      name    string
      input   int
      want    string
      wantErr bool
  }{ /* cases */ }
  ```

### DON'T

- **Don't put assertions inside test case struct**: Assertions belong in the test loop, not in case definitions [LearningGo#13]
  ```go
  // BAD: function field for assertion
  tests := []struct {
      name   string
      assert func(t *testing.T, result int)
  }{ /* mixes data with logic */ }
  ```

- **Don't share mutable state between subtests**: Each subtest must be independent, especially with t.Parallel [Concurrency#4]
  ```go
  // BAD: shared counter across subtests
  count := 0
  for _, tt := range tests {
      t.Run(tt.name, func(t *testing.T) {
          count++ // data race with t.Parallel()
      })
  }
  ```

- **Don't skip error case tests**: Error paths contain critical validation logic; always test them [Mistakes#83]
  ```go
  // BAD: only testing the happy path
  tests := []struct{ name, input, want string }{
      {"valid", "good-input", "expected"},
      // missing: empty input, invalid format, boundary values
  }
  ```

---

## Section 3: Assertions & Helpers

### DO

- **Mark helpers with t.Helper**: Report failures at the caller's line, not inside the helper [Mistakes#82]
  ```go
  func assertNoError(t *testing.T, err error) {
      t.Helper()
      if err != nil {
          t.Fatalf("unexpected error: %v", err)
      }
  }
  ```

- **Use t.Cleanup for teardown**: Always runs even after t.Fatal, unlike defer [Mistakes#84]
  ```go
  func setupDB(t *testing.T) *sql.DB {
      t.Helper()
      db, err := sql.Open("sqlite3", ":memory:")
      require.NoError(t, err)
      t.Cleanup(func() { db.Close() })
      return db
  }
  ```

- **Use require for fatal and assert for non-fatal**: Stop test on precondition failure, continue on value checks [LearnWithTests#4]
  ```go
  require.NoError(t, err)           // fatal: stop if setup fails
  assert.Equal(t, expected, actual)  // non-fatal: report and continue
  ```

- **Use cmp.Diff for struct comparison**: Get readable diffs instead of opaque inequality messages [LearningGo#13]
  ```go
  if diff := cmp.Diff(want, got); diff != "" {
      t.Errorf("mismatch (-want +got):\n%s", diff)
  }
  ```

### DON'T

- **Don't use t.Fatal in goroutines**: t.Fatal calls runtime.Goexit, which only exits the current goroutine [Mistakes#82]
  ```go
  // BAD: t.Fatal in a spawned goroutine
  go func() {
      if err := doWork(); err != nil {
          t.Fatal(err) // panics or hangs
      }
  }()
  ```

- **Don't assert on string representations**: String formats change; compare structured values instead [LearningGo#13]
  ```go
  // BAD: brittle string comparison
  assert.Equal(t, "User{Name:alice}", fmt.Sprint(user))
  // GOOD: compare fields directly
  assert.Equal(t, "alice", user.Name)
  ```

---

## Section 4: Mocks & Test Doubles

### DO

- **Define narrow interfaces at test site**: Accept only the methods the code under test needs [LearnWithTests#9]
  ```go
  type UserFinder interface {
      FindByID(ctx context.Context, id string) (*User, error)
  }
  ```

- **Use httptest for HTTP testing**: NewRecorder for unit tests, NewServer for integration tests [GoBook#11]
  ```go
  req := httptest.NewRequest(http.MethodGet, "/users/1", nil)
  w := httptest.NewRecorder()
  handler.ServeHTTP(w, req)
  assert.Equal(t, http.StatusOK, w.Code)
  ```

- **Prefer fakes over mock frameworks**: Function-field structs are simpler and more readable for small interfaces [LearnWithTests#9]
  ```go
  type FakeStore struct {
      FindFn func(ctx context.Context, id string) (*User, error)
  }
  func (f *FakeStore) FindByID(ctx context.Context, id string) (*User, error) {
      return f.FindFn(ctx, id)
  }
  ```

- **Use httptest.NewServer for external service fakes**: Simulate third-party APIs in integration tests [GoBook#11]
  ```go
  srv := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
      w.WriteHeader(http.StatusOK)
      fmt.Fprintln(w, `{"status":"ok"}`)
  }))
  t.Cleanup(srv.Close)
  ```

### DON'T

- **Don't mock types you don't own**: Wrap third-party types behind your own interface instead [LearnWithTests#9]
  ```go
  // BAD: mocking sql.DB directly
  // GOOD: define your own Store interface, mock that
  type Store interface {
      GetUser(ctx context.Context, id string) (*User, error)
  }
  ```

- **Don't verify mock call counts by default**: Assert on outputs, not on how many times a mock was called [LearnWithTests#9]
  ```go
  // BAD: fragile coupling to implementation
  assert.Equal(t, 1, mock.FindCallCount)
  // GOOD: assert on the result
  assert.Equal(t, "alice", user.Name)
  ```

- **Don't generate mocks for simple interfaces**: Code-generated mocks add complexity; use manual fakes for 1-2 method interfaces [LearnWithTests#9]
  ```go
  // BAD: //go:generate mockgen for a single-method interface
  // GOOD: hand-write a 5-line fake struct
  type FakeNotifier struct{ Sent []string }
  func (f *FakeNotifier) Send(msg string) { f.Sent = append(f.Sent, msg) }
  ```

> Cross-ref: See `/golang:coder` Section 2 for interface design patterns.

---

## Section 5: Integration Tests

### DO

- **Separate with build tags**: Keep integration tests out of normal `go test` runs [CloudNative#12]
  ```go
  //go:build integration

  package myservice_test
  // Run: go test -tags integration ./...
  ```

- **Use testcontainers-go for dependencies**: Spin up real databases and services in containers [CloudNative#12]
  ```go
  pgContainer, err := postgres.Run(ctx,
      "postgres:16-alpine",
      postgres.WithDatabase("testdb"),
  )
  require.NoError(t, err)
  t.Cleanup(func() { pgContainer.Terminate(ctx) })
  ```

- **Use testing.Short for fast mode**: Skip slow tests when running with -short flag [GoBook#11]
  ```go
  func TestSlowOperation(t *testing.T) {
      if testing.Short() {
          t.Skip("skipping slow test in short mode")
      }
      // slow test logic
  }
  ```

### DON'T

- **Don't share database state between tests**: Each test must set up and tear down its own data [CloudNative#12]
  ```go
  // BAD: tests depend on rows inserted by other tests
  // GOOD: each test creates its own data
  func TestCreateUser(t *testing.T) {
      db := freshDB(t) // isolated schema or transaction
      // test logic
  }
  ```

- **Don't use file-based separation for integration tests**: Build tags are the idiomatic way; separate directories fragment packages [LearningGo#13]
  ```go
  // BAD: integration_tests/user_test.go (separate directory)
  // GOOD: user_integration_test.go with //go:build integration
  //go:build integration
  package myservice_test
  ```

---

## Section 6: Benchmarks

### DO

- **Name BenchmarkFunctionName_Scenario**: Follow the same naming pattern as tests [GoBook#11]
  ```go
  func BenchmarkParseJSON_SmallPayload(b *testing.B) {
      data := []byte(`{"name":"alice"}`)
      for b.Loop() { ParseJSON(data) }
  }
  ```

- **Call b.ResetTimer after setup**: Exclude setup cost from measurement [Mistakes#89]
  ```go
  func BenchmarkProcess(b *testing.B) {
      data := loadLargeDataset() // expensive setup
      b.ResetTimer()             // reset after setup
      for b.Loop() { Process(data) }
  }
  ```

- **Call b.ReportAllocs for allocation tracking**: Surface allocation counts per operation [Mistakes#89]
  ```go
  func BenchmarkEncode(b *testing.B) {
      b.ReportAllocs()
      for b.Loop() { Encode(testData) }
  }
  ```

- **Prevent compiler from eliminating results**: Assign results to a package-level sink variable [GoBook#11]
  ```go
  var sink any

  func BenchmarkCompute(b *testing.B) {
      var result int
      for b.Loop() { result = Compute(42) }
      sink = result // prevents dead code elimination
  }
  ```

### DON'T

- **Don't benchmark without benchstat**: Raw numbers are meaningless without statistical comparison [Mistakes#89]
  ```bash
  # GOOD: collect multiple runs, compare statistically
  go test -bench=. -count=5 ./... > old.txt
  # make changes
  go test -bench=. -count=5 ./... > new.txt
  benchstat old.txt new.txt
  ```

- **Don't benchmark in -race mode**: Race detector adds 5-10x overhead, distorting benchmark results [Concurrency#6]
  ```bash
  # BAD: go test -bench=. -race ./...
  # GOOD: run benchmarks and race detection separately
  go test -race ./...         # correctness
  go test -bench=. ./...      # performance
  ```

> Cross-ref: See `/golang:reviewer` Section 5 for performance review checklist.

---

## Section 7: Fuzzing

### DO

- **Target parsing and validation functions**: Fuzz functions that accept arbitrary input: parsers, deserializers, validators [LearningGo#13]
  ```go
  func FuzzParseJSON(f *testing.F) {
      f.Add(`{"key":"value"}`)
      f.Fuzz(func(t *testing.T, data string) {
          ParseJSON([]byte(data)) // must not panic
      })
  }
  ```

- **Add seed corpus for edge cases**: Provide known interesting inputs to guide the fuzzer [LearningGo#13]
  ```go
  f.Add(`{}`)
  f.Add(`{"name":""}`)
  f.Add(`null`)
  f.Add(strings.Repeat("a", 10000)) // large input
  ```

- **Add crash inputs to testdata/corpus**: Commit discovered failures as regression tests [LearningGo#13]
  ```bash
  # Crash inputs are saved automatically to:
  # testdata/fuzz/FuzzParseJSON/<hash>
  # Commit these files to prevent regressions
  git add testdata/fuzz/
  ```

### DON'T

- **Don't fuzz functions with external dependencies**: Fuzz targets must be fast, deterministic, and side-effect-free [LearningGo#13]
  ```go
  // BAD: fuzzing a function that hits a database
  f.Fuzz(func(t *testing.T, query string) {
      db.Query(query) // external dependency, slow, non-deterministic
  })
  ```

- **Don't run fuzzing in CI without time limit**: Fuzzing runs indefinitely by default; set -fuzztime for CI [LearningGo#13]
  ```bash
  # BAD: go test -fuzz=. ./...              (runs forever)
  # GOOD: go test -fuzz=. -fuzztime=30s ./... (bounded)
  # GOOD: go test -run=Fuzz ./...           (seed corpus only)
  ```

---

## Section 8: Race Detection & Coverage

### DO

- **Run -race in CI always**: Data races are undefined behavior; detect them early [Concurrency#6]
  ```bash
  go test -race ./...
  # Add to CI pipeline as a required check
  ```

- **Use -count=1 to bypass test cache**: Ensure tests actually run instead of using cached results [Mistakes#84]
  ```bash
  go test -count=1 -race ./...
  # Forces re-execution even if code hasn't changed
  ```

- **Generate coverage profiles**: Track test coverage and identify untested code paths [GoBook#11]
  ```bash
  go test -coverprofile=coverage.out ./...
  go tool cover -html=coverage.out -o coverage.html
  go tool cover -func=coverage.out | tail -1
  ```

### DON'T

- **Don't chase 100% coverage**: Aim for meaningful coverage of critical paths; 100% creates brittle tests [Mistakes#83]
  ```go
  // BAD: testing trivial getters just for coverage
  func TestUser_Name(t *testing.T) {
      u := User{Name: "alice"}
      assert.Equal(t, "alice", u.Name) // adds coverage, not value
  }
  ```

- **Don't ignore race detector findings**: Every race report is a real bug; fix it immediately [Concurrency#6]
  ```bash
  # Race detector output is never a false positive.
  # WARNING: DATA RACE means the code has undefined behavior.
  # Fix the race, do not suppress the detector.
  ```

> Cross-ref: See `/golang:reviewer` Section 2 for concurrency safety review checklist.

---
