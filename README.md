# go-plugins

A Claude Code plugin marketplace providing production-grade Go patterns and best practices as skills.

## Plugins

| Plugin | Version | Description |
|--------|---------|-------------|
| [go-coder](./plugins/go-coder) | 0.0.1 | Production Go patterns — types, interfaces, error handling, concurrency, service design, resilience |
| [go-reviewer](./plugins/go-reviewer) | 0.0.1 | Code review checklists — error handling, concurrency safety, naming, API design, performance, security |
| [go-tester](./plugins/go-tester) | 0.0.1 | Testing patterns — table-driven, benchmarks, fuzzing, mocks, integration tests, coverage |

## Installation

### 1. Add marketplace

```bash
/plugin marketplace add ppzxc/go-plugins
```

### 2. Install plugins

```bash
/plugin install go-coder    # writing production Go code
/plugin install go-reviewer # reviewing Go code
/plugin install go-tester   # writing Go tests
```

## Project Structure

```
go-plugins/
├── .claude-plugin/
│   └── marketplace.json        # Marketplace metadata
└── plugins/
    ├── go-coder/               # Production Go patterns
    │   ├── .claude-plugin/plugin.json
    │   └── skills/go-coder/SKILL.md
    ├── go-reviewer/            # Code review checklists
    │   ├── .claude-plugin/plugin.json
    │   └── skills/go-reviewer/SKILL.md
    └── go-tester/              # Testing patterns
        ├── .claude-plugin/plugin.json
        └── skills/go-tester/SKILL.md
```

## Author

**ppzxc** — [ppzxc.github.io](https://ppzxc.github.io)
