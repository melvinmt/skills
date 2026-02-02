---
name: rn-ios-extensions
description: Build native iOS extensions from React Native - Apple Watch apps, Dynamic Island, Live Activities, widgets, and share intents. Use when working with expo-apple-targets, Voltra, expo-share-intent, or building native iOS features without leaving RN.
---

# React Native iOS Extensions

Build native iOS experiences from your React Native codebase using three complementary libraries.

## Quick Reference

| Feature | Library | Notes |
|---------|---------|-------|
| Apple Watch apps | expo-apple-targets | Full SwiftUI, WatchConnectivity |
| App Clips | expo-apple-targets | Can include React Native |
| Safari Extensions | expo-apple-targets | JavaScript injection support |
| Widgets (SwiftUI) | expo-apple-targets | Native SwiftUI required |
| Widgets (JSX) | Voltra | React components, no Swift |
| Dynamic Island | Voltra | JSX-based layouts |
| Live Activities | Voltra | Lock screen, push updates |
| Share Sheet | expo-share-intent | iOS + Android support |

---

## expo-apple-targets

Generate native Apple targets (widgets, watch apps, clips) outside `/ios` while keeping Continuous Native Generation benefits.

### Requirements

- CocoaPods 1.16.2+ (Ruby 3.2.0)
- Xcode 16+ (macOS 15 Sequoia)
- Expo SDK 52+

### Setup

```bash
# Generate a target (e.g., widget, watch, clip)
npx create-target widget

# Set Apple Team ID in app.json
# Run prebuild
npx expo prebuild -p ios --clean

# Open Xcode and develop in expo:targets/<target>
xed ios
```

### Target Configuration

Each target needs `targets/<name>/expo-target.config.js`:

```javascript
/** @type {import('@bacons/apple-targets/app.plugin').Config} */
module.exports = {
  type: "widget",
  name: "My Widget",
  icon: "../assets/icon.png",
  deploymentTarget: "17.0",
  colors: {
    $accent: { color: "red", darkColor: "blue" },
    $widgetBackground: "#1a1a1a",
  },
  entitlements: {
    "com.apple.security.application-groups": ["group.com.myapp.data"],
  },
};
```

### Sharing Data Between App and Target

1. Configure App Groups in `app.json`:

```json
{
  "expo": {
    "ios": {
      "entitlements": {
        "com.apple.security.application-groups": ["group.com.myapp.data"]
      }
    }
  }
}
```

2. Set data from React Native:

```javascript
import { ExtensionStorage } from "@bacons/apple-targets";

const storage = new ExtensionStorage("group.com.myapp.data");
storage.set("myKey", { value: 42 });
ExtensionStorage.reloadWidget();
```

3. Read in Swift:

```swift
let defaults = UserDefaults(suiteName: "group.com.myapp.data")
let value = defaults?.string(forKey: "myKey")
```

For full target types and configuration, see [references/expo-apple-targets.md](references/expo-apple-targets.md).

---

## Voltra

Build Live Activities, Dynamic Island layouts, and widgets using React components. No Swift required.

### Installation

```bash
npm install voltra
```

Add to `app.json`:

```json
{
  "plugins": ["voltra"]
}
```

### Live Activity

```javascript
import { startLiveActivity } from 'voltra/client';
import { Voltra } from 'voltra';

const activity = (
  <Voltra.VStack style={{ padding: 16, backgroundColor: '#101828' }}>
    <Voltra.Symbol name="timer" tintColor="#38BDF8" />
    <Voltra.Text style={{ color: '#F8FAFC', fontSize: 18 }}>
      Recording: 02:34
    </Voltra.Text>
  </Voltra.VStack>
);

await startLiveActivity({ lockScreen: activity });
```

### Hook API (with hot reload)

```javascript
import { useLiveActivity } from 'voltra/client';

function RecordingActivity() {
  const { start, update, end } = useLiveActivity();
  
  const handleStart = () => {
    start({
      lockScreen: <TimerUI elapsed={0} />,
      dynamicIsland: {
        compact: <CompactUI />,
        expanded: <ExpandedUI />,
      },
    });
  };
}
```

