# Cerebras Cloud Deployment

Run GLM 4.7 on Cerebras for maximum speed (1500+ tokens/sec, 20x faster than Sonnet 4.5).

## Quick Start

1. Get API key: https://cloud.cerebras.ai
2. Model ID: `zai-glm-4.7`

## Model Specs

| Spec | Value |
|------|-------|
| Parameters | ~358B total, ~32B active per token (MoE) |
| Context | 131K tokens input |
| Max output | 40K tokens via `max_completion_tokens` |
| Speed | 1500+ tokens/sec |
| Privacy | Inputs/outputs processed in memory, not persisted |

## API Parameters

### Reasoning Control

```python
# Disable reasoning for simple tasks
disable_reasoning=True

# Enable reasoning for complex tasks (default)
disable_reasoning=False
```

### Memory Control

```python
# Preserve thinking state between turns (for agent loops)
clear_thinking=False

# Reset state each turn (for one-off calls)
clear_thinking=True
```

### Sampling Defaults

```python
temperature=1
top_p=0.95
```

These defaults work well for most workloads. Adjust one at a time as needed.

## Code Example (Cerebras SDK)

```python
from cerebras.cloud.sdk import Cerebras

client = Cerebras(api_key="your-api-key")

response = client.chat.completions.create(
    model="zai-glm-4.7",
    messages=[
        {"role": "system", "content": "You are a helpful coding assistant. Always respond in English."},
        {"role": "user", "content": "Write a Python function to parse JSON safely"}
    ],
    max_completion_tokens=4096,
    temperature=1,
    top_p=0.95,
    disable_reasoning=True,  # Fast mode for simple tasks
    clear_thinking=True
)

print(response.choices[0].message.content)
```

## OpenAI SDK Compatibility

Cerebras is OpenAI-compatible. Pass GLM-specific fields via `extra_body`:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.cerebras.ai/v1",
    api_key="your-cerebras-api-key"
)

response = client.chat.completions.create(
    model="zai-glm-4.7",
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user", "content": "Hello!"}
    ],
    extra_body={
        "disable_reasoning": True,
        "clear_thinking": True
    }
)
```

## When to Use Cerebras

- Production workloads requiring speed
- Agentic workflows with many iterations
- High-volume batch processing
- When privacy matters (no data persistence)
