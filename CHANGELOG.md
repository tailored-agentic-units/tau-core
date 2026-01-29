# Changelog

All notable changes to tau-core will be documented in this file.

## [v0.0.1] - 2026-01-29

Initial release of tau-core, ported from go-agents v0.3.0.

**Packages**:
- `pkg/agent` - High-level Agent interface with Chat, Vision, Tools, Embed methods
- `pkg/client` - HTTP client with retry logic and exponential backoff
- `pkg/config` - Configuration loading with human-readable durations
- `pkg/mock` - Mock implementations for testing (MockAgent, MockClient, MockProvider)
- `pkg/model` - Model runtime type bridging config to execution
- `pkg/protocol` - Protocol constants (Chat, Vision, Tools, Embeddings) and Message types
- `pkg/providers` - Provider implementations (Ollama, Azure AI Foundry)
- `pkg/request` - Protocol-specific request types
- `pkg/response` - Response parsing and streaming support

**Features**:
- Multi-protocol support: Chat, Vision, Tools, Embeddings
- Multi-provider support: Ollama, Azure (API Key and Entra ID auth)
- Streaming responses for Chat, Vision
- Configuration option merging (model defaults + runtime overrides)
- Thread-safe connection pooling
- Retry with exponential backoff and jitter
