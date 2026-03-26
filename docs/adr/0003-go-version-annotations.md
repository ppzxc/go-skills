# ADR-0003: Go Version Annotations

## Status

Accepted (2026-03-26)

## Context

일부 규칙이 특정 Go 버전에서만 유효하거나, 특정 버전 이후에 도입된 기능을 사용한다. 버전 정보가 없으면:
- Go 1.21을 사용하는 프로젝트에서 Go 1.22+ 전용 패턴을 적용하여 컴파일 오류 발생
- 이미 해결된 문제(예: Go 1.22의 loop variable 수정)에 대한 불필요한 워크어라운드 적용
- AI 에이전트가 프로젝트의 Go 버전을 고려하지 않고 코드 생성

## Decision

버전 특정 규칙에 `[Go X.Y+]` 또는 `[Go < X.Y only]` 태그를 소스 인용 뒤에 추가한다.

적용 대상:
| 규칙 | 태그 | 위치 |
|------|------|------|
| ServeMux 패턴 `GET /users/{id}` | `[Go 1.22+]` | coder §7 |
| Loop variable capture DON'T | `[Go < 1.22 only]` | coder §5 |
| `context.WithoutCancel` | `[Go 1.21+]` | coder §6 |
| `errors.Join` | `[Go 1.20+]` | coder §3 |
| `b.Loop()` 벤치마크 | `[Go 1.24+]` | tester §6 |

형식: `- **[SHOULD] Use Go 1.22+ ServeMux patterns**: ... [LetsGo] [Go 1.22+]`

## Consequences

- AI 에이전트가 프로젝트의 `go.mod` 버전과 대조하여 적합한 패턴만 제안 가능
- 향후 Go 버전 업데이트 시 태그를 검색하여 더 이상 필요 없는 워크어라운드를 식별 가능
- 새로운 Go 버전 출시 시 기존 규칙의 태그 업데이트 유지보수 필요
