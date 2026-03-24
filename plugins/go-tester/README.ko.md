# go-tester

Go 테스트 패턴 레퍼런스 스킬입니다.

## 포함 내용

1. **Table-Driven Tests** — t.Run, 병렬 서브테스트
2. **Test Helpers** — t.Helper, t.Cleanup, t.Parallel
3. **Mocks** — 인터페이스 기반 수동 mock
4. **Data Race Detection** — go test -race
5. **Benchmarks** — sub-benchmark, benchstat
6. **Fuzzing** — testing.F, seed corpus (Go 1.18+)
7. **Integration Testing** — testcontainers-go, build 태그
8. **httptest** — 핸들러/미들웨어 테스트
9. **Golden File Testing** — testdata/ 컨벤션
10. **Test Lifecycle** — TestMain, testing.TB 헬퍼
11. **Coverage & Profiling** — -cover, -cpuprofile

## 설치

```bash
/plugin marketplace add ppzxc/go-plugins
/plugin install go-tester
```

## 사용 방법

```
/go-tester
```

Go 테스트 작성, 디버깅, 리뷰 시 자동으로 활성화됩니다.
