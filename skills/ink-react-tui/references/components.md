# Ink Components Reference

Complete reference for all Ink built-in components.

## Text

Display and style text content. Only accepts text nodes and nested `<Text>` components.

### Props

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `color` | `string` | - | Text color (named, hex, rgb) |
| `backgroundColor` | `string` | - | Background color |
| `dimColor` | `boolean` | `false` | Dim the color |
| `bold` | `boolean` | `false` | Bold text |
| `italic` | `boolean` | `false` | Italic text |
| `underline` | `boolean` | `false` | Underlined text |
| `strikethrough` | `boolean` | `false` | Strikethrough text |
| `inverse` | `boolean` | `false` | Swap foreground/background |
| `wrap` | `string` | `'wrap'` | Text wrapping mode |

### Color Values

Uses [chalk](https://github.com/chalk/chalk) under the hood:

```tsx
<Text color="green">Named color</Text>
<Text color="#ff5500">Hex color</Text>
<Text color="rgb(255, 136, 0)">RGB color</Text>
```

### Wrap Modes

| Value | Description |
|-------|-------------|
| `wrap` | Wrap to multiple lines (default) |
| `truncate` | Truncate at end with ellipsis |
| `truncate-end` | Same as `truncate` |
| `truncate-start` | Ellipsis at start |
| `truncate-middle` | Ellipsis in middle |

```tsx
<Box width={10}>
  <Text wrap="truncate">Hello World</Text>
</Box>
// Output: Hello Wor…
```

---

## Box

Flexbox container for layout. Every `<Box>` behaves like `display: flex`.

### Dimensions

| Prop | Type | Description |
|------|------|-------------|
| `width` | `number \| string` | Width in spaces or percentage |
| `height` | `number \| string` | Height in lines or percentage |
| `minWidth` | `number` | Minimum width |
| `minHeight` | `number` | Minimum height |

```tsx
<Box width={20}>Fixed width</Box>
<Box width="50%">Half of parent</Box>
<Box height={5}>5 lines tall</Box>
```

### Padding

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `padding` | `number` | `0` | All sides |
| `paddingX` | `number` | `0` | Left and right |
| `paddingY` | `number` | `0` | Top and bottom |
| `paddingTop` | `number` | `0` | Top only |
| `paddingBottom` | `number` | `0` | Bottom only |
| `paddingLeft` | `number` | `0` | Left only |
| `paddingRight` | `number` | `0` | Right only |

### Margin

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `margin` | `number` | `0` | All sides |
| `marginX` | `number` | `0` | Left and right |
| `marginY` | `number` | `0` | Top and bottom |
| `marginTop` | `number` | `0` | Top only |
| `marginBottom` | `number` | `0` | Bottom only |
| `marginLeft` | `number` | `0` | Left only |
| `marginRight` | `number` | `0` | Right only |

### Gap

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `gap` | `number` | `0` | Gap between children |
| `columnGap` | `number` | `0` | Horizontal gap |
| `rowGap` | `number` | `0` | Vertical gap |

```tsx
<Box gap={1} flexWrap="wrap" width={10}>
  <Text>A</Text>
  <Text>B</Text>
  <Text>C</Text>
</Box>
```

### Flexbox

| Prop | Type | Default | Values |
|------|------|---------|--------|
| `flexDirection` | `string` | `'row'` | `row`, `row-reverse`, `column`, `column-reverse` |
| `flexGrow` | `number` | `0` | Grow factor |
| `flexShrink` | `number` | `1` | Shrink factor |
| `flexBasis` | `number \| string` | - | Base size |
| `flexWrap` | `string` | `'nowrap'` | `nowrap`, `wrap`, `wrap-reverse` |
| `alignItems` | `string` | - | `flex-start`, `center`, `flex-end` |
| `alignSelf` | `string` | `'auto'` | `auto`, `flex-start`, `center`, `flex-end` |
| `justifyContent` | `string` | - | `flex-start`, `center`, `flex-end`, `space-between`, `space-around`, `space-evenly` |

```tsx
// Horizontal layout with spacing
<Box justifyContent="space-between" width={40}>
  <Text>Left</Text>
  <Text>Right</Text>
</Box>

// Vertical layout
<Box flexDirection="column">
  <Text>Top</Text>
  <Text>Bottom</Text>
</Box>

// Fill remaining space
<Box>
  <Text>Label:</Text>
  <Box flexGrow={1}>
    <Text>Fills remaining space</Text>
  </Box>
</Box>
```

### Borders

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `borderStyle` | `string \| object` | - | Border style |
| `borderColor` | `string` | - | All border colors |
| `borderTopColor` | `string` | - | Top border color |
| `borderBottomColor` | `string` | - | Bottom border color |
| `borderLeftColor` | `string` | - | Left border color |
| `borderRightColor` | `string` | - | Right border color |
| `borderDimColor` | `boolean` | `false` | Dim all borders |
| `borderTop` | `boolean` | `true` | Show top border |
| `borderBottom` | `boolean` | `true` | Show bottom border |
| `borderLeft` | `boolean` | `true` | Show left border |
| `borderRight` | `boolean` | `true` | Show right border |

Border styles: `single`, `double`, `round`, `bold`, `singleDouble`, `doubleSingle`, `classic`.

```tsx
<Box borderStyle="round" borderColor="green" padding={1}>
  <Text>Rounded green border</Text>
</Box>

// Custom border characters
<Box borderStyle={{
  topLeft: '╭',
  top: '─',
  topRight: '╮',
  left: '│',
  bottomLeft: '╰',
  bottom: '─',
  bottomRight: '╯',
  right: '│'
}}>
  <Text>Custom border</Text>
</Box>
```

### Background

| Prop | Type | Description |
|------|------|-------------|
| `backgroundColor` | `string` | Background color for entire box |

```tsx
<Box backgroundColor="blue" padding={1}>
  <Text color="white">Blue background</Text>
</Box>
```

### Visibility

| Prop | Type | Default | Values |
|------|------|---------|--------|
| `display` | `string` | `'flex'` | `flex`, `none` |
| `overflow` | `string` | `'visible'` | `visible`, `hidden` |
| `overflowX` | `string` | `'visible'` | `visible`, `hidden` |
| `overflowY` | `string` | `'visible'` | `visible`, `hidden` |

---

## Newline

Insert newline characters. Must be inside `<Text>`.

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `count` | `number` | `1` | Number of newlines |

```tsx
<Text>
  Line 1<Newline />
  Line 2<Newline count={2} />
  Line 4
</Text>
```

---

## Spacer

Flexible space that fills available room on the main axis.

```tsx
// Horizontal spacing
<Box>
  <Text>Left</Text>
  <Spacer />
  <Text>Right</Text>
</Box>

// Vertical spacing
<Box flexDirection="column" height={10}>
  <Text>Top</Text>
  <Spacer />
  <Text>Bottom</Text>
</Box>
```

---

## Static

Permanently render items above the dynamic UI. Items are rendered once and never re-rendered.

| Prop | Type | Description |
|------|------|-------------|
| `items` | `Array<T>` | Array of items to render |
| `style` | `object` | Container styles (Box props) |
| `children` | `(item: T, index: number) => ReactNode` | Render function |

```tsx
<Static items={completedTasks}>
  {(task, index) => (
    <Box key={task.id}>
      <Text color="green">✔ {task.title}</Text>
    </Box>
  )}
</Static>
```

Important: Only new items are rendered. Previously rendered items are not updated.

---

## Transform

Transform the string output of child components.

| Prop | Type | Description |
|------|------|-------------|
| `transform` | `(line: string, index: number) => string` | Transform function |

```tsx
// Uppercase
<Transform transform={output => output.toUpperCase()}>
  <Text>hello</Text>
</Transform>

// Hanging indent
<Transform transform={(line, index) => 
  index === 0 ? line : '    ' + line
}>
  <Text>{longText}</Text>
</Transform>
```

Note: Transform must not change output dimensions or layout will break.
