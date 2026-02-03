# Machine-Accessible Websites Skill

Make websites accessible to both humans and AI agents by implementing the "AIs are the web's new user" pattern.

## What's Included

- **Floating Toggle**: HUMAN/MACHINE switcher component
- **Content Negotiation**: Return markdown for `Accept: text/markdown`
- **Alternate Pages**: Markdown version at each URL + `.md`
- **llms.txt**: AI-friendly site overview following llmstxt.org spec
- **robots.txt**: Exclude `.md` pages from search engines

## Installation

```bash
npx skills add melvinmt/skills --skill machine-accessible-websites
```

<details>
<summary>bun</summary>

```bash
bunx skills add melvinmt/skills --skill machine-accessible-websites
```

</details>

<details>
<summary>pnpm</summary>

```bash
pnpm dlx skills add melvinmt/skills --skill machine-accessible-websites
```

</details>

## Implementation Checklist

| Component | Description |
|-----------|-------------|
| Floating toggle | HUMAN/MACHINE switcher at page bottom |
| Content negotiation | Middleware for `Accept: text/markdown` |
| .md pages | `/about` → `/about.md` |
| llms.txt | Site overview at `/llms.txt` |
| robots.txt | Exclude `.md` from crawlers |

## License

MIT
