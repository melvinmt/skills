# llms.txt Examples

Templates and examples for implementing the [llmstxt.org](https://llmstxt.org) specification.

---

## Basic Template

```markdown
# Project Name

> Brief one-line description of what this project/site does.

Essential context that LLMs need to understand this project.

## Docs

- [Getting Started](https://example.com/docs/start.md): Quick setup guide
- [API Reference](https://example.com/docs/api.md): Complete API documentation
- [Configuration](https://example.com/docs/config.md): Configuration options

## Examples

- [Basic Usage](https://example.com/examples/basic.md): Simple usage examples
- [Advanced Patterns](https://example.com/examples/advanced.md): Complex use cases

## Optional

- [Changelog](https://example.com/changelog.md): Version history
- [Contributing](https://example.com/contributing.md): How to contribute
```

---

## SaaS Product Example

```markdown
# Acme Analytics

> Real-time analytics platform for web and mobile applications.

Acme Analytics provides event tracking, user segmentation, and funnel analysis. 
The API uses REST with JSON payloads. Authentication requires an API key in the 
X-Acme-Key header.

## Docs

- [Quick Start](https://acme.io/docs/quickstart.md): Get started in 5 minutes
- [API Reference](https://acme.io/docs/api.md): Complete REST API documentation
- [SDKs](https://acme.io/docs/sdks.md): JavaScript, Python, and Ruby SDKs
- [Events Schema](https://acme.io/docs/events.md): Event naming and properties

## Guides

- [User Tracking](https://acme.io/guides/tracking.md): Track user behavior
- [Funnels](https://acme.io/guides/funnels.md): Build conversion funnels
- [Segments](https://acme.io/guides/segments.md): Create user segments

## Optional

- [Pricing](https://acme.io/pricing.md): Plans and pricing details
- [Status](https://status.acme.io/llms.txt): API status and uptime
```

---

## Open Source Library Example

```markdown
# FastWidget

> A lightweight JavaScript widget library for building interactive UI components.

FastWidget is framework-agnostic and works with vanilla JS, React, Vue, and Svelte.
Components are tree-shakeable and total bundle size is under 5KB gzipped.

Key concepts:
- Widgets are defined as plain objects with render() methods
- State management uses signals (reactive primitives)
- No virtual DOM - direct DOM manipulation for performance

## Docs

- [Installation](https://fastwidget.dev/docs/install.md): npm/yarn/pnpm setup
- [Quick Start](https://fastwidget.dev/docs/quickstart.md): Build your first widget
- [API Reference](https://fastwidget.dev/docs/api.md): Complete API documentation
- [Signals](https://fastwidget.dev/docs/signals.md): Reactive state management

## Examples

- [Counter](https://fastwidget.dev/examples/counter.md): Basic counter widget
- [Todo List](https://fastwidget.dev/examples/todo.md): CRUD operations
- [Data Table](https://fastwidget.dev/examples/table.md): Sortable, filterable table

## Optional

- [Migration from v1](https://fastwidget.dev/docs/migration.md): Breaking changes
- [Comparison](https://fastwidget.dev/docs/comparison.md): vs React, Vue, etc.
```

---

## E-commerce Example

```markdown
# ShopCo

> Online marketplace for handcrafted goods and artisan products.

ShopCo connects artisans with customers. Products span categories including 
jewelry, home decor, clothing, and art. All items are handmade by independent 
creators.

## Info

- [About Us](https://shopco.com/about.md): Our mission and story
- [How It Works](https://shopco.com/how-it-works.md): Buying and selling process
- [Categories](https://shopco.com/categories.md): Product category overview

## Policies

- [Shipping](https://shopco.com/shipping.md): Shipping options and times
- [Returns](https://shopco.com/returns.md): Return policy and process
- [Seller Guidelines](https://shopco.com/seller-guidelines.md): For sellers

## Optional

- [FAQ](https://shopco.com/faq.md): Common questions
- [Contact](https://shopco.com/contact.md): Support information
```

---

## Documentation Site Example

```markdown
# Kubernetes Docs

> Official documentation for the Kubernetes container orchestration platform.

Kubernetes (K8s) is an open-source system for automating deployment, scaling,
and management of containerized applications.

## Getting Started

- [Overview](https://kubernetes.io/docs/overview.md): What is Kubernetes
- [Install](https://kubernetes.io/docs/install.md): Installation methods
- [First App](https://kubernetes.io/docs/first-app.md): Deploy your first app

## Concepts

- [Pods](https://kubernetes.io/docs/pods.md): The basic unit of deployment
- [Services](https://kubernetes.io/docs/services.md): Networking and load balancing
- [Deployments](https://kubernetes.io/docs/deployments.md): Managing replicas

## Reference

- [kubectl](https://kubernetes.io/docs/kubectl.md): CLI reference
- [API](https://kubernetes.io/docs/api.md): REST API reference

## Optional

- [Troubleshooting](https://kubernetes.io/docs/troubleshooting.md): Common issues
- [Release Notes](https://kubernetes.io/docs/releases.md): Version history
```

---

## Machine Landing Page

When displaying llms.txt as the machine version of your landing page:

```javascript
// Fetch and display llms.txt content for machine mode landing page
async function getMachineLandingContent() {
  const response = await fetch('/llms.txt');
  return await response.text();
}
```

The landing page in machine mode should display the raw llms.txt content, providing AI agents with a structured overview of your entire site.

---

## Best Practices

1. **Keep it concise** - LLMs have limited context windows
2. **Use .md URLs** - Point to markdown versions of pages
3. **Include key context** - What LLMs need to understand your project
4. **Organize by purpose** - Group links logically (Docs, Examples, etc.)
5. **Use Optional section** - For less critical content that can be skipped
6. **Update regularly** - Keep links current as content changes

---

## Generating llms.txt

For static site generators, automate llms.txt generation:

```javascript
// build-llms-txt.js
const fs = require('fs');
const path = require('path');

const config = {
  name: 'My Project',
  description: 'Brief description here.',
  context: 'Additional context for LLMs.',
  sections: {
    Docs: [
      { title: 'Getting Started', path: '/docs/start.md', desc: 'Quick setup' },
      { title: 'API Reference', path: '/docs/api.md', desc: 'Full API docs' },
    ],
    Examples: [
      { title: 'Basic Usage', path: '/examples/basic.md', desc: 'Simple examples' },
    ],
  },
};

function generateLlmsTxt(config, baseUrl) {
  let content = `# ${config.name}\n\n`;
  content += `> ${config.description}\n\n`;
  content += `${config.context}\n\n`;

  for (const [section, links] of Object.entries(config.sections)) {
    content += `## ${section}\n\n`;
    for (const link of links) {
      content += `- [${link.title}](${baseUrl}${link.path}): ${link.desc}\n`;
    }
    content += '\n';
  }

  return content;
}

const llmsTxt = generateLlmsTxt(config, 'https://example.com');
fs.writeFileSync('public/llms.txt', llmsTxt);
```
