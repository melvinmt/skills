# Useful Ink Components

Third-party components to extend Ink functionality.

## Input Components

### ink-text-input

Text input field with cursor.

```bash
npm install ink-text-input
```

```tsx
import TextInput from 'ink-text-input';

const App = () => {
  const [value, setValue] = useState('');

  return (
    <Box>
      <Text>Enter name: </Text>
      <TextInput value={value} onChange={setValue} />
    </Box>
  );
};
```

Props: `value`, `onChange`, `placeholder`, `focus`, `mask` (for passwords), `showCursor`.

### ink-select-input

Select/dropdown menu with arrow navigation.

```bash
npm install ink-select-input
```

```tsx
import SelectInput from 'ink-select-input';

const items = [
  {label: 'First', value: 'first'},
  {label: 'Second', value: 'second'},
  {label: 'Third', value: 'third'},
];

const App = () => {
  const handleSelect = item => console.log(item.value);

  return <SelectInput items={items} onSelect={handleSelect} />;
};
```

Props: `items`, `onSelect`, `onHighlight`, `initialIndex`, `indicatorComponent`, `itemComponent`, `limit`.

### ink-multi-select

Multi-select with checkboxes.

```bash
npm install ink-multi-select
```

```tsx
import MultiSelect from 'ink-multi-select';

const items = [
  {label: 'Option A', value: 'a'},
  {label: 'Option B', value: 'b'},
  {label: 'Option C', value: 'c'},
];

const App = () => {
  const handleSubmit = items => console.log(items);

  return <MultiSelect items={items} onSubmit={handleSubmit} />;
};
```

### ink-confirm-input

Yes/No confirmation.

```bash
npm install ink-confirm-input
```

```tsx
import ConfirmInput from 'ink-confirm-input';

const App = () => {
  const [confirmed, setConfirmed] = useState(null);

  if (confirmed !== null) {
    return <Text>{confirmed ? 'Confirmed!' : 'Cancelled'}</Text>;
  }

  return (
    <Box>
      <Text>Are you sure? </Text>
      <ConfirmInput onSubmit={setConfirmed} />
    </Box>
  );
};
```

### ink-quicksearch-input

Searchable select with type-to-filter.

```bash
npm install ink-quicksearch-input
```

---

## Display Components

### ink-spinner

Loading spinners.

```bash
npm install ink-spinner
```

```tsx
import Spinner from 'ink-spinner';

const App = () => (
  <Text>
    <Spinner type="dots" /> Loading...
  </Text>
);
```

Types: `dots`, `line`, `pipe`, `simpleDots`, `simpleDotsScrolling`, `star`, `balloon`, `noise`, `bounce`, `boxBounce`, `clock`, `earth`, `moon`, `runner`, `pong`, `shark`, `dqpb`, and many more.

### ink-progress-bar

Progress bar component.

```bash
npm install ink-progress-bar
```

```tsx
import ProgressBar from 'ink-progress-bar';

const App = () => (
  <Box flexDirection="column">
    <ProgressBar percent={0.5} />
    <Text>50% complete</Text>
  </Box>
);
```

Props: `percent` (0-1), `left`, `right`, `character`, `width`.

### ink-table

Render data as tables.

```bash
npm install ink-table
```

```tsx
import Table from 'ink-table';

const data = [
  {name: 'Alice', age: 30, city: 'NYC'},
  {name: 'Bob', age: 25, city: 'LA'},
];

const App = () => <Table data={data} />;
```

Props: `data`, `columns`, `padding`, `header`, `skeleton`, `cell`.

### ink-divider

Horizontal divider line.

```bash
npm install ink-divider
```

```tsx
import Divider from 'ink-divider';

const App = () => (
  <Box flexDirection="column">
    <Text>Section 1</Text>
    <Divider title="Next Section" />
    <Text>Section 2</Text>
  </Box>
);
```

### ink-link

Terminal hyperlinks (for supported terminals).

```bash
npm install ink-link
```

```tsx
import Link from 'ink-link';

const App = () => (
  <Text>
    Visit <Link url="https://example.com">Example</Link>
  </Text>
);
```

### ink-gradient

Gradient colored text.

```bash
npm install ink-gradient
```

```tsx
import Gradient from 'ink-gradient';

const App = () => (
  <Gradient name="rainbow">
    <Text>Rainbow text!</Text>
  </Gradient>
);
```

Names: `rainbow`, `cristal`, `teen`, `mind`, `morning`, `vice`, `passion`, `fruit`, `instagram`, `atlas`, `retro`, `summer`, `pastel`, `rainbow`.

### ink-big-text

Large ASCII text (figlet-style).

```bash
npm install ink-big-text
```

```tsx
import BigText from 'ink-big-text';

const App = () => <BigText text="Hello" font="chrome" />;
```

Fonts: `block`, `slick`, `tiny`, `grid`, `pallet`, `shade`, `simple`, `simpleBlock`, `3d`, `simple3d`, `chrome`, `huge`.

### ink-ascii

ASCII art text with many fonts.

```bash
npm install ink-ascii
```

### ink-image / ink-picture

Display images in terminal.

```bash
npm install ink-image
```

```tsx
import Image from 'ink-image';

const App = () => <Image src="./logo.png" width={50} />;
```

---

## Layout Components

### ink-box / ink-titled-box

Box with a title.

```bash
npm install @paoloricciuti/ink-titled-box
```

```tsx
import TitledBox from '@paoloricciuti/ink-titled-box';

const App = () => (
  <TitledBox title="Settings" borderStyle="round">
    <Text>Content here</Text>
  </TitledBox>
);
```

### ink-scroll-view

Scrollable container.

```bash
npm install ink-scroll-view
```

### ink-scroll-list

Scrollable list with virtualization.

```bash
npm install ink-scroll-list
```

### ink-virtual-list

Virtualized list for performance.

```bash
npm install ink-virtual-list
```

---

## Task & Workflow

### ink-task-list

Task list with status indicators.

```bash
npm install ink-task-list
```

```tsx
import {TaskList, Task} from 'ink-task-list';

const App = () => (
  <TaskList>
    <Task label="Install dependencies" state="success" />
    <Task label="Build project" state="loading" />
    <Task label="Run tests" state="pending" />
  </TaskList>
);
```

States: `pending`, `loading`, `success`, `warning`, `error`.

### ink-stepper

Step-by-step wizard.

```bash
npm install ink-stepper
```

---

## Code & Syntax

### ink-syntax-highlight

Syntax highlighted code.

```bash
npm install ink-syntax-highlight
```

```tsx
import SyntaxHighlight from 'ink-syntax-highlight';

const code = `const x = 1;`;

const App = () => (
  <SyntaxHighlight language="javascript" code={code} />
);
```

### ink-markdown

Render markdown with syntax highlighting.

```bash
npm install ink-markdown
```

---

## Utility

### ink-spawn

Spawn and display child processes.

```bash
npm install ink-spawn
```

### ink-tab

Tabbed interface.

```bash
npm install ink-tab
```

### ink-form

Form with multiple inputs.

```bash
npm install ink-form
```

### ink-chart

Sparkline and bar charts.

```bash
npm install ink-chart
```

---

## Useful Hooks

### ink-use-stdout-dimensions

Subscribe to terminal size changes.

```bash
npm install ink-use-stdout-dimensions
```

```tsx
import {useStdoutDimensions} from 'ink-use-stdout-dimensions';

const App = () => {
  const [columns, rows] = useStdoutDimensions();

  return <Text>Terminal: {columns}x{rows}</Text>;
};
```
