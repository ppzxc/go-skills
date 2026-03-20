# modern-patterns

Production-grade Go 패턴 레퍼런스 스킬입니다.

*100 Go Mistakes*, *Learning Go 2nd Edition*, *Cloud Native Go* 세 권을 기반으로 작성되었습니다.

## 포함 내용

1. Error Handling — `%w` vs `%v`, `errors.Is/As`, sentinel/custom types
2. Context — 전달 방법, timeout, value 사용 규칙
3. Generics — 타입 제약, 함수 vs 메서드
4. Concurrency — goroutine lifecycle, errgroup vs WaitGroup, mutex vs channel, leak 방지
5. Interface Design — accept interfaces / return structs, 생성자 반환 타입 규칙
6. Slice & Map — nil vs empty, 사전 할당, backing array aliasing
7. Constructor & Config — functional options, config struct
8. Testing — table-driven, t.Helper/Cleanup/Parallel, manual mock
9. Resilience — retry backoff, graceful shutdown, circuit breaker
10. Project Structure — cmd/internal/pkg, 패키지 네이밍

## 설치

```bash
/plugin marketplace add ppzxc/modern-patterns
/plugin install modern-patterns@modern-patterns
```
