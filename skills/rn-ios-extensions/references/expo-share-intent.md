# expo-share-intent Reference

Handle shared content from the iOS/Android share sheet.

## Installation

```bash
npm install expo-share-intent expo-linking patch-package
```

### Required Patch

Copy patch file to `patches/xcode+3.0.1.patch`:

```diff
--- a/node_modules/xcode/lib/pbxProject.js
+++ b/node_modules/xcode/lib/pbxProject.js
@@ -1680,7 +1680,10 @@ function pbxBuildFileSection(project) {
     return project.pbxBuildFileSection();
 }

-function pbxBuildFileForFile(file) {
+function pbxBuildFileForFile(file, buildFileUUID) {
+    if (buildFileUUID) {
+        return { uuid: buildFileUUID, isa: 'PBXBuildFile', fileRef: file.fileRef };
+    }
     return {
         isa: 'PBXBuildFile',
         fileRef: file.fileRef,
```

Add to `package.json`:

```json
{
  "scripts": {
    "postinstall": "patch-package"
  }
}
```

---

## Configuration

### Basic Setup

In `app.json`:

```json
{
  "scheme": "myapp",
  "plugins": ["expo-share-intent"]
}
```

### Full Configuration

```json
{
  "scheme": "myapp",
  "plugins": [
    ["expo-share-intent", {
      "iosActivationRules": {
        "NSExtensionActivationSupportsText": true,
        "NSExtensionActivationSupportsWebURLWithMaxCount": 1,
        "NSExtensionActivationSupportsWebPageWithMaxCount": 1,
        "NSExtensionActivationSupportsImageWithMaxCount": 10,
        "NSExtensionActivationSupportsMovieWithMaxCount": 5,
        "NSExtensionActivationSupportsFileWithMaxCount": 5
      },
      "androidIntentFilters": ["text/*", "image/*", "video/*", "*/*"],
      "androidMultiIntentFilters": ["image/*", "video/*"],
      "iosShareExtensionName": "MyApp Share Extension"
    }]
  ]
}
```

---

## iOS Activation Rules

| Rule | Type | Description |
|------|------|-------------|
| `NSExtensionActivationSupportsText` | boolean | Enable text sharing |
| `NSExtensionActivationSupportsWebURLWithMaxCount` | number | Max URLs to accept |
| `NSExtensionActivationSupportsWebPageWithMaxCount` | number | Max web pages (includes metadata) |
| `NSExtensionActivationSupportsImageWithMaxCount` | number | Max images to accept |
| `NSExtensionActivationSupportsMovieWithMaxCount` | number | Max videos to accept |
| `NSExtensionActivationSupportsFileWithMaxCount` | number | Max files to accept |

Default:
```json
{
  "NSExtensionActivationSupportsWebURLWithMaxCount": 1,
  "NSExtensionActivationSupportsWebPageWithMaxCount": 1
}
```

### Custom Query

For complex rules, use a predicate string:

```json
{
  "iosActivationRules": "SUBQUERY(extensionItems, $item, SUBQUERY($item.attachments, $attachment, ANY $attachment.registeredTypeIdentifiers UTI-CONFORMS-TO \"public.image\").@count > 0).@count > 0"
}
```

---

## Android Intent Filters

| MIME Type | Description |
|-----------|-------------|
| `text/*` | Text and URLs |
| `image/*` | Images |
| `video/*` | Videos |
| `audio/*` | Audio files |
| `*/*` | All file types |

### Single File Sharing

```json
{
  "androidIntentFilters": ["text/*", "image/*"]
}
```

### Multiple File Sharing

```json
{
  "androidMultiIntentFilters": ["image/*", "video/*", "audio/*"]
}
```

---

## Plugin Options

| Option | Description |
|--------|-------------|
| `iosActivationRules` | Object or string for iOS share rules |
| `iosShareExtensionName` | Custom name for share extension |
| `iosAppGroupIdentifier` | Custom app group (e.g., `group.custom.myapp`) |
| `androidIntentFilters` | MIME types for single file sharing |
| `androidMultiIntentFilters` | MIME types for multiple file sharing |
| `androidMainActivityAttributes` | Activity attributes (default: `{ "android:launchMode": "singleTask" }`) |
| `preprocessorInjectJS` | JavaScript to inject into webpage preprocessor |
| `disableAndroid` | Disable Android share intent |
| `disableIOS` | Disable iOS share extension |

---

## Hook API

### useShareIntent

```javascript
import { useShareIntent } from "expo-share-intent";

function App() {
  const {
    hasShareIntent,   // boolean - true when share data is available
    shareIntent,      // ShareIntent object
    resetShareIntent, // () => void - clear share data
    error,            // string | null - error message
  } = useShareIntent();

  useEffect(() => {
    if (hasShareIntent) {
      handleSharedContent(shareIntent);
      resetShareIntent();
    }
  }, [hasShareIntent]);
}
```

### Options

```javascript
const { shareIntent } = useShareIntent({
  // Disable in Expo Go (no native module)
  disabled: false,
  
  // Debug logging
  debug: false,
  
  // Reset on state change
  resetOnBackground: true,
});
```

---

## ShareIntent Object

