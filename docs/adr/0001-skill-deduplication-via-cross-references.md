# ADR-0001: Skill 중복 제거를 위한 Cross-Reference 도입

- **Status:** Accepted
- **Date:** 2026-03-26

## Context and Problem Statement

Go skills(coder, reviewer, tester)의 세 SKILL.md 파일 간에 약 30개의 규칙이 중복되어 있다. 전체 토큰의 ~25%가 낭비되고, 규칙 수정 시 여러 파일을 동시에 업데이트해야 하며, AI 에이전트가 같은 규칙의 미묘하게 다른 버전을 보고 혼란을 겪을 수 있다.

## Decision Drivers

- 토큰 효율성: AI 에이전트 컨텍스트 윈도우 절약
- 유지보수성: 규칙 변경 시 단일 수정 지점
- 역할 명확성: coder/reviewer/tester 각 스킬의 고유 관점 보존

## Considered Options

1. **Cross-Reference Only** — coder를 정본으로 지정, reviewer/tester에서 중복 제거 후 참조 블록으로 대체
2. **공통 규칙 파일 분리** — shared-rules.md를 만들어 세 스킬이 공통 참조
3. **현상 유지** — 중복을 허용하고 각 스킬을 독립적으로 유지

## Decision Outcome

선택: **Option 1 — Cross-Reference Only**

coder를 모든 공유 규칙의 정본(canonical reference)으로 지정한다. reviewer와 tester에서 중복된 CHECK 규칙을 제거하고, coder 섹션을 가리키는 cross-reference 블록으로 대체한다.

단, FLAG 규칙은 유지한다. reviewer의 FLAG("코드 리뷰 시 지적하라")와 coder의 DO("코드 작성 시 이렇게 하라")는 역할이 다르다.

### Cross-reference 형식

```markdown
### CHECK

> These checks duplicate coder patterns. See `/golang:coder` Section 3 for full error handling rules.
>
> Verify: errors checked [MUST] · wrapped with %w [SHOULD] · consistent strategy [SHOULD]
```

### 제거 대상

| 영역 | reviewer 중복 수 | tester 중복 수 | 원본 위치 |
|------|-----------------|---------------|----------|
| Error Handling | CHECK 5개 | 0 | coder §3 |
| Concurrency | CHECK 5개 | 0 | coder §5 |
| API Design | CHECK 5개 | 0 | coder §2, §4, §6 |
| Performance | FLAG 7개 | 0 | coder §1, §10 |
| Mocks/Interfaces | 0 | 1개 | coder §2 |

### Positive Consequences

- 전체 토큰 ~24% 절감 (~20,300 → ~15,500)
- 규칙 수정 시 coder 한 곳만 변경
- FLAG vs CHECK 구분이 명확해져 각 스킬의 역할이 분명해짐

### Negative Consequences

- reviewer/tester 사용 시 coder 스킬도 함께 참조해야 하는 의존성 발생
- cross-reference가 가리키는 섹션 번호 변경 시 동기화 필요
