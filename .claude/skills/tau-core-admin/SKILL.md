---
name: tau-core-admin
description: >
  REQUIRED for contributing to tau-core. Use when extending providers,
  adding protocols, modifying architecture, or writing tests.
  Triggers: new provider, new protocol, Provider interface, BaseProvider,
  testing, coverage, tests/ directory, pkg/ internals, architecture.
---

# tau-core Contributor Guide

## When This Skill Applies

- Adding new LLM providers
- Implementing new protocols
- Modifying core architecture
- Writing or modifying tests
- Understanding package internals

## Architecture Overview

### Protocol-Centric Design

Protocols are the primary abstraction. No separate capability layer.

```
Agent.Chat(prompt)
  → Client.ExecuteProtocol(ChatRequest)
    → Provider.PrepareRequest()
      → HTTP Request with Retry
    → Provider.ProcessResponse()
  → ChatResponse
```

### Package Dependency Hierarchy (low → high)

1. `pkg/config` - Configuration loading
2. `pkg/protocol` - Protocol constants, Message types
3. `pkg/response` - Response parsing
4. `pkg/providers` - Provider interface and implementations
5. `pkg/model` - Runtime model type
6. `pkg/request` - Protocol-specific requests
7. `pkg/client` - HTTP orchestration
8. `pkg/agent` - High-level API
9. `pkg/mock` - Test mocks

### Package Responsibilities

| Package | Responsibility |
|---------|---------------|
| `protocol` | Protocol constants (Chat, Vision, Tools, Embeddings), Message struct |
| `response` | Response types, parsers, streaming chunks |
| `providers` | Provider interface, BaseProvider, Ollama/Azure implementations |
| `model` | Model runtime type, config-to-domain conversion |
| `request` | ChatRequest, VisionRequest, ToolsRequest, EmbeddingsRequest |
| `client` | HTTP client, retry logic, protocol execution |
| `agent` | Agent interface, message initialization, option merging |
| `mock` | MockAgent, MockClient, MockProvider for testing |

## Extension Patterns

### Adding a New Provider

1. Create `pkg/providers/<name>.go`
2. Embed `BaseProvider` for common functionality
3. Implement `Provider` interface:
   - `GetEndpoint(protocol Protocol) (string, error)`
   - `PrepareRequest(ctx, request) (*Request, error)`
   - `ProcessResponse(resp, protocol) (any, error)`
   - `SetHeaders(req *http.Request)`
4. Register in `pkg/providers/registry.go`
5. Add tests in `tests/providers/<name>_test.go`

### Adding a New Protocol

1. Add constant to `pkg/protocol/protocol.go`
2. Update `IsValid()` and `SupportsStreaming()`
3. Create request type in `pkg/request/<protocol>.go`
4. Create response type in `pkg/response/<protocol>.go`
5. Add `Parse<Protocol>Response()` function
6. Update providers to support new endpoint
7. Add agent method if needed

## Testing Strategy

### Test Organization

Tests in `tests/` directory mirror `pkg/` structure:
- `tests/config/` → tests `pkg/config/`
- `tests/providers/` → tests `pkg/providers/`

### Black-Box Testing

All tests use `package <name>_test`:

```go
package config_test

import (
    "testing"
    "github.com/tailored-agentic-units/tau-core/pkg/config"
)
```

### Table-Driven Tests

```go
tests := []struct {
    name     string
    input    string
    expected string
}{
    {name: "valid", input: "test", expected: "result"},
}

for _, tt := range tests {
    t.Run(tt.name, func(t *testing.T) {
        // test logic
    })
}
```

### HTTP Mocking

```go
server := httptest.NewServer(http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    json.NewEncoder(w).Encode(mockResponse)
}))
defer server.Close()
// Use server.URL in provider config
```

### Coverage Philosophy

Focus on ensuring all testable public infrastructure is covered:
- Public interfaces and their implementations
- Exported functions and methods
- Request/response parsing
- Configuration loading
- Protocol routing

```bash
go test ./tests/... -coverprofile=coverage.out -coverpkg=./pkg/...
go tool cover -func=coverage.out
```
