# Voltra Reference

Build Live Activities, Dynamic Island, and widgets using React components.

## Installation

```bash
npm install voltra
```

Add to `app.json`:

```json
{
  "plugins": ["voltra"]
}
```

Run prebuild:

```bash
npx expo prebuild -p ios --clean
```

---

## Components

### Layout Components

#### VStack

Vertical stack layout.

```jsx
<Voltra.VStack style={{ spacing: 8, padding: 16 }}>
  <Voltra.Text>Item 1</Voltra.Text>
  <Voltra.Text>Item 2</Voltra.Text>
</Voltra.VStack>
```

#### HStack

Horizontal stack layout.

```jsx
<Voltra.HStack style={{ spacing: 12, alignment: 'center' }}>
  <Voltra.Symbol name="star.fill" />
  <Voltra.Text>Rating</Voltra.Text>
</Voltra.HStack>
```

#### ZStack

Overlay/z-axis stack.

```jsx
<Voltra.ZStack>
  <Voltra.Image source={{ uri: backgroundUrl }} />
  <Voltra.Text>Overlay Text</Voltra.Text>
</Voltra.ZStack>
```

#### Spacer

Flexible space that expands.

```jsx
<Voltra.HStack>
  <Voltra.Text>Left</Voltra.Text>
  <Voltra.Spacer />
  <Voltra.Text>Right</Voltra.Text>
</Voltra.HStack>
```

---

### Visual Components

#### Text

```jsx
<Voltra.Text
  style={{
    color: '#F8FAFC',
    fontSize: 18,
    fontWeight: '600',
    lineLimit: 2,
  }}
>
  Hello World
</Voltra.Text>
```

#### Symbol (SF Symbols)

```jsx
<Voltra.Symbol
  name="car.fill"
  type="hierarchical"  // monochrome, hierarchical, palette, multicolor
  scale="large"        // small, medium, large
  tintColor="#38BDF8"
/>
```

#### Image

```jsx
<Voltra.Image
  source={{ uri: 'https://example.com/image.png' }}
  style={{ width: 40, height: 40, borderRadius: 8 }}
  resizeMode="cover"
/>
```

#### ProgressView

```jsx
<Voltra.ProgressView
  value={0.65}
  style={{ tintColor: '#10B981' }}
/>
```

#### Gauge

```jsx
<Voltra.Gauge
  value={0.75}
  label="Progress"
  currentValueLabel="75%"
/>
```

---

### Interactive Components

#### Button

```jsx
<Voltra.Button
  id="action-button"
  style={{ backgroundColor: '#3B82F6', padding: 12, borderRadius: 8 }}
>
  <Voltra.Text style={{ color: 'white' }}>Tap Me</Voltra.Text>
</Voltra.Button>
```

#### Link

```jsx
<Voltra.Link url="myapp://action/start">
  <Voltra.Text style={{ color: '#3B82F6' }}>Open in App</Voltra.Text>
</Voltra.Link>
```

---

## Live Activity API

### Start Live Activity

```javascript
import { startLiveActivity } from 'voltra/client';
import { Voltra } from 'voltra';

const activityId = await startLiveActivity({
  // Required: Lock screen view
  lockScreen: (
    <Voltra.VStack style={{ padding: 16 }}>
      <Voltra.Text>Lock Screen Content</Voltra.Text>
    </Voltra.VStack>
  ),
  
  // Optional: Dynamic Island views
  dynamicIsland: {
    // Compact leading (left side when minimized)
    compactLeading: <Voltra.Symbol name="timer" />,
    
    // Compact trailing (right side when minimized)
    compactTrailing: <Voltra.Text>2:34</Voltra.Text>,
    
    // Minimal (when multiple activities, shows one)
    minimal: <Voltra.Symbol name="timer" />,
    
    // Expanded (when long-pressed)
    expanded: (
      <Voltra.VStack>
        <Voltra.Text>Expanded View</Voltra.Text>
      </Voltra.VStack>
    ),
  },
});
```

### Update Live Activity

```javascript
import { updateLiveActivity } from 'voltra/client';

await updateLiveActivity(activityId, {
  lockScreen: (
    <Voltra.VStack style={{ padding: 16 }}>
      <Voltra.Text>Updated Content</Voltra.Text>
    </Voltra.VStack>
  ),
});
```

### End Live Activity

```javascript
import { endLiveActivity } from 'voltra/client';

await endLiveActivity(activityId, {
  // Optional: Final content to show
  lockScreen: (
    <Voltra.VStack>
      <Voltra.Text>Completed!</Voltra.Text>
    </Voltra.VStack>
  ),
  // How long to keep on lock screen after ending
  dismissalPolicy: 'default', // 'default', 'immediate', or date
});
```

---

## useLiveActivity Hook

Provides hot reload support during development.

