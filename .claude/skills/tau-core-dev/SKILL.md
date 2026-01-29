---
name: tau-core-dev
description: >
  Use when building applications with tau-core. Covers installation,
  configuration, protocol usage (Chat, Vision, Tools, Embeddings),
  provider setup, and testing with mocks.
  Triggers: import tau-core, agent.New, Chat, Vision, Tools, Embed,
  config.json, Ollama setup, Azure setup, mock package.
---

# tau-core Usage Guide

## When This Skill Applies

- Installing and configuring tau-core
- Creating agents and executing protocols
- Setting up providers (Ollama, Azure)
- Testing code that uses tau-core

## Getting Started

### Installation

```bash
go get github.com/tailored-agentic-units/tau-core
```

### Basic Usage

```go
import (
    "github.com/tailored-agentic-units/tau-core/pkg/agent"
    "github.com/tailored-agentic-units/tau-core/pkg/config"
)

cfg, _ := config.LoadAgentConfig("config.json")
a, _ := agent.New(cfg)

response, _ := a.Chat(ctx, "Hello, world!")
fmt.Println(response.Choices[0].Message.Content)
```

## Configuration

### Config Structure

```json
{
  "name": "my-agent",
  "system_prompt": "You are a helpful assistant",
  "client": {
    "timeout": "24s",
    "retry": {
      "max_retries": 3,
      "initial_backoff": "1s"
    }
  },
  "provider": {
    "name": "ollama",
    "base_url": "http://localhost:11434"
  },
  "model": {
    "name": "llama3.2:3b",
    "capabilities": {
      "chat": {"max_tokens": 4096, "temperature": 0.7}
    }
  }
}
```

### Duration Format

Supports human-readable strings: `"24s"`, `"1m"`, `"2h"`

### Option Precedence

1. Model config provides baseline defaults
2. Runtime options override config values
3. Model name always added automatically

```go
// Config: {"temperature": 0.7}
// Runtime override:
a.Chat(ctx, "prompt", map[string]any{"temperature": 0.9})
// Result: temperature = 0.9
```

## Protocol Usage

### Chat

```go
resp, _ := a.Chat(ctx, "Explain Go interfaces")
content := resp.Choices[0].Message.Content
```

### Chat Streaming

```go
chunks, _ := a.ChatStream(ctx, "Tell me a story")
for chunk := range chunks {
    if chunk.Error != nil {
        log.Fatal(chunk.Error)
    }
    fmt.Print(chunk.Content())
}
```

### Vision

```go
// Local file or URL
images := []string{"./photo.jpg", "https://example.com/image.png"}
resp, _ := a.Vision(ctx, "Describe this image", images)
```

### Tools

```go
tools := []agent.Tool{
    {
        Name: "get_weather",
        Description: "Get weather for a location",
        Parameters: map[string]any{
            "type": "object",
            "properties": map[string]any{
                "location": map[string]any{"type": "string"},
            },
        },
    },
}
resp, _ := a.Tools(ctx, "What's the weather in Dallas?", tools)
// Handle resp.Choices[0].Message.ToolCalls
```

### Embeddings

```go
resp, _ := a.Embed(ctx, "Text to embed")
vector := resp.Data[0].Embedding // []float64
```

## Provider Setup

### Ollama

```bash
docker compose up -d  # Starts Ollama with llama3.2:3b
```

```json
{
  "provider": {
    "name": "ollama",
    "base_url": "http://localhost:11434"
  }
}
```

### Azure (API Key)

```json
{
  "provider": {
    "name": "azure",
    "base_url": "https://your-resource.openai.azure.com/openai",
    "options": {
      "deployment": "gpt-4o",
      "api_version": "2025-01-01-preview",
      "auth_type": "api_key"
    }
  }
}
```

Pass token at runtime: `-token $AZURE_API_KEY`

### Azure (Entra ID)

Same config with `"auth_type": "bearer"`, pass bearer token.

## Testing with Mocks

### Simple Mock

```go
import "github.com/tailored-agentic-units/tau-core/pkg/mock"

mockAgent := mock.NewSimpleChatAgent("test-id", "Expected response")

// Use mockAgent in your code
result, _ := mockAgent.Chat(ctx, "any prompt")
// result contains "Expected response"
```

### Helper Constructors

| Constructor | Use Case |
|-------------|----------|
| `NewSimpleChatAgent(id, response)` | Basic chat |
| `NewStreamingChatAgent(id, chunks)` | Streaming |
| `NewToolsAgent(id, toolCalls)` | Tool calling |
| `NewEmbeddingsAgent(id, vector)` | Embeddings |
| `NewFailingAgent(id, err)` | Error handling |