```typescript
interface ShareIntent {
  // Raw text content or URL string
  text?: string;
  
  // Extracted URL from text
  webUrl?: string;
  
  // Shared files
  files?: ShareIntentFile[];
  
  // Metadata (iOS web pages)
  meta?: {
    title?: string;
    [key: string]: string; // og:image, description, etc.
  };
}

interface ShareIntentFile {
  path: string;           // file:///local/path/filename
  mimeType: string;       // image/jpeg, video/mp4, etc.
  fileName: string;       // original filename
  size: number;           // size in bytes
  width?: number;         // image/video width
  height?: number;        // image/video height
  duration?: number;      // video duration in ms
}
```

### Examples

**Shared URL:**
```javascript
{
  text: "Check this out: https://example.com/article",
  webUrl: "https://example.com/article",
  meta: {
    title: "My Article",
    "og:image": "https://example.com/image.jpg"
  }
}
```

**Shared Image:**
```javascript
{
  files: [{
    path: "file:///var/mobile/.../image.jpg",
    mimeType: "image/jpeg",
    fileName: "photo.jpg",
    size: 2567402,
    width: 1920,
    height: 1080
  }]
}
```

---

## Provider API

For apps with multiple screens/providers:

```javascript
import { ShareIntentProvider, useShareIntentContext } from "expo-share-intent";

// Root component
export default function App() {
  return (
    <ShareIntentProvider>
      <NavigationContainer>
        <RootNavigator />
      </NavigationContainer>
    </ShareIntentProvider>
  );
}

// Any nested component
function ShareHandler() {
  const { hasShareIntent, shareIntent, resetShareIntent } = useShareIntentContext();
  // ...
}
```

---

## Expo Router Integration

Handle share intents in your layout:

```javascript
// app/_layout.tsx
import { useShareIntent } from "expo-share-intent";
import { useRouter } from "expo-router";

export default function RootLayout() {
  const { hasShareIntent, shareIntent, resetShareIntent } = useShareIntent();
  const router = useRouter();

  useEffect(() => {
    if (hasShareIntent) {
      // Navigate to share handler screen
      router.push({
        pathname: "/share",
        params: { 
          text: shareIntent.text,
          url: shareIntent.webUrl,
        },
      });
      resetShareIntent();
    }
  }, [hasShareIntent]);

  return <Stack />;
}
```

---

## React Navigation Integration

Use custom linking config:

```javascript
import { ShareIntentProvider } from "expo-share-intent";
import { NavigationContainer } from "@react-navigation/native";

const linking = {
  prefixes: ["myapp://"],
  config: {
    screens: {
      Share: "share",
      Home: "",
    },
  },
  // Handle share intent URLs
  getStateFromPath: (path, options) => {
    if (path.includes("shareintent")) {
      return {
        routes: [{ name: "Share" }],
      };
    }
    return undefined;
  },
};

export default function App() {
  return (
    <ShareIntentProvider>
      <NavigationContainer linking={linking}>
        <Navigator />
      </NavigationContainer>
    </ShareIntentProvider>
  );
}
```

---

## Handling Different Content Types

```javascript
function handleShareIntent(shareIntent) {
  // Text/URL
  if (shareIntent.webUrl) {
    return handleSharedUrl(shareIntent.webUrl);
  }
  
  if (shareIntent.text) {
    return handleSharedText(shareIntent.text);
  }
  
  // Files
  if (shareIntent.files?.length) {
    const file = shareIntent.files[0];
    
    if (file.mimeType.startsWith("image/")) {
      return handleSharedImage(file);
    }
    
    if (file.mimeType.startsWith("video/")) {
      return handleSharedVideo(file);
    }
    
    if (file.mimeType.startsWith("audio/")) {
      return handleSharedAudio(file);
    }
    
    return handleSharedFile(file);
  }
}
```

---

## Webpage Metadata (iOS)

When `NSExtensionActivationSupportsWebPageWithMaxCount` is enabled, iOS extracts metadata:

```javascript
const { shareIntent } = useShareIntent();

if (shareIntent.meta) {
  console.log("Page title:", shareIntent.meta.title);
  console.log("OG Image:", shareIntent.meta["og:image"]);
  console.log("Description:", shareIntent.meta["description"]);
}
```

### Custom JavaScript Injection

Extract custom data from web pages:

```json
{
  "plugins": [
    ["expo-share-intent", {
      "iosActivationRules": {
        "NSExtensionActivationSupportsWebPageWithMaxCount": 1
      },
      "preprocessorInjectJS": "metas['custom-data'] = document.querySelector('[data-share-value]')?.textContent"
    }]
  ]
}
```

---

## Build Commands

```bash
# Prebuild (required for native modules)
npx expo prebuild --clean

# Run on iOS
npx expo run:ios

# Run on Android
npx expo run:android
```

Cannot use Expo Go - requires custom dev client.

---

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Config sync failed | Add `xcode+3.0.1.patch` |
| Multiple extension targets in EAS | Manually configure `app.json` and `eas.json` |
| Share intent not received | Check URL scheme matches `app.json` |
| Files not accessible | Check file permissions and copy to app directory |
| Works in dev, not in production | Rebuild with `npx expo prebuild --clean` |
