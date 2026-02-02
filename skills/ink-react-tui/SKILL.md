---
name: ink-react-tui
description: Build terminal user interfaces with React using Ink. Use when creating CLIs, terminal apps, TUIs, or when the user mentions Ink, terminal UI, or React CLI apps.
---

# Ink React TUI

Build interactive command-line interfaces using React components with [Ink](https://github.com/vadimdemedes/ink).

## Quick Start

```bash
# Scaffold new project
npx create-ink-app my-cli
npx create-ink-app --typescript my-cli

# Or add to existing project
npm install ink react
```

Basic app:

```tsx
import React from 'react';
import {render, Text} from 'ink';

const App = () => <Text color="green">Hello, CLI!</Text>;

render(<App />);
```

---

## Core Components

### Text

Display and style text. Only text nodes and nested `<Text>` allowed inside.

```tsx
import {Text} from 'ink';

<Text color="green">Green text</Text>
<Text color="#005cc5">Hex color</Text>
<Text bold>Bold</Text>
<Text italic>Italic</Text>
<Text underline>Underlined</Text>
<Text strikethrough>Strikethrough</Text>
<Text inverse>Inverted colors</Text>
<Text dimColor>Dimmed</Text>
<Text backgroundColor="blue" color="white">With background</Text>
```

Text wrapping:

```tsx
<Box width={10}>
  <Text wrap="truncate">Long text gets truncated…</Text>
  <Text wrap="truncate-middle">He…ld</Text>
  <Text wrap="truncate-start">…World</Text>
</Box>
```

### Box

Flexbox container (like `<div style="display: flex">`).

```tsx
import {Box, Text} from 'ink';

// Basic layout
<Box flexDirection="column" padding={1}>
  <Text>Row 1</Text>
  <Text>Row 2</Text>
</Box>

// With border
<Box borderStyle="round" borderColor="green" padding={1}>
  <Text>Bordered content</Text>
</Box>

// Flex alignment
<Box justifyContent="space-between" width={40}>
  <Text>Left</Text>
  <Text>Right</Text>
</Box>
```

Key properties:
- **Dimensions**: `width`, `height`, `minWidth`, `minHeight`
- **Padding**: `padding`, `paddingX`, `paddingY`, `paddingTop`, etc.
- **Margin**: `margin`, `marginX`, `marginY`, `marginTop`, etc.
- **Flex**: `flexDirection`, `flexGrow`, `flexShrink`, `flexWrap`, `alignItems`, `justifyContent`
- **Borders**: `borderStyle` (`single`, `double`, `round`, `bold`, `classic`), `borderColor`
- **Background**: `backgroundColor`
- **Gap**: `gap`, `columnGap`, `rowGap`

For full property reference, see [references/components.md](references/components.md).

### Newline & Spacer

```tsx
import {Text, Newline, Spacer, Box} from 'ink';

// Newline inside Text
<Text>
  Line 1<Newline />Line 2<Newline count={2} />Line 4
</Text>

// Spacer fills available space
<Box width={40}>
  <Text>Left</Text>
  <Spacer />
  <Text>Right</Text>
</Box>
```

### Static

Render permanent output above the dynamic UI (logs, completed tasks).

```tsx
import {Static, Box, Text} from 'ink';

const App = ({completedTasks}) => (
  <>
    <Static items={completedTasks}>
      {task => (
        <Box key={task.id}>
          <Text color="green">✔ {task.title}</Text>
        </Box>
      )}
    </Static>
    <Box marginTop={1}>
      <Text dimColor>Processing...</Text>
    </Box>
  </>
);
```

### Transform

Transform rendered text output.

```tsx
import {Transform, Text} from 'ink';

<Transform transform={output => output.toUpperCase()}>
  <Text>hello world</Text>
</Transform>
// Output: HELLO WORLD
```

---

## Essential Hooks

### useInput

Handle keyboard input.

```tsx
import {useInput, useApp} from 'ink';

const App = () => {
  const {exit} = useApp();

  useInput((input, key) => {
    if (input === 'q') exit();
    if (key.upArrow) { /* move up */ }
    if (key.downArrow) { /* move down */ }
    if (key.return) { /* select */ }
    if (key.escape) { /* cancel */ }
  });

  return <Text>Press q to quit, arrows to navigate</Text>;
};
```

Key properties: `leftArrow`, `rightArrow`, `upArrow`, `downArrow`, `return`, `escape`, `ctrl`, `shift`, `tab`, `backspace`, `delete`, `pageUp`, `pageDown`, `meta`.

Disable input capture: `useInput(handler, {isActive: false})`.

### useApp

Control app lifecycle.

```tsx
import {useApp} from 'ink';

const App = () => {
  const {exit} = useApp();
  
  // Exit after 5 seconds
  useEffect(() => {
    setTimeout(() => exit(), 5000);
  }, []);

  return <Text>Exiting soon...</Text>;
};
```

### useFocus & useFocusManager

Manage focus between components.

```tsx
import {useFocus, Box, Text} from 'ink';

const FocusableItem = ({label}) => {
  const {isFocused} = useFocus();
  
  return (
    <Box>
      <Text color={isFocused ? 'green' : undefined}>
        {isFocused ? '>' : ' '} {label}
      </Text>
    </Box>
  );
};

// Tab cycles focus, Shift+Tab goes backwards
```

Focus options: `autoFocus`, `isActive`, `id`.

For complete hooks reference, see [references/hooks.md](references/hooks.md).

---

## Common Patterns

### Interactive List

```tsx
import React, {useState} from 'react';
import {render, Box, Text, useInput, useApp} from 'ink';

const items = ['Option A', 'Option B', 'Option C'];

const SelectList = () => {
  const [selected, setSelected] = useState(0);
  const {exit} = useApp();

  useInput((input, key) => {
    if (key.upArrow) setSelected(s => Math.max(0, s - 1));
    if (key.downArrow) setSelected(s => Math.min(items.length - 1, s + 1));
    if (key.return) {
      console.log(`Selected: ${items[selected]}`);
      exit();
    }
    if (input === 'q') exit();
  });

  return (
    <Box flexDirection="column">
      {items.map((item, i) => (
        <Text key={item} color={i === selected ? 'green' : undefined}>
          {i === selected ? '>' : ' '} {item}
        </Text>
      ))}
    </Box>
  );
};

render(<SelectList />);
```

### Progress Indicator

```tsx
import React, {useState, useEffect} from 'react';
import {render, Box, Text} from 'ink';

const ProgressBar = ({percent}) => {
  const width = 20;
  const filled = Math.round(width * percent / 100);
  
  return (
    <Box>
      <Text>[</Text>
      <Text color="green">{'█'.repeat(filled)}</Text>
      <Text>{'░'.repeat(width - filled)}</Text>
      <Text>] {percent}%</Text>
    </Box>
  );
};

const App = () => {
  const [progress, setProgress] = useState(0);

  useEffect(() => {
    const timer = setInterval(() => {
      setProgress(p => p >= 100 ? 100 : p + 10);
    }, 200);
    return () => clearInterval(timer);
  }, []);

  return <ProgressBar percent={progress} />;
};

render(<App />);
```

### Task List with Static Output

```tsx
import React, {useState, useEffect} from 'react';
import {render, Static, Box, Text} from 'ink';

const App = () => {
  const [completed, setCompleted] = useState([]);
  const [current, setCurrent] = useState('Task 1');

  useEffect(() => {
    const tasks = ['Task 1', 'Task 2', 'Task 3'];
    let i = 0;
    const timer = setInterval(() => {
      if (i < tasks.length) {
        setCompleted(prev => [...prev, {id: i, title: tasks[i]}]);
        i++;
        setCurrent(tasks[i] || 'Done!');
      }
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  return (
    <>
      <Static items={completed}>
        {task => (
          <Text key={task.id} color="green">✔ {task.title}</Text>
        )}
      </Static>
      <Text color="yellow">⏳ {current}</Text>
    </>
  );
};

render(<App />);
```

---

## Render API

```tsx
import {render} from 'ink';

const {rerender, unmount, waitUntilExit, clear} = render(<App />);

// Update props
rerender(<App count={2} />);

// Programmatic exit
unmount();

// Wait for exit
await waitUntilExit();

// Clear output
clear();
```

Options:

```tsx
render(<App />, {
  stdout: process.stdout,
  stdin: process.stdin,
  stderr: process.stderr,
  exitOnCtrlC: true,
  patchConsole: true,
  debug: false,
  maxFps: 30,
});
```

---

## Testing

Use `ink-testing-library`:

```tsx
import {render} from 'ink-testing-library';
import {Text} from 'ink';

const {lastFrame} = render(<Text>Hello</Text>);
expect(lastFrame()).toBe('Hello');
```

---

## Useful Third-Party Components

| Package | Purpose |
|---------|---------|
| `ink-text-input` | Text input field |
| `ink-spinner` | Loading spinners |
| `ink-select-input` | Select/dropdown |
| `ink-progress-bar` | Progress bars |
| `ink-table` | Data tables |
| `ink-link` | Terminal hyperlinks |
| `ink-gradient` | Gradient text |
| `ink-big-text` | Large ASCII text |

For full list, see [references/useful-components.md](references/useful-components.md).
