# tau-core

A platform and model agnostic Go agent primitive library for the tailored-agentic-units ecosystem.

## Quick Reference

| Action | Command |
|--------|---------|
| Test | `go test ./tests/...` |
| Coverage | `go test ./tests/... -coverprofile=coverage.out -coverpkg=./pkg/...` |
| Validate | `go vet ./...` |
| Prompt | `go run ./cmd/prompt-agent -config <config.json> -prompt "..."` |
| Ollama | `docker compose up -d` |

## Project Structure

```
tau-core/
├── cmd/prompt-agent/    # CLI testing utility
├── pkg/                 # Library packages
│   ├── agent/           # High-level agent API
│   ├── client/          # HTTP client layer
│   ├── config/          # Configuration loading
│   ├── mock/            # Test mocks
│   ├── model/           # Model runtime type
│   ├── protocol/        # Protocol types (Chat, Vision, Tools, Embeddings)
│   ├── providers/       # Provider implementations (Ollama, Azure)
│   ├── request/         # Protocol-specific requests
│   └── response/        # Response parsing
├── scripts/             # Azure infrastructure scripts
└── tests/               # Black-box tests (mirrors pkg/)
```

## Skills

Skills load automatically based on context:

| Skill | Use When |
|-------|----------|
| go-patterns | General Go design patterns, interfaces, error handling |
| tau-core-admin | Contributing, extending providers/protocols, testing |
| tau-core-dev | Building applications with tau-core, configuration |

## Session Continuity

Plan files in `.claude/plans/` enable session continuity across machines.

### Saving Session State

When pausing work, append a context snapshot to the active plan file:

```markdown
## Context Snapshot - [YYYY-MM-DD HH:MM]

**Current State**: [Brief description of where work stands]

**Files Modified**:
- [List of files changed this session]

**Next Steps**:
- [Immediate next action]
- [Subsequent actions]

**Key Decisions**:
- [Important decisions made and rationale]

**Blockers/Questions**:
- [Any unresolved issues]
```

### Restoring Session State

When continuing work from a plan file:

1. Read the plan file to restore context
2. Review the most recent Context Snapshot
3. Resume from the documented Next Steps
4. Update the snapshot when pausing again

## Dependencies

- `github.com/google/uuid` - Agent identification (UUIDv7)

## Configuration

See `.admin/configs/` for example configurations.
