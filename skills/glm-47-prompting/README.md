# GLM 4.7 Prompting Skill

Best practices for prompting GLM 4.7, Z.AI's strongest open-source coding model (~358B params, ~32B active per token via MoE).

## What's Included

- **Prompt Structure**: Front-loading instructions, firm language, language defaults
- **Task Design**: Role-play patterns, task breakdown strategies
- **Reasoning Control**: When to enable/disable reasoning, clear_thinking usage
- **Multi-Agent Patterns**: Critic agents, pairing with frontier models
- **Deployment**: Cerebras cloud and Ollama local setup

## Installation

```bash
npx skills add melvinmt/skills --skill glm-47-prompting
```

<details>
<summary>bun</summary>

```bash
bunx skills add melvinmt/skills --skill glm-47-prompting
```

</details>

<details>
<summary>pnpm</summary>

```bash
pnpm dlx skills add melvinmt/skills --skill glm-47-prompting
```

</details>

## Key Rules

| Rule | Description |
|------|-------------|
| Front-load instructions | Place critical directives at the start |
| Use firm language | MUST, STRICTLY, NEVER, ALWAYS |
| Specify default language | Prevent unexpected language switching |
| Leverage role-play | GLM excels at maintaining personas |
| Break up tasks | One reasoning pass per prompt |

## License

MIT
