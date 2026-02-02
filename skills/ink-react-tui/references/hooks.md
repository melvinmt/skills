# Ink Hooks Reference

Complete reference for all Ink hooks.

## useInput

Handle keyboard input from the user.

```tsx
import {useInput} from 'ink';

useInput((input, key) => {
  // input: the character typed (string)
  // key: object with key information
});
```

### Key Object Properties

| Property | Type | Default | Description |
|----------|------|---------|-------------|
| `leftArrow` | `boolean` | `false` | Left arrow pressed |
| `rightArrow` | `boolean` | `false` | Right arrow pressed |
| `upArrow` | `boolean` | `false` | Up arrow pressed |
| `downArrow` | `boolean` | `false` | Down arrow pressed |
| `return` | `boolean` | `false` | Enter/Return pressed |
| `escape` | `boolean` | `false` | Escape pressed |
| `ctrl` | `boolean` | `false` | Ctrl held |
| `shift` | `boolean` | `false` | Shift held |
| `tab` | `boolean` | `false` | Tab pressed |
| `backspace` | `boolean` | `false` | Backspace pressed |
| `delete` | `boolean` | `false` | Delete pressed |
| `pageUp` | `boolean` | `false` | Page Up pressed |
| `pageDown` | `boolean` | `false` | Page Down pressed |
| `meta` | `boolean` | `false` | Meta/Command pressed |

### Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `isActive` | `boolean` | `true` | Enable/disable input capture |

```tsx
// Conditional input handling
useInput(handler, {isActive: isModalOpen});
```

### Example

```tsx
import {useInput, useApp} from 'ink';

const App = () => {
  const {exit} = useApp();
  const [position, setPosition] = useState(0);

  useInput((input, key) => {
    if (input === 'q' || key.escape) {
      exit();
    }
    if (key.upArrow) {
      setPosition(p => Math.max(0, p - 1));
    }
    if (key.downArrow) {
      setPosition(p => p + 1);
    }
    if (key.return) {
      // Handle selection
    }
    if (key.ctrl && input === 'c') {
      // Ctrl+C (if exitOnCtrlC is false)
    }
  });

  return <Text>Position: {position}</Text>;
};
```

---

## useApp

Control the Ink application lifecycle.

```tsx
import {useApp} from 'ink';

const {exit} = useApp();
```

### Methods

#### exit(error?)

Unmount the entire Ink app.

| Parameter | Type | Description |
|-----------|------|-------------|
| `error` | `Error` | Optional error (causes `waitUntilExit` to reject) |

```tsx
const App = () => {
  const {exit} = useApp();

  useEffect(() => {
    doAsyncWork()
      .then(() => exit())
      .catch(err => exit(err));
  }, []);

  return <Text>Working...</Text>;
};
```

---

## useStdin

Access the stdin stream.

```tsx
import {useStdin} from 'ink';

const {stdin, isRawModeSupported, setRawMode} = useStdin();
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `stdin` | `stream.Readable` | The stdin stream |
| `isRawModeSupported` | `boolean` | Whether raw mode is available |
| `setRawMode` | `(enabled: boolean) => void` | Enable/disable raw mode |

### Example

```tsx
const App = () => {
  const {isRawModeSupported} = useStdin();

  if (!isRawModeSupported) {
    return <Text>Interactive input not supported</Text>;
  }

  return <InteractiveComponent />;
};
```

---

## useStdout

Access the stdout stream and write directly to it.

```tsx
import {useStdout} from 'ink';

const {stdout, write} = useStdout();
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `stdout` | `stream.Writable` | The stdout stream |
| `write` | `(data: string) => void` | Write to stdout without affecting Ink output |

```tsx
const App = () => {
  const {write} = useStdout();

  useEffect(() => {
    write('External log message\n');
  }, []);

  return <Text>Ink UI here</Text>;
};
```

---

## useStderr

Access the stderr stream.

```tsx
import {useStderr} from 'ink';

const {stderr, write} = useStderr();
```

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `stderr` | `stream.Writable` | The stderr stream |
| `write` | `(data: string) => void` | Write to stderr without affecting Ink output |

---

## useFocus

Make a component focusable with Tab navigation.

```tsx
import {useFocus} from 'ink';

const {isFocused} = useFocus(options);
```

### Return Value

| Property | Type | Description |
|----------|------|-------------|
| `isFocused` | `boolean` | Whether this component is currently focused |

### Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `autoFocus` | `boolean` | `false` | Auto-focus if no component is focused |
| `isActive` | `boolean` | `true` | Enable/disable focus for this component |
| `id` | `string` | - | Focus ID for programmatic focusing |

### Example

```tsx
import {useFocus, Box, Text} from 'ink';

const FocusableInput = ({label}) => {
  const {isFocused} = useFocus();

  return (
    <Box borderStyle={isFocused ? 'bold' : 'single'}>
      <Text color={isFocused ? 'green' : undefined}>
        {label}
      </Text>
    </Box>
  );
};

const App = () => (
  <Box flexDirection="column">
    <FocusableInput label="First" />
    <FocusableInput label="Second" />
    <FocusableInput label="Third" />
  </Box>
);

// Tab cycles through: First -> Second -> Third -> First
// Shift+Tab goes backwards
```

### With ID for Programmatic Focus

```tsx
const Input = ({id, label}) => {
  const {isFocused} = useFocus({id});
  return <Text color={isFocused ? 'green' : undefined}>{label}</Text>;
};
```

---

## useFocusManager

Programmatically control focus.

```tsx
import {useFocusManager} from 'ink';

const {enableFocus, disableFocus, focusNext, focusPrevious, focus} = useFocusManager();
```

### Methods

| Method | Description |
|--------|-------------|
| `enableFocus()` | Enable focus management (enabled by default) |
| `disableFocus()` | Disable focus management, unfocus current component |
| `focusNext()` | Focus next component (same as Tab) |
| `focusPrevious()` | Focus previous component (same as Shift+Tab) |
| `focus(id: string)` | Focus component with specific ID |

### Example

```tsx
import {useFocusManager, useInput} from 'ink';

const App = () => {
  const {focus, focusNext} = useFocusManager();

  useInput((input, key) => {
    if (input === '1') focus('input-1');
    if (input === '2') focus('input-2');
    if (key.tab) focusNext();
  });

  return (
    <Box flexDirection="column">
      <FocusableInput id="input-1" label="Press 1" />
      <FocusableInput id="input-2" label="Press 2" />
    </Box>
  );
};
```

---

## useIsScreenReaderEnabled

Detect if a screen reader is active.

```tsx
import {useIsScreenReaderEnabled, Text} from 'ink';

const App = () => {
  const isScreenReaderEnabled = useIsScreenReaderEnabled();

  return (
    <Text>
      {isScreenReaderEnabled
        ? 'Accessible output here'
        : 'Visual output here'}
    </Text>
  );
};
```

Enable screen reader mode via:
- `render(<App />, {isScreenReaderEnabled: true})`
- Environment variable: `INK_SCREEN_READER=true`
