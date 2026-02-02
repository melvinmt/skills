# Skills

A collection of Claude skills for specialized workflows.

## Available Skills

| Skill | Description |
|-------|-------------|
| [glm-47-prompting](skills/glm-47-prompting/) | Best practices for prompting GLM 4.7 |

## Installation

```bash
npx skills add melvinmt/skills
```

<details>
<summary>Other package managers</summary>

```bash
# bun
bunx skills add melvinmt/skills

# pnpm
pnpm dlx skills add melvinmt/skills
```

</details>

<details>
<summary>Claude Code plugin</summary>

```
/plugin install <skill-name>
```

</details>

<details>
<summary>Manual installation</summary>

Copy the skill folder to your Claude skills directory.

</details>

## Adding New Skills

Each skill is a self-contained folder under `skills/`. To add a new skill:

1. Create a new folder: `skills/<skill-name>/`
2. Add a `SKILL.md` with YAML frontmatter (name, description)
3. Optionally add `references/`, `scripts/`, or `assets/` folders
4. Add `marketplace.json` for one-command installation

## License

MIT
