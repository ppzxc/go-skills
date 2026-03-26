# ADR-0002: RFC 2119 기반 Severity Tags 도입

- **Status:** Accepted
- **Date:** 2026-03-26

## Context and Problem Statement

약 100개의 Go 규칙에 우선순위 구분이 없어서, AI 에이전트가 트레이드오프 결정 시 모든 규칙을 동일 중요도로 취급한다. 코드 리뷰 시 치명적 결함과 스타일 권장사항이 구분되지 않고, 새로운 기여자가 어떤 규칙을 먼저 따라야 하는지 판단하기 어렵다.

## Decision Drivers

- AI 에이전트의 판단 근거 제공: 버그 vs 스타일 구분
- 코드 리뷰 효율성: 심각도별 필터링
- 업계 표준 용어 사용: 외부 개발자에게도 의미 명확

## Considered Options

1. **RFC 2119 (MUST/SHOULD/MAY)** — 업계 표준 3단계 분류
2. **숫자 우선순위 (P0/P1/P2)** — 인시던트 관리식 분류
3. **커스텀 태그 (Critical/Recommended/Optional)** — 자체 용어

## Decision Outcome

선택: **Option 1 — RFC 2119 (MUST/SHOULD/MAY)**

모든 규칙의 bold 제목에 severity tag를 추가한다.

### 분류 기준

| Tag | 기준 | 예시 |
|-----|------|------|
| `[MUST]` | 위반 시 버그, 보안 취약점, 데이터 손실, 정의되지 않은 동작 | error 검사, SQL 파라미터화, race condition 방지 |
| `[SHOULD]` | Go 관용적 코드, 유지보수성/가독성 크게 향상 | error wrapping, consumer-side interface, 구조적 로깅 |
| `[MAY]` | 상황에 따라 유용하나 보편적 적용 시 과도 | sync.Pool, generics, functional options |

### 형식

```markdown
- **[MUST] Always check errors**: Never discard... [Mistakes#48]
```

WHEN 항목은 severity tag 대상에서 제외한다 (조건부 가이드라인).

### Positive Consequences

- AI 에이전트가 `[MUST]` 위반을 우선 수정하는 판단 근거 확보
- 코드 리뷰 시 심각도별 필터링 가능
- RFC 2119 표준 용어로 외부 개발자에게도 의미 명확

### Negative Consequences

- 모든 규칙에 일관된 태그가 필요하므로 신규 규칙 추가 시 분류 의무 발생
- 경계 사례에서 MUST vs SHOULD 판단이 주관적일 수 있음
