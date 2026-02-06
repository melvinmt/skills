# Browser Setup for Agentic Design

Give the coding agent visual access to the website it's building.

## Cursor: Browser Tools MCP

The Browser Tools MCP plugin connects Cursor's agent to a running browser session.

### Installation

1. Install the **Browser Tools** MCP server in Cursor settings
2. Install the companion browser extension in Chrome/Chromium
3. Start your dev server (`npm run dev`)
4. Open the site in the browser with the extension active

### Capabilities

| Action | Description |
|--------|-------------|
| `screenshot` | Capture current viewport as image |
| `console_logs` | Read browser console output |
| `network_requests` | Inspect network activity |
| `click` / `type` | Interact with page elements |
| `evaluate` | Run JavaScript in the page context |

### Usage Pattern

After every code change:

```
1. Save files (triggers hot reload)
2. Wait 1-2 seconds for rebuild
3. Take screenshot: "Screenshot the current page"
4. Evaluate: "Check the layout at mobile width"
5. Fix issues and repeat
```

### Responsive Testing

Resize the viewport to test breakpoints:

```
"Screenshot the page at 375px width"
"Now screenshot at 1280px width"
"Compare the two — is the layout adapting properly?"
```

---

## Claude Code: Chrome DevTools

Claude Code can control Chrome directly via the DevTools Protocol.

### Setup

1. Launch Chrome with remote debugging:

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222
```

2. Claude Code connects via the DevTools Protocol automatically when available

### Capabilities

- Full DOM inspection
- Screenshot capture
- Console log access
- Network monitoring
- JavaScript evaluation
- Element interaction

### Usage Pattern

Same feedback loop as Cursor:

```
Make change → Save → Screenshot → Evaluate → Fix → Repeat
```

---

## Tips for Effective Visual Feedback

1. **Always screenshot after changes** — Don't assume the change looks right
2. **Screenshot at multiple widths** — Mobile (375px), tablet (768px), desktop (1280px)
3. **Check dark/light states** — If the design has a dark mode, verify both
4. **Inspect hover states** — Use the browser to simulate hover and check transitions
5. **Check real content** — Replace placeholder text before evaluating layout
6. **Full-page screenshots** — Don't just check above the fold
