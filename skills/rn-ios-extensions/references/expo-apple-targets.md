# expo-apple-targets Reference

## Supported Target Types

| Type | Description |
|------|-------------|
| `widget` | Home screen widget / Live Activity |
| `watch` | Apple Watch app (with companion iOS app) |
| `clip` | App Clip |
| `safari` | Safari Extension |
| `share` | Share Extension |
| `action` | Share Action |
| `app-intent` | App Intent Extension |
| `notification-content` | Notification Content Extension |
| `notification-service` | Notification Service Extension |
| `intent` | Siri Intent Extension |
| `intent-ui` | Siri Intent UI Extension |
| `spotlight` | Spotlight Index Extension |
| `bg-download` | Background Download Extension |
| `quicklook-thumbnail` | Quick Look Thumbnail Extension |
| `location-push` | Location Push Service Extension |
| `credentials-provider` | Credentials Provider Extension |
| `account-auth` | Account Authentication Extension |
| `device-activity-monitor` | Device Activity Monitor Extension |
| `shield-configuration` | Shield Configuration Extension |
| `shield-action` | Shield Action Extension |

---

## expo-target.config.js Schema

```javascript
/** @type {import('@bacons/apple-targets/app.plugin').Config} */
module.exports = {
  // Required: Target type from table above
  type: "widget",

  // Optional: Display name (defaults to directory name)
  name: "My Widget",

  // Optional: App icon (can be URL)
  icon: "../assets/icon.png",

  // Optional: iOS deployment target (defaults to 18.0)
  deploymentTarget: "17.0",

  // Optional: Bundle identifier
  // Prefix with "." to append to main app bundle ID
  bundleIdentifier: ".mywidget",

  // Optional: Frameworks to link
  frameworks: ["SwiftUI", "WidgetKit"],

  // Optional: Entitlements object
  entitlements: {
    "com.apple.security.application-groups": ["group.com.myapp.data"],
  },

  // Optional: Color assets (generates xcassets)
  colors: {
    // Special colors
    $accent: { color: "red", darkColor: "blue" },
    $widgetBackground: "#1a1a1a",
    // Custom colors for SwiftUI
    gradient1: { light: "#E4975D", dark: "#3E72A0" },
  },

  // Optional: Image assets (generates xcassets)
  images: {
    myImage: "../assets/image.png",
  },

  // Optional: Export JS bundle for React Native targets
  // Use for App Clips or Share Extensions with RN
  exportJs: false,
};
```

### Function Config (Access Expo Config)

```javascript
/** @type {import('@bacons/apple-targets/app.plugin').ConfigFunction} */
module.exports = (config) => ({
  type: "widget",
  entitlements: {
    // Mirror app groups from main app
    "com.apple.security.application-groups":
      config.ios.entitlements["com.apple.security.application-groups"],
  },
});
```

---

## Special Colors

| Name | Build Setting | Purpose |
|------|---------------|---------|
| `$accent` | `ASSETCATALOG_COMPILER_GLOBAL_ACCENT_COLOR_NAME` | Tint color for widget buttons |
| `$widgetBackground` | `ASSETCATALOG_COMPILER_WIDGET_BACKGROUND_COLOR_NAME` | Widget background |

---

## Entitlements

### Automatic Entitlements

- **App Clips**: Automatically set `com.apple.developer.parent-application-identifiers`
- **App Groups**: Mirrors `ios.entitlements['com.apple.security.application-groups']` from `app.json` if defined

### Manual Entitlements

If `entitlements` object is not defined in config, add a `*.entitlements` file to the target directory.

---

## ExtensionStorage API

Share data between React Native app and native targets.

### In React Native

```javascript
import { ExtensionStorage } from "@bacons/apple-targets";

// Create storage with App Group
const storage = new ExtensionStorage("group.com.myapp.data");

// Set values (string, number, object, or array)
storage.set("myKey", "myValue");
storage.set("myData", { count: 42, name: "test" });

// Remove value
storage.set("myKey", undefined);

// Reload all widgets
ExtensionStorage.reloadWidget();

// Reload specific widget by kind
ExtensionStorage.reloadWidget("MyWidgetKind");
```

