# Ink React TUI Skill

Build interactive terminal user interfaces with React using [Ink](https://github.com/vadimdemedes/ink).

## What's Included

- **Core Components**: Text, Box, Newline, Spacer, Static, Transform
- **Essential Hooks**: useInput, useApp, useFocus, useFocusManager
- **Common Patterns**: Interactive lists, progress indicators, task lists
- **Render API**: rerender, unmount, waitUntilExit, clear
- **Testing**: ink-testing-library usage

## Installation

```bash
npx skills add melvinmt/skills --skill ink-react-tui
```

<details>
<summary>bun</summary>

```bash
bunx skills add melvinmt/skills --skill ink-react-tui
```

</details>

<details>
<summary>pnpm</summary>

```bash
pnpm dlx skills add melvinmt/skills --skill ink-react-tui
```

</details>

## Quick Start

```tsx
import React from 'react';
import {render, Text} from 'ink';

const App = () => <Text color="green">Hello, CLI!</Text>;

render(<App />);
```

## Key Components

| Component | Purpose |
|-----------|---------|
| `Text` | Display and style text |
| `Box` | Flexbox container layout |
| `Static` | Permanent output above dynamic UI |
| `Spacer` | Fill available space |

## License

MIT
