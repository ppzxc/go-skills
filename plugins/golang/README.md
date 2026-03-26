# golang

A production-grade Go patterns plugin for Claude Code, providing three skills for writing, reviewing, and testing Go code.

Based on three canonical books:

- *100 Go Mistakes and How to Avoid Them* (Teiva Harsanyi)
- *Learning Go, 2nd Edition* (Jon Bodner)
- *Cloud Native Go* (Matthew Titmus)

## Skills

| Skill | Command | Description |
|-------|---------|-------------|
| coder | `/golang:coder` | Types, interfaces, error handling, concurrency, service design, resilience |
| reviewer | `/golang:reviewer` | Error handling, concurrency safety, naming, API design, performance, security |
| tester | `/golang:tester` | Table-driven tests, benchmarks, fuzzing, mocks, integration tests, coverage |

## Installation

```bash
/plugin marketplace add ppzxc/golang-plugins
/plugin install golang
```

## Usage

Invoke a skill by its fully-qualified slash command:

```
/golang:coder    # writing or refactoring Go production code
/golang:reviewer # reviewing Go code in pull requests
/golang:tester   # writing or improving Go tests
```

Skills also activate automatically based on context when working with Go code.

## Author

**ppzxc** — [ppzxc.github.io](https://ppzxc.github.io)
