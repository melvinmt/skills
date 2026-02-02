# Toggle Component Implementation

## HTML Structure

```html
<div id="human-machine-toggle" class="hm-toggle">
  <div class="hm-toggle-buttons">
    <button data-mode="human" class="active" aria-pressed="true">
      <span class="indicator">◉</span> <span class="label">ʜᴜᴍᴀɴ</span>
    </button>
    <button data-mode="machine" aria-pressed="false">
      <span class="indicator">○</span> <span class="label">ᴍᴀᴄʜɪɴᴇ</span>
    </button>
  </div>
  
  <!-- Machine mode toolbar (hidden by default) -->
  <div class="hm-machine-toolbar" hidden>
    <span class="permalink">
      <span class="icon">📎</span>
      <a href="" class="md-link"></a>
    </span>
    <button class="copy-btn" title="Copy markdown">
      <span class="copy-icon">📋</span>
    </button>
  </div>
</div>

<!-- Machine content container (hidden by default) -->
<div id="machine-content" class="machine-content" hidden>
  <pre><code class="markdown-raw"></code></pre>
</div>
```

---

## CSS Styles

```css
.hm-toggle {
  position: fixed;
  bottom: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 9999;
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 8px;
}

.hm-toggle-buttons {
  display: flex;
  background: rgba(0, 0, 0, 0.8);
  border-radius: 24px;
  padding: 4px;
  gap: 4px;
}

.hm-toggle-buttons button {
  background: transparent;
  border: none;
  color: rgba(255, 255, 255, 0.6);
  padding: 8px 16px;
  border-radius: 20px;
  cursor: pointer;
  font-family: inherit;
  font-size: 12px;
  letter-spacing: 0.5px;
  transition: all 0.2s ease;
  display: flex;
  align-items: center;
  gap: 6px;
}

.hm-toggle-buttons button:hover {
  color: rgba(255, 255, 255, 0.9);
}

.hm-toggle-buttons button.active {
  background: rgba(255, 255, 255, 0.15);
  color: #fff;
}

.hm-toggle-buttons .indicator {
  font-size: 10px;
}

.hm-toggle-buttons .label {
  font-variant: small-caps;
  text-transform: lowercase;
}

.hm-machine-toolbar {
  display: flex;
  align-items: center;
  gap: 12px;
  background: rgba(0, 0, 0, 0.7);
  border-radius: 8px;
  padding: 6px 12px;
  font-size: 12px;
}

.hm-machine-toolbar .permalink {
  display: flex;
  align-items: center;
  gap: 6px;
  color: rgba(255, 255, 255, 0.8);
}

.hm-machine-toolbar .md-link {
  color: #4fc3f7;
  text-decoration: none;
}

.hm-machine-toolbar .md-link:hover {
  text-decoration: underline;
}

.hm-machine-toolbar .copy-btn {
  background: transparent;
  border: none;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
  transition: background 0.2s;
}

.hm-machine-toolbar .copy-btn:hover {
  background: rgba(255, 255, 255, 0.1);
}

.machine-content {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 80px;
  background: #1e1e1e;
  overflow: auto;
  z-index: 9998;
}

.machine-content pre {
  margin: 0;
  padding: 24px;
  font-family: 'Monaco', 'Menlo', 'Ubuntu Mono', monospace;
  font-size: 14px;
  line-height: 1.6;
  white-space: pre-wrap;
  word-wrap: break-word;
}

.machine-content code {
  color: #d4d4d4;
}
```

---

## JavaScript

