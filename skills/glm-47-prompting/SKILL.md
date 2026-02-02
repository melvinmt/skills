---
name: glm-47-prompting
description: Best practices for prompting GLM 4.7 effectively. Use when user asks about GLM prompting, GLM 4.7 best practices, Cerebras API setup, Ollama GLM setup, optimizing GLM prompts, or working with Z.AI's open-source coding model. Covers instruction placement, reasoning control, multi-agent patterns, and deployment (cloud/local).
---

# GLM 4.7 Prompting Best Practices

GLM 4.7 is Z.AI's strongest open-source coding model (~358B params, ~32B active per token via MoE). This skill covers 10 rules for optimal prompting.

## Deployment Options

- **Cerebras cloud**: For API parameters, SDK examples, and cloud setup, see [references/cerebras.md](references/cerebras.md)
- **Ollama local**: For local installation, ollama launch, and VRAM requirements, see [references/ollama.md](references/ollama.md)

---

## Prompt Structure

### Rule 1: Front-load instructions

GLM 4.7 has a strong bias toward the beginning of the prompt (more so than other models). Place all mandatory instructions at the absolute start of your system prompt.

- Output quality degrades at extreme context lengths
- Think tags reinforce earlier instructions
- Critical directives MUST appear first

### Rule 2: Use firm, direct language

GLM 4.7 responds best to firm language that removes ambiguity. Use strong directives.

**Do:**
```
Before writing any code, you MUST first read and fully comprehend the architecture.md file. All code you generate must STRICTLY conform to the existing patterns.
```

**Don't:**
```
Please read and follow my architecture.md...
```

Keywords that work: `MUST`, `STRICTLY`, `NEVER`, `ALWAYS`, `REQUIRED`

### Rule 3: Specify a default language

GLM 4.7 is multilingual and can switch languages unexpectedly. Add to your system prompt:

```
Always respond in English.
```

Without this, the model may output reasoning traces in Chinese on the first turn.

---

## Task Design

### Rule 4: Leverage role-play

GLM 4.7 excels at maintaining roles and personas. Its internal thinking blocks mirror role prompts closely.

**Example:**
```
You are acting as a senior security engineer conducting a code review. You MUST identify all potential vulnerabilities, focusing on injection attacks, authentication bypasses, and data exposure risks.
```

Use explicit personas or create multi-agent systems with distinct roles.

### Rule 5: Break up the task

GLM 4.7 does NOT support interleaved thinking (unlike Claude/GPT). It performs a single reasoning pass per prompt before acting.

**Instead of:** "Refactor this codebase"

**Do:**
1. List all dependencies and their versions
2. Propose the new directory structure
3. Generate migration scripts for each module
4. Verify each migration compiles

This incremental approach matches GLM's execution-first tendencies.

---

## Reasoning Control

### Rule 6: Disable reasoning for simple tasks

GLM 4.7 often includes verbose internal thought blocks. For straightforward tasks, this slows down responses.

**Methods to minimize:**
- API: Set `disable_reasoning: true`
- Prompt: "Skip reasoning for straightforward tasks"
- Use structured outputs (JSON, lists) that discourage verbose reasoning
- Set `clear_thinking: true` to remove internal state between turns
- Set appropriate `max_completion_tokens` limits

### Rule 7: Enable reasoning for complex tasks

For complex problem-solving, reasoning becomes valuable.

**Methods to enhance:**
- API: Ensure `disable_reasoning: false` (or omit)
- Prompt: "Think step by step before answering"
- Use chain-of-thought examples showing the reasoning process

### Rule 8: Use clear_thinking to control memory

Controls internal thinking state across calls:

| Setting | Use Case |
|---------|----------|
| `clear_thinking: false` | Agent loops, multi-step plans, coding sessions |
| `clear_thinking: true` | One-off calls, batch jobs, when seeing drift |

---

## Multi-Agent Patterns

### Rule 9: Use critic agents

Employ specialized sub-agents to review outputs before advancing. Decouple generation from validation.

**Critic types:**

| Agent | Focus |
|-------|-------|
| Code Review | SOLID/DRY/YAGNI principles, maintainability |
| QA Expert | User flows, edge cases, integration points |
| Security Review | Vulnerabilities, unsafe patterns, compliance |
| Performance Audit | Bottlenecks, inefficient algorithms, resource leaks |

Each critic gets a focused persona (leverages Rule 4).

### Rule 10: Pair with frontier models

GLM 4.7 may fall short on the toughest 10% of use cases. Three patterns:

1. **Router**: Route simple tasks to GLM 4.7, fall back to slower models for complex queries
2. **Backbone**: Use GLM 4.7 as fast backbone, loop in frontier models only when needed
3. **Plan-execute**: Use Claude/GPT to create plan, execute rapidly with GLM 4.7

Leverage GLM's 17x speed advantage for the majority of tasks.
