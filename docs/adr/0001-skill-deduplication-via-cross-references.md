# ADR-0001: Skill Deduplication via Cross-References

## Status

Accepted (2026-03-26)

## Context

Go skills(coder, reviewer, tester)의 세 SKILL.md 파일 간에 약 30개의 규칙이 중복되어 있었다. 이로 인해:
- 전체 토큰의 ~25%가 중복 내용에 낭비됨
- 규칙 수정 시 여러 파일을 동시에 업데이트해야 하는 유지보수 부담
- AI 에이전트가 같은 규칙의 미묘하게 다른 버전을 보고 혼란을 겪을 가능성

중복이 발생하는 주요 영역:
| 영역 | reviewer 중복 수 | tester 중복 수 | 원본 위치 |
|------|-----------------|---------------|----------|
| Error Handling | CHECK 5개 | 0 | coder §3 |
| Concurrency | CHECK 5개 | 0 | coder §5 |
| API Design | CHECK 5개 | 0 | coder §2, §4, §6 |
| Performance | FLAG 7개 | 0 | coder §1, §10 |
| Mocks/Interfaces | 0 | 1개 | coder §2 |

## Decision

**coder를 모든 공유 규칙의 정본(canonical reference)으로 지정한다.** reviewer와 tester에서 중복 규칙을 제거하고, 해당 위치에 coder 섹션을 가리키는 cross-reference 블록으로 대체한다.

Cross-reference 형식:
```markdown
### CHECK

> These checks duplicate coder patterns. See `/golang:coder` Section 3 for full error handling rules with code examples.
>
> Verify: errors checked [MUST] · wrapped with %w [SHOULD] · consistent strategy [SHOULD]
```

**단, FLAG 규칙은 유지한다.** reviewer의 FLAG 규칙은 "코드 리뷰 시 이것을 지적하라"는 관점이고, coder의 DO 규칙은 "코드 작성 시 이렇게 하라"는 관점으로 역할이 다르다.

## Consequences

- 전체 토큰 ~24% 절감 (~20,300 → ~15,500)
- 규칙 수정 시 coder 한 곳만 변경하면 됨
- reviewer/tester 사용 시 coder 스킬도 함께 참조해야 하는 의존성 발생
- FLAG vs CHECK 구분이 명확해져 각 스킬의 역할이 분명해짐