```javascript
class HumanMachineToggle {
  constructor() {
    this.mode = 'human';
    this.markdownContent = null;
    this.init();
  }

  init() {
    this.toggle = document.getElementById('human-machine-toggle');
    this.buttons = this.toggle.querySelectorAll('[data-mode]');
    this.toolbar = this.toggle.querySelector('.hm-machine-toolbar');
    this.mdLink = this.toggle.querySelector('.md-link');
    this.copyBtn = this.toggle.querySelector('.copy-btn');
    this.machineContent = document.getElementById('machine-content');
    this.humanContent = document.querySelector('main') || document.body;

    // Set permalink
    const mdUrl = this.getMdUrl();
    this.mdLink.href = mdUrl;
    this.mdLink.textContent = mdUrl.replace(/^https?:\/\//, '');

    this.bindEvents();
  }

  getMdUrl() {
    const path = window.location.pathname;
    if (path === '/' || path === '') {
      return window.location.origin + '/index.md';
    }
    return window.location.origin + path + '.md';
  }

  bindEvents() {
    this.buttons.forEach(btn => {
      btn.addEventListener('click', () => this.switchMode(btn.dataset.mode));
    });

    this.copyBtn.addEventListener('click', () => this.copyMarkdown());
  }

  async switchMode(mode) {
    if (mode === this.mode) return;
    this.mode = mode;

    // Update button states
    this.buttons.forEach(btn => {
      const isActive = btn.dataset.mode === mode;
      btn.classList.toggle('active', isActive);
      btn.setAttribute('aria-pressed', isActive);
      
      // Update indicator
      const indicator = btn.querySelector('.indicator');
      indicator.textContent = isActive ? '◉' : '○';
    });

    if (mode === 'machine') {
      await this.showMachineMode();
    } else {
      this.showHumanMode();
    }
  }

  async showMachineMode() {
    // Fetch markdown content if not cached
    if (!this.markdownContent) {
      try {
        const response = await fetch(this.getMdUrl());
        this.markdownContent = await response.text();
      } catch (error) {
        this.markdownContent = 'Error loading markdown content';
      }
    }

    // Show markdown content
    this.machineContent.querySelector('.markdown-raw').textContent = this.markdownContent;
    this.machineContent.hidden = false;
    this.toolbar.hidden = false;
    
    // Hide human content
    this.humanContent.style.display = 'none';
  }

  showHumanMode() {
    this.machineContent.hidden = true;
    this.toolbar.hidden = true;
    this.humanContent.style.display = '';
  }

  async copyMarkdown() {
    if (!this.markdownContent) return;

    try {
      await navigator.clipboard.writeText(this.markdownContent);
      
      // Visual feedback
      const icon = this.copyBtn.querySelector('.copy-icon');
      const originalText = icon.textContent;
      icon.textContent = '✓';
      setTimeout(() => icon.textContent = originalText, 1500);
    } catch (error) {
      console.error('Failed to copy:', error);
    }
  }
}

// Initialize on DOM ready
document.addEventListener('DOMContentLoaded', () => {
  new HumanMachineToggle();
});
```

---

## React Component

```tsx
import { useState, useEffect } from 'react';

interface HumanMachineToggleProps {
  mdUrl?: string;
}

export function HumanMachineToggle({ mdUrl }: HumanMachineToggleProps) {
  const [mode, setMode] = useState<'human' | 'machine'>('human');
  const [markdown, setMarkdown] = useState<string>('');
  const [copied, setCopied] = useState(false);

  const resolvedMdUrl = mdUrl || getMdUrl();

  function getMdUrl() {
    const path = typeof window !== 'undefined' ? window.location.pathname : '/';
    if (path === '/' || path === '') return '/index.md';
    return path + '.md';
  }

  useEffect(() => {
    if (mode === 'machine' && !markdown) {
      fetch(resolvedMdUrl)
        .then(res => res.text())
        .then(setMarkdown)
        .catch(() => setMarkdown('Error loading markdown'));
    }
  }, [mode, resolvedMdUrl, markdown]);

  async function copyToClipboard() {
    await navigator.clipboard.writeText(markdown);
    setCopied(true);
    setTimeout(() => setCopied(false), 1500);
  }

  return (
    <>
      {mode === 'machine' && (
        <div className="machine-content">
          <pre><code>{markdown}</code></pre>
        </div>
      )}

      <div className="hm-toggle">
        <div className="hm-toggle-buttons">
          <button
            onClick={() => setMode('human')}
            className={mode === 'human' ? 'active' : ''}
            aria-pressed={mode === 'human'}
          >
            <span>{mode === 'human' ? '◉' : '○'}</span>
            <span className="label">ʜᴜᴍᴀɴ</span>
          </button>
          <button
            onClick={() => setMode('machine')}
            className={mode === 'machine' ? 'active' : ''}
            aria-pressed={mode === 'machine'}
          >
            <span>{mode === 'machine' ? '◉' : '○'}</span>
            <span className="label">ᴍᴀᴄʜɪɴᴇ</span>
          </button>
        </div>

        {mode === 'machine' && (
          <div className="hm-machine-toolbar">
            <span className="permalink">
              📎 <a href={resolvedMdUrl}>{resolvedMdUrl}</a>
            </span>
            <button onClick={copyToClipboard} title="Copy markdown">
              {copied ? '✓' : '📋'}
            </button>
          </div>
        )}
      </div>
    </>
  );
}
```

---

## Notes

- Toggle always starts in HUMAN mode (no persistence)
- Machine mode fetches `.md` content on first activation
- Markdown content is cached for the session
- Copy button provides visual feedback on success