### In Swift (Widget/Extension)

```swift
import WidgetKit

struct MyWidget: Widget {
    var body: some WidgetConfiguration {
        StaticConfiguration(kind: "MyWidgetKind", provider: Provider()) { entry in
            MyWidgetView(entry: entry)
        }
    }
}

struct Provider: TimelineProvider {
    func getTimeline(in context: Context, completion: @escaping (Timeline<Entry>) -> Void) {
        let defaults = UserDefaults(suiteName: "group.com.myapp.data")
        let value = defaults?.string(forKey: "myKey") ?? "default"
        
        let entry = SimpleEntry(date: Date(), value: value)
        let timeline = Timeline(entries: [entry], policy: .atEnd)
        completion(timeline)
    }
}
```

---

## Widget Example

### Directory Structure

```
targets/
└── my-widget/
    ├── expo-target.config.js
    ├── Info.plist
    ├── MyWidget.swift
    └── assets/
        └── preview.png
```

### expo-target.config.js

```javascript
/** @type {import('@bacons/apple-targets/app.plugin').Config} */
module.exports = {
  type: "widget",
  icon: "../../icons/widget.png",
  colors: {
    $widgetBackground: "#1a1a1a",
    $accent: "#007AFF",
  },
  entitlements: {
    "com.apple.security.application-groups": ["group.com.myapp.data"],
  },
};
```

### MyWidget.swift

```swift
import WidgetKit
import SwiftUI

struct Provider: TimelineProvider {
    func placeholder(in context: Context) -> SimpleEntry {
        SimpleEntry(date: Date(), message: "Loading...")
    }

    func getSnapshot(in context: Context, completion: @escaping (SimpleEntry) -> Void) {
        let entry = SimpleEntry(date: Date(), message: "Preview")
        completion(entry)
    }

    func getTimeline(in context: Context, completion: @escaping (Timeline<SimpleEntry>) -> Void) {
        let defaults = UserDefaults(suiteName: "group.com.myapp.data")
        let message = defaults?.string(forKey: "message") ?? "Hello"
        
        let entry = SimpleEntry(date: Date(), message: message)
        let timeline = Timeline(entries: [entry], policy: .after(Date().addingTimeInterval(60)))
        completion(timeline)
    }
}

struct SimpleEntry: TimelineEntry {
    let date: Date
    let message: String
}

struct MyWidgetEntryView: View {
    var entry: Provider.Entry

    var body: some View {
        VStack {
            Text(entry.message)
                .font(.headline)
        }
        .containerBackground(.fill.tertiary, for: .widget)
    }
}

@main
struct MyWidget: Widget {
    let kind: String = "MyWidget"

    var body: some WidgetConfiguration {
        StaticConfiguration(kind: kind, provider: Provider()) { entry in
            MyWidgetEntryView(entry: entry)
        }
        .configurationDisplayName("My Widget")
        .description("Shows messages from the app.")
        .supportedFamilies([.systemSmall, .systemMedium])
    }
}
```

---

## Apple Watch Example

### expo-target.config.js

```javascript
/** @type {import('@bacons/apple-targets/app.plugin').Config} */
module.exports = {
  type: "watch",
  name: "My Watch App",
  deploymentTarget: "10.0",
  colors: {
    $accent: "#FF6B6B",
  },
};
```

---

## CocoaPods Integration

Add `pods.rb` in target root for custom Podfile modifications:

```ruby
# Example: Enable React Native in App Clip
require File.join(File.dirname(`node --print "require.resolve('react-native/package.json')"`), "scripts/react_native_pods")

use_expo_modules!
config = use_native_modules!

use_react_native!(
  :path => config[:reactNativePath],
  :hermes_enabled => true,
  :app_path => "#{Pod::Config.instance.installation_root}/.."
)
```

---

## Build Commands

```bash
# Generate target
npx create-target widget

# Prebuild with targets
npx expo prebuild -p ios --clean

# Prebuild without React Native (faster for pure Swift targets)
npx expo prebuild --template node_modules/@bacons/apple-targets/prebuild-blank.tgz --clean

# Open in Xcode
xed ios
```
