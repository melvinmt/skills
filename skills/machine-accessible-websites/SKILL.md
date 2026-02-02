---
name: machine-accessible-websites
description: Implement websites accessible to both humans and AI agents. Covers floating HUMAN/MACHINE toggle, llms.txt specification, content negotiation via Accept headers, .md alternate pages, and robots.txt exclusions. Use when building agent-friendly websites, implementing llms.txt, or adding machine-readable content.
---

# Machine-Accessible Websites

Make websites accessible to both humans and AI agents by implementing the "AIs are the web's new user" pattern.

## Quick Start

Implement these 5 components:

1. **Floating toggle** - HUMAN/MACHINE switcher at page bottom
2. **Content negotiation** - Return markdown for `Accept: text/markdown`
3. **Alternate .md pages** - Markdown version at each URL + `.md`
4. **llms.txt** - AI-friendly site overview at `/llms.txt`
5. **robots.txt** - Exclude `.md` pages from search engines

---

## 1. Floating HUMAN/MACHINE Toggle

Fixed-position toggle at bottom center of each page.

**States:**
- Human (default): `◉ ʜᴜᴍᴀɴ  ○ ᴍᴀᴄʜɪɴᴇ` → shows HTML page
- Machine: `○ ʜᴜᴍᴀɴ  ◉ ᴍᴀᴄʜɪɴᴇ` → shows markdown with permalink + copy button

**Behavior:**
- State resets to HUMAN on page refresh (no localStorage/cookies)
- Machine mode shows: raw markdown, permalink to `.md` endpoint, copy button
- No close button needed (toggle navigates between modes)

For implementation details, see [references/toggle-component.md](references/toggle-component.md).

---

## 2. Content Negotiation

Return markdown when agents request it via header:

```
Accept: text/markdown
```

**Server middleware pattern:**

```javascript
function contentNegotiation(req, res, next) {
  if (req.headers.accept?.includes('text/markdown')) {
    const mdPath = getMarkdownPath(req.path);
    return res.type('text/markdown').sendFile(mdPath);
  }
  next();
}
```

All links in markdown responses MUST be formatted as: `[link text](url)`

For framework-specific examples, see [references/content-negotiation.md](references/content-negotiation.md).

---

## 3. Alternate .md Pages

Create markdown version of each page accessible via `.md` suffix:

| HTML Page | Markdown Alternate |
|-----------|-------------------|
| `/` | `/index.md` |
| `/about` | `/about.md` |
| `/docs/api` | `/docs/api.md` |

**Add alternate link tag to HTML `<head>`:**

```html
<link rel="alternate" type="text/markdown" href="/about.md" title="Markdown version">
```

---

## 4. llms.txt Implementation

Create `/llms.txt` following the [llmstxt.org](https://llmstxt.org) specification.

**Required structure:**

```markdown
# Project Name

> Brief one-line description of your project.

Key context for LLMs about your project.

## Docs

- [Getting Started](https://example.com/docs/start.md): Quick setup guide
- [API Reference](https://example.com/docs/api.md): Complete API docs

## Optional

- [Advanced Config](https://example.com/docs/advanced.md): Edge cases
```

**Machine version of landing page** should display the contents of `llms.txt`.

For templates and examples, see [references/llms-txt-examples.md](references/llms-txt-examples.md).

---

## 5. robots.txt Exclusion

Exclude `.md` pages from traditional search engine crawling:

```
User-agent: *
Disallow: /*.md$
Disallow: /llms.txt
```

This prevents duplicate content issues while keeping pages accessible to AI agents.

---

## Implementation Checklist

```
- [ ] Add floating HUMAN/MACHINE toggle component
- [ ] Implement content negotiation middleware
- [ ] Generate .md version of each page
- [ ] Add <link rel="alternate"> tags to HTML pages
- [ ] Create /llms.txt with site overview
- [ ] Configure machine mode landing page to show llms.txt
- [ ] Update robots.txt to exclude .md pages
- [ ] Test with curl -H "Accept: text/markdown"
```

---

## Testing

**Test content negotiation:**

```bash
curl -H "Accept: text/markdown" https://yoursite.com/about
```

**Test .md endpoint:**

```bash
curl https://yoursite.com/about.md
```

**Test llms.txt:**

```bash
curl https://yoursite.com/llms.txt
```
