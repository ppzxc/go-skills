# ADR-0003: Go Version Annotations 도입

- **Status:** Accepted
- **Date:** 2026-03-26

## Context and Problem Statement

일부 규칙이 특정 Go 버전에서만 유효하거나 특정 버전 이후에 도입된 기능을 사용한다. 버전 정보가 없으면 Go 1.21 프로젝트에서 Go 1.22+ 전용 패턴을 적용하여 컴파일 오류가 발생하고, 이미 해결된 문제에 대한 불필요한 워크어라운드가 적용된다.

## Decision Drivers

- 프로젝트 Go 버전과의 호환성 보장
- AI 에이전트가 `go.mod` 버전을 기준으로 적합한 패턴만 제안
- 향후 Go 버전 업그레이드 시 더 이상 필요 없는 워크어라운드 식별

## Considered Options

1. **인라인 태그 `[Go X.Y+]`** — 규칙 끝에 버전 태그 추가
2. **별도 호환성 테이블** — 문서 하단에 버전별 규칙 매트릭스
3. **조건부 섹션** — Go 버전별로 섹션을 분리

## Decision Outcome

선택: **Option 1 — 인라인 태그 `[Go X.Y+]`**

버전 특정 규칙에 소스 인용 뒤 `[Go X.Y+]` 또는 `[Go < X.Y only]` 태그를 추가한다.

### 적용 대상

| 규칙 | 태그 | 위치 |
|------|------|------|
| ServeMux 패턴 `GET /users/{id}` | `[Go 1.22+]` | coder §7 |
| Loop variable capture DON'T | `[Go < 1.22 only]` | coder §5 |
| `context.WithoutCancel` | `[Go 1.21+]` | coder §6 |
| `errors.Join` | `[Go 1.20+]` | coder §3 |
| `b.Loop()` 벤치마크 | `[Go 1.24+]` | tester §6 |

### 형식

```markdown
- **[SHOULD] Use Go 1.22+ ServeMux patterns**: ... [LetsGo] [Go 1.22+]
```

### Positive Consequences

- AI 에이전트가 프로젝트 `go.mod` 버전과 대조하여 적합한 패턴만 제안 가능
- Go 버전 업그레이드 시 태그 검색으로 불필요한 워크어라운드 제거 용이

### Negative Consequences

- 새로운 Go 버전 출시 시 기존 규칙의 태그 업데이트 유지보수 필요
- 태그가 많아지면 규칙 제목이 길어질 수 있음
