# Content Negotiation Implementation

Return markdown content when agents request it via the `Accept: text/markdown` header.

---

## Express.js Middleware

```javascript
const path = require('path');
const fs = require('fs');

function contentNegotiation(markdownDir) {
  return (req, res, next) => {
    const acceptHeader = req.headers.accept || '';
    
    if (!acceptHeader.includes('text/markdown')) {
      return next();
    }

    const mdPath = resolveMarkdownPath(req.path, markdownDir);
    
    if (fs.existsSync(mdPath)) {
      res.type('text/markdown; charset=utf-8');
      return res.sendFile(mdPath);
    }

    next();
  };
}

function resolveMarkdownPath(urlPath, markdownDir) {
  // Handle root path
  if (urlPath === '/' || urlPath === '') {
    return path.join(markdownDir, 'index.md');
  }
  
  // Remove leading slash and add .md extension
  const relativePath = urlPath.replace(/^\//, '') + '.md';
  return path.join(markdownDir, relativePath);
}

// Usage
app.use(contentNegotiation(path.join(__dirname, 'content/markdown')));
```

---

## Next.js Middleware

```typescript
// middleware.ts
import { NextRequest, NextResponse } from 'next/server';
import { readFile } from 'fs/promises';
import path from 'path';

export async function middleware(request: NextRequest) {
  const acceptHeader = request.headers.get('accept') || '';
  
  if (!acceptHeader.includes('text/markdown')) {
    return NextResponse.next();
  }

  const pathname = request.nextUrl.pathname;
  const mdPath = getMdPath(pathname);

  try {
    // For static .md files in public directory
    const mdUrl = new URL(mdPath, request.url);
    const response = await fetch(mdUrl);
    
    if (response.ok) {
      const markdown = await response.text();
      return new NextResponse(markdown, {
        headers: { 'Content-Type': 'text/markdown; charset=utf-8' },
      });
    }
  } catch {
    // Fall through to normal response
  }

  return NextResponse.next();
}

function getMdPath(pathname: string): string {
  if (pathname === '/' || pathname === '') {
    return '/index.md';
  }
  return pathname + '.md';
}

export const config = {
  matcher: ['/((?!api|_next/static|_next/image|favicon.ico).*)'],
};
```

---

## Next.js API Route Alternative

```typescript
// pages/api/markdown/[...path].ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { readFileSync, existsSync } from 'fs';
import path from 'path';

export default function handler(req: NextApiRequest, res: NextApiResponse) {
  const pathSegments = req.query.path as string[];
  const mdPath = path.join(process.cwd(), 'content', ...pathSegments) + '.md';

  if (!existsSync(mdPath)) {
    return res.status(404).send('Markdown not found');
  }

  const content = readFileSync(mdPath, 'utf-8');
  res.setHeader('Content-Type', 'text/markdown; charset=utf-8');
  res.send(content);
}
```

---

## Fastify Plugin

```javascript
async function contentNegotiationPlugin(fastify, options) {
  const { markdownDir } = options;

  fastify.addHook('preHandler', async (request, reply) => {
    const acceptHeader = request.headers.accept || '';
    
    if (!acceptHeader.includes('text/markdown')) {
      return;
    }

    const mdPath = resolveMarkdownPath(request.url, markdownDir);
    
    try {
      const content = await fs.promises.readFile(mdPath, 'utf-8');
      reply.type('text/markdown; charset=utf-8').send(content);
    } catch {
      // File doesn't exist, continue to normal handler
    }
  });
}

function resolveMarkdownPath(url, markdownDir) {
  const pathname = new URL(url, 'http://localhost').pathname;
  if (pathname === '/' || pathname === '') {
    return path.join(markdownDir, 'index.md');
  }
  return path.join(markdownDir, pathname + '.md');
}

// Usage
fastify.register(contentNegotiationPlugin, { 
  markdownDir: './content/markdown' 
});
```

---

## Python Flask

```python
from flask import Flask, request, send_file, abort
from pathlib import Path

app = Flask(__name__)
MARKDOWN_DIR = Path('./content/markdown')

@app.before_request
def content_negotiation():
    accept = request.headers.get('Accept', '')
    
    if 'text/markdown' not in accept:
        return None
    
    path = request.path
    if path == '/' or path == '':
        md_path = MARKDOWN_DIR / 'index.md'
    else:
        md_path = MARKDOWN_DIR / (path.lstrip('/') + '.md')
    
    if md_path.exists():
        return send_file(md_path, mimetype='text/markdown; charset=utf-8')
    
    return None
```

---

## Python FastAPI

```python
from fastapi import FastAPI, Request
from fastapi.responses import PlainTextResponse
from pathlib import Path

app = FastAPI()
MARKDOWN_DIR = Path('./content/markdown')

@app.middleware("http")
async def content_negotiation(request: Request, call_next):
    accept = request.headers.get('accept', '')
    
    if 'text/markdown' in accept:
        path = request.url.path
        if path == '/' or path == '':
            md_path = MARKDOWN_DIR / 'index.md'
        else:
            md_path = MARKDOWN_DIR / (path.lstrip('/') + '.md')
        
        if md_path.exists():
            content = md_path.read_text()
            return PlainTextResponse(
                content, 
                media_type='text/markdown; charset=utf-8'
            )
    
    return await call_next(request)
```

---

## Nginx Configuration

For serving static `.md` files with content negotiation:

```nginx
map $http_accept $markdown_suffix {
    default "";
    "~text/markdown" ".md";
}

server {
    location / {
        # Try markdown version first if Accept header matches
        try_files $uri$markdown_suffix $uri $uri/ =404;
        
        # Set correct content type for .md files
        location ~ \.md$ {
            add_header Content-Type "text/markdown; charset=utf-8";
        }
    }
}
```

---

## Testing

```bash
# Test content negotiation
curl -H "Accept: text/markdown" https://yoursite.com/about

# Should return markdown content, not HTML

# Compare with normal request
curl https://yoursite.com/about
# Should return HTML
```
