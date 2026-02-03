# React Native iOS Extensions Skill

Build native iOS extensions from React Native - Apple Watch apps, Dynamic Island, Live Activities, widgets, and share intents.

## What's Included

- **expo-apple-targets**: Widgets, Watch apps, App Clips, Safari Extensions
- **Voltra**: Live Activities, Dynamic Island, widgets using JSX
- **expo-share-intent**: Handle shared content from iOS/Android share sheet
- **Combined Patterns**: Mixing libraries for advanced features

## Installation

```bash
npx skills add melvinmt/skills --skill rn-ios-extensions
```

<details>
<summary>bun</summary>

```bash
bunx skills add melvinmt/skills --skill rn-ios-extensions
```

</details>

<details>
<summary>pnpm</summary>

```bash
pnpm dlx skills add melvinmt/skills --skill rn-ios-extensions
```

</details>

## Feature Matrix

| Feature | Library | Notes |
|---------|---------|-------|
| Apple Watch apps | expo-apple-targets | Full SwiftUI |
| Widgets (SwiftUI) | expo-apple-targets | Native Swift required |
| Widgets (JSX) | Voltra | React components |
| Dynamic Island | Voltra | JSX-based layouts |
| Live Activities | Voltra | Lock screen, push updates |
| Share Sheet | expo-share-intent | iOS + Android |

## License

MIT
