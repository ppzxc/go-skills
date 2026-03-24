# go-plugins

Claude Code 플러그인 마켓플레이스입니다. Production-grade Go 개발에 필요한 패턴과 베스트 프랙티스를 스킬로 제공합니다.

## 포함된 플러그인

| 플러그인 | 버전 | 설명 |
|---------|------|------|
| [go-coder](./plugins/go-coder) | 0.0.1 | Go 코딩 패턴 — 타입, 인터페이스, 에러 처리, 동시성, 서비스 설계, 복원력 |
| [go-reviewer](./plugins/go-reviewer) | 0.0.1 | 코드 리뷰 체크리스트 — 에러 처리, 동시성 안전, 네이밍, API 설계, 성능, 보안 |
| [go-tester](./plugins/go-tester) | 0.0.1 | 테스트 패턴 — table-driven, 벤치마크, fuzzing, mock, 통합 테스트, 커버리지 |

## 설치

### 1. 마켓플레이스 등록

```bash
/plugin marketplace add ppzxc/go-plugins
```

### 2. 플러그인 설치

```bash
/plugin install go-coder    # Go 코드 작성 시
/plugin install go-reviewer # Go 코드 리뷰 시
/plugin install go-tester   # Go 테스트 작성 시
```

## 프로젝트 구조

```
go-plugins/
├── .claude-plugin/
│   └── marketplace.json        # 마켓플레이스 메타데이터
└── plugins/
    ├── go-coder/               # Production Go 패턴
    │   ├── .claude-plugin/plugin.json
    │   └── skills/go-coder/SKILL.md
    ├── go-reviewer/            # 코드 리뷰 체크리스트
    │   ├── .claude-plugin/plugin.json
    │   └── skills/go-reviewer/SKILL.md
    └── go-tester/              # 테스트 패턴
        ├── .claude-plugin/plugin.json
        └── skills/go-tester/SKILL.md
```

## 저자

**ppzxc** — [ppzxc.github.io](https://ppzxc.github.io)