### Widget

```javascript
import { Voltra } from 'voltra';

export function MyWidget() {
  return (
    <Voltra.VStack style={{ padding: 16 }}>
      <Voltra.Text style={{ fontSize: 24, fontWeight: '600' }}>
        Hello Widget
      </Voltra.Text>
    </Voltra.VStack>
  );
}
```

### Push Updates (Server-Side)

```javascript
import { renderLiveActivityToString } from 'voltra/server';
import { Voltra } from 'voltra';

const payload = renderLiveActivityToString({
  lockScreen: (
    <Voltra.VStack>
      <Voltra.Text>Updated from server!</Voltra.Text>
    </Voltra.VStack>
  ),
});

// Send payload via APNS
```

For components and API details, see [references/voltra.md](references/voltra.md).

---

## expo-share-intent

Handle shared content (URLs, text, images, files) from the iOS/Android share sheet.

### Installation

```bash
npm install expo-share-intent patch-package
```

Add patch file from [example/patches](https://github.com/achorein/expo-share-intent/tree/main/example/basic/patches).

Configure `app.json`:

```json
{
  "scheme": "myapp",
  "plugins": [
    ["expo-share-intent", {
      "iosActivationRules": {
        "NSExtensionActivationSupportsWebURLWithMaxCount": 1,
        "NSExtensionActivationSupportsImageWithMaxCount": 5,
        "NSExtensionActivationSupportsFileWithMaxCount": 1
      },
      "androidIntentFilters": ["text/*", "image/*"]
    }]
  ]
}
```

### Usage

```javascript
import { useShareIntent } from "expo-share-intent";

function App() {
  const { hasShareIntent, shareIntent, resetShareIntent } = useShareIntent();

  useEffect(() => {
    if (hasShareIntent) {
      console.log("Shared text:", shareIntent.text);
      console.log("Shared URL:", shareIntent.webUrl);
      console.log("Shared files:", shareIntent.files);
      resetShareIntent();
    }
  }, [hasShareIntent]);
}
```

### Share Intent Data

| Property | Description |
|----------|-------------|
| `shareIntent.text` | Raw shared text or URL |
| `shareIntent.webUrl` | Extracted URL from text |
| `shareIntent.files` | Array of `{ path, mimeType, fileName, size }` |
| `shareIntent.meta` | Metadata including `title` and Open Graph tags |

For configuration options, see [references/expo-share-intent.md](references/expo-share-intent.md).

---

## Combined Patterns

### Voltra Widget + expo-apple-targets for Advanced Features

Use Voltra for widget UI, expo-apple-targets for App Groups:

```javascript
// React Native side
import { ExtensionStorage } from "@bacons/apple-targets";

const storage = new ExtensionStorage("group.com.myapp.data");
storage.set("widgetData", { count: 42, lastUpdated: Date.now() });
ExtensionStorage.reloadWidget();
```

### Share Intent to Live Activity

When content is shared, start a Live Activity:

```javascript
import { useShareIntent } from "expo-share-intent";
import { startLiveActivity } from "voltra/client";

function App() {
  const { hasShareIntent, shareIntent } = useShareIntent();

  useEffect(() => {
    if (hasShareIntent && shareIntent.files?.length) {
      startLiveActivity({
        lockScreen: <ProcessingUI file={shareIntent.files[0]} />,
      });
    }
  }, [hasShareIntent]);
}
```

---

## EAS Build Notes

- All libraries work with EAS Build
- expo-apple-targets handles code signing automatically
- For expo-share-intent, ensure only one extension target during credentials setup
- Don't commit `/ios` and `/android` folders; rebuild before EAS builds

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Widget not showing | Use iOS 18+, long-press app icon to add widget |
| Share intent config sync failed | Add `xcode+3.0.1.patch` to patches folder |
| Live Activity not updating | Check APNS configuration and push token |
| Watch app not connecting | Verify WatchConnectivity session activation |
