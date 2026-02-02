# Ollama Local Deployment

Run GLM 4.7 locally for privacy and offline use.

## Requirements

- Ollama v0.15+
- ~23GB VRAM for `glm-4.7-flash`

## Model Variants

| Model | Type | VRAM | Context |
|-------|------|------|---------|
| `glm-4.7-flash` | Local | ~23GB | 64K recommended |
| `glm-4.7:cloud` | Hosted | N/A | Full context |

## Quick Start

```bash
# Install the model (~23GB VRAM required)
ollama pull glm-4.7-flash

# Or use cloud-hosted version (full context length)
ollama pull glm-4.7:cloud
```

## One-Command Setup with ollama launch

Launch coding tools with GLM 4.7 pre-configured:

```bash
# Claude Code
ollama launch claude

# OpenCode
ollama launch opencode

# Codex
ollama launch codex
```

This guides you to select models and launches the integration. No environment variables or config files needed.

### Config-Only Mode

To configure without launching immediately:

```bash
ollama launch opencode --config
```

## Context Length Configuration

Coding tools work best with full context. Update Ollama settings to at least 64000 tokens.

See Ollama's [context length documentation](https://ollama.com/docs) for configuration details.

## Supported Integrations

- Claude Code
- OpenCode
- Codex
- Droid

## Cloud Fallback

For extended coding sessions (5+ hours) or when local resources are limited:

```bash
ollama pull glm-4.7:cloud
```

The cloud variant offers:
- Full context length
- No VRAM requirements
- Generous free tier limits

See [ollama.com/pricing](https://ollama.com/pricing) for details.

## When to Use Ollama

- Privacy-sensitive workloads
- Offline development
- Local experimentation
- When you have sufficient GPU resources