```javascript
import { useLiveActivity } from 'voltra/client';

function MyComponent() {
  const { 
    start,      // Start activity
    update,     // Update activity
    end,        // End activity
    activityId, // Current activity ID
    isActive,   // Whether activity is running
  } = useLiveActivity();

  const handleStart = async () => {
    await start({
      lockScreen: <MyLockScreenUI elapsed={0} />,
      dynamicIsland: {
        compactLeading: <Voltra.Symbol name="record.circle" />,
        compactTrailing: <Voltra.Text>00:00</Voltra.Text>,
        expanded: <MyExpandedUI />,
      },
    });
  };

  const handleUpdate = async (elapsed: number) => {
    await update({
      lockScreen: <MyLockScreenUI elapsed={elapsed} />,
      dynamicIsland: {
        compactTrailing: <Voltra.Text>{formatTime(elapsed)}</Voltra.Text>,
      },
    });
  };

  const handleEnd = async () => {
    await end({
      lockScreen: <MyCompletedUI />,
      dismissalPolicy: 'default',
    });
  };

  return (
    <View>
      {!isActive ? (
        <Button title="Start" onPress={handleStart} />
      ) : (
        <Button title="Stop" onPress={handleEnd} />
      )}
    </View>
  );
}
```

---

## Server-Side Rendering (Push Updates)

Update Live Activities from your server via APNS.

### Render Payload

```javascript
import { renderLiveActivityToString } from 'voltra/server';
import { Voltra } from 'voltra';

const payload = renderLiveActivityToString({
  lockScreen: (
    <Voltra.VStack style={{ padding: 16, backgroundColor: '#101828' }}>
      <Voltra.HStack style={{ spacing: 8 }}>
        <Voltra.Symbol name="car.fill" tintColor="#38BDF8" />
        <Voltra.Text style={{ color: '#F8FAFC', fontSize: 18 }}>
          Driver arrived
        </Voltra.Text>
      </Voltra.HStack>
      <Voltra.Text style={{ color: '#94A3B8', fontSize: 12, marginTop: 8 }}>
        Ready for pickup at Building A
      </Voltra.Text>
    </Voltra.VStack>
  ),
});

// payload is JSON string ready for APNS
```

### Send via APNS

```javascript
// Example using apns2 library
import apn from 'apn';

const notification = new apn.Notification({
  topic: 'com.myapp.push-type.liveactivity',
  pushType: 'liveactivity',
  payload: {
    'content-state': JSON.parse(payload),
    'timestamp': Date.now(),
    'event': 'update', // 'update' or 'end'
  },
});

await apnProvider.send(notification, pushToken);
```

---

## Widgets

### Basic Widget

```javascript
// widgets/MyWidget.tsx
import { Voltra } from 'voltra';

export function MyWidget({ data }) {
  return (
    <Voltra.VStack style={{ padding: 16, backgroundColor: '#1a1a1a' }}>
      <Voltra.Text style={{ fontSize: 24, fontWeight: '700', color: 'white' }}>
        {data.title}
      </Voltra.Text>
      <Voltra.Spacer />
      <Voltra.Text style={{ fontSize: 14, color: '#9CA3AF' }}>
        {data.subtitle}
      </Voltra.Text>
    </Voltra.VStack>
  );
}
```

### Widget Sizes

Handle different widget sizes:

```javascript
export function MyWidget({ data, family }) {
  if (family === 'small') {
    return <SmallWidget data={data} />;
  }
  if (family === 'medium') {
    return <MediumWidget data={data} />;
  }
  return <LargeWidget data={data} />;
}
```

---

## Styling

### Style Properties

Common style properties for Voltra components:

```javascript
const style = {
  // Layout
  padding: 16,
  paddingHorizontal: 12,
  paddingVertical: 8,
  margin: 4,
  
  // Size
  width: 100,
  height: 50,
  minWidth: 40,
  maxWidth: 200,
  
  // Background
  backgroundColor: '#101828',
  
  // Border
  borderRadius: 12,
  borderWidth: 1,
  borderColor: '#374151',
  
  // Text-specific
  color: '#F8FAFC',
  fontSize: 18,
  fontWeight: '600',
  lineLimit: 2,
  
  // Stack-specific
  spacing: 8,
  alignment: 'center', // leading, center, trailing
};
```

---

## Events and Interactions

### Button Events

Handle button taps via deep links:

```javascript
// In Live Activity
<Voltra.Button id="pause-timer">
  <Voltra.Text>Pause</Voltra.Text>
</Voltra.Button>

// In your app, handle the deep link
// URL: myapp://voltra/action?id=pause-timer
```

### Link Navigation

```javascript
<Voltra.Link url="myapp://screen/details?id=123">
  <Voltra.Text>View Details</Voltra.Text>
</Voltra.Link>
```

---

## Plugin Configuration

In `app.json`:

```json
{
  "plugins": [
    ["voltra", {
      // Enable debug logging
      "debug": true,
      
      // Widget configuration
      "widgets": {
        "MyWidget": {
          "kind": "MyWidget",
          "displayName": "My Widget",
          "description": "Shows important info",
          "supportedFamilies": ["small", "medium", "large"]
        }
      }
    }]
  ]
}
```
