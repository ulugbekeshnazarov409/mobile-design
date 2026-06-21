# Notifications & Native Capabilities — permission flows + native UI

Native features (notifications, camera, photos, location, share) are where AI apps leak the most: a raw OS prompt fired on launch, no priming, no denied state, and the platform's default UI bolted in unstyled. A native capability is a **flow with states**, not a one-line API call. This file is the UI/UX layer around `expo-*` modules on **Expo SDK 56** (New Architecture default).

> Read `interaction-patterns.md` (every screen is a state machine — "permission denied" is a real state), `real-world-constraints.md` (offline/permission dead-ends), and `frameworks/react-native-expo.md` (install with `npx expo install`, never bare `npm i` for native deps). Location UI lives in `map-ui-patterns.md`.

---

## The universal permission flow (apply to EVERY capability)

The #1 AI tell is requesting permission on app launch, out of context, with the OS dialog as the entire UX. Never do that. The pattern, for notifications / camera / photos / location / mic alike:

1. **In-context, not on launch.** Ask the moment the user reaches for the feature ("Add photo" → then ask), so the value is obvious.
2. **Pre-permission priming screen first.** A *your own* screen explaining the why and the benefit, with "Not now" + "Continue". Only on "Continue" do you call the real `requestPermissionsAsync()`. You get exactly one shot at the OS dialog on iOS — don't waste it cold.
3. **Branch on the real status** (granted / denied / undetermined / limited) — never assume granted.
4. **When denied, the OS will not re-prompt.** Route to system Settings with `Linking.openSettings()` and explain what to toggle.
5. **Always a non-blocking fallback.** Notifications off → still works, just no pushes. Camera off → pick from library or type manually. Never dead-end the user.

```tsx
import * as ImagePicker from 'expo-image-picker';
import { Linking } from 'react-native';

async function ensureCameraThen(open: () => void, showDeniedSheet: () => void) {
  const { status, canAskAgain } = await ImagePicker.getCameraPermissionsAsync();
  if (status === 'granted') return open();
  if (status === 'undetermined' || canAskAgain) {
    // show YOUR priming screen here, then on "Continue":
    const req = await ImagePicker.requestCameraPermissionsAsync();
    return req.granted ? open() : showDeniedSheet();
  }
  showDeniedSheet(); // denied + !canAskAgain → cannot re-prompt
}
// in the denied sheet's CTA:  Linking.openSettings();
```

### Status → UI mapping

| Status | What it means | UI to show |
| --- | --- | --- |
| `undetermined` | never asked | Priming screen → then OS prompt on Continue |
| `granted` | full access | Proceed silently; no celebration needed |
| `limited` (iOS Photos) | user picked *some* photos | Use the selection; offer "Select more" → `presentLimitedLibraryPicker` |
| `denied` (`canAskAgain: true`) | dismissed, can retry | Inline re-ask CTA at point of use |
| `denied` (`canAskAgain: false`) | hard denied | Explainer + **"Open Settings"** (`Linking.openSettings()`) + a working fallback |

> Use `useState` + a focus listener (or re-check `getXPermissionsAsync()` on screen focus) to re-read status after the user returns from Settings — the value can change while your app is backgrounded.

---

## expo-notifications (push + local)

`expo-notifications` push needs a **development build** for remote push on a physical device (Expo Go no longer supports remote push as of SDK 53+). Local notifications work in a dev build; the Android emulator and iOS simulator cannot receive *remote* push. Config plugin in `app.json` for icon/color/sounds:

```json
{ "plugins": [["expo-notifications", {
  "icon": "./assets/notif-icon.png",
  "color": "#0A84FF",
  "sounds": ["./assets/ding.wav"]
}]] }
```

### Handler, request, token

```tsx
import * as Notifications from 'expo-notifications';
import * as Device from 'expo-device';
import { Platform } from 'react-native';

// Set ONCE at module top — controls foreground presentation
Notifications.setNotificationHandler({
  handleNotification: async () => ({
    shouldShowBanner: true,   // SDK 53+ field names
    shouldShowList: true,
    shouldPlaySound: true,
    shouldSetBadge: true,
  }),
});

export async function registerForPush() {
  if (!Device.isDevice) return null; // no push on simulators
  if (Platform.OS === 'android') {
    await Notifications.setNotificationChannelAsync('default', {
      name: 'Default', importance: Notifications.AndroidImportance.HIGH,
      vibrationPattern: [0, 250, 250, 250], lightColor: '#0A84FF',
    });
  }
  const { status } = await Notifications.getPermissionsAsync();
  let final = status;
  if (status !== 'granted') final = (await Notifications.requestPermissionsAsync()).status;
  if (final !== 'granted') return null; // fallback: in-app inbox still works
  const token = (await Notifications.getExpoPushTokenAsync({
    projectId: '<eas-project-id>',
  })).data;
  return token; // send to your backend
}
```

### Tap → deep link, badge, channels

```tsx
// Handle taps (cold start + warm) → route into the app
const last = Notifications.useLastNotificationResponse();
useEffect(() => {
  const screen = last?.notification.request.content.data?.screen;
  if (screen) router.push(screen as string); // expo-router deep link target
}, [last]);

await Notifications.setBadgeCountAsync(unread); // 0 to clear on read
```

- **Android channels are mandatory** for importance/sound/vibration; create them before sending. Per-channel so users can mute categories.
- **In-app vs system banner:** if a push arrives while the user is *already on the relevant screen*, suppress the system banner (`shouldShowBanner: false` conditionally) and show a subtle in-app toast/banner instead — a system banner for the screen you're looking at is noise.
- **In-app banner design:** slides from the top inside the safe area, rounded card, app icon + title + body (2 lines max), auto-dismiss ~4s, swipe-up to dismiss, tap to navigate, light haptic on appear (see `gestures-and-haptics.md`). Don't mimic the OS banner pixel-for-pixel — make it yours.

---

## expo-image-picker & expo-camera (photos)

`expo-image-picker` works in any build (no dev build needed); `expo-camera`'s `CameraView` needs a **development build** for full features. Config plugins declare the iOS usage strings:

```json
{ "plugins": [
  ["expo-image-picker", { "photosPermission": "Pick a profile photo." }],
  ["expo-camera", { "cameraPermission": "Scan and capture receipts." }]
]}
```

```tsx
import * as ImagePicker from 'expo-image-picker';
import { Image } from 'expo-image'; // not RN <Image> — better caching + transitions

const res = await ImagePicker.launchImageLibraryAsync({
  mediaTypes: ['images'],     // SDK 52+ string array, not MediaTypeOptions
  allowsEditing: true,        // square crop for avatars
  quality: 0.7,               // compress — never upload originals
  selectionLimit: 1,
});
if (!res.canceled) setUri(res.assets[0].uri);
```

```tsx
// Selected-image UI: show immediately, with upload state overlaid
<View>
  <Image source={uri} style={{ width: 96, height: 96, borderRadius: 48 }}
         contentFit="cover" transition={200} placeholder={blurhash} />
  {uploading && <ActivityIndicator style={StyleSheet.absoluteFill} />}
  <Pressable onPress={remove} style={styles.removeBadge}><X /></Pressable>
</View>
```

- **Show the local URI instantly** (optimistic), then upload with a determinate progress ring (0–100%) — not an endless spinner. Surface retry on failure.
- **Camera preview UI:** request via `useCameraPermissions()`; render `<CameraView>` full-bleed with controls (shutter, flip, flash) floating inside safe areas — same chrome discipline as `map-ui-patterns.md`. Show a styled "Enable camera" prompt for denied, never a blank black view.

---

## Location (brief — UI lives elsewhere)

`expo-location` config plugin sets iOS usage strings + Android background flag. Request **foreground** in context; handle precise vs approximate (`accuracy`, iOS 14+ reduced accuracy) and denied with a manual-search fallback. Full map/marker/permission-banner UI → **`map-ui-patterns.md`**.

```tsx
import * as Location from 'expo-location';
const { status } = await Location.requestForegroundPermissionsAsync();
if (status !== 'granted') return showManualSearch(); // never dead-end
const pos = await Location.getCurrentPositionAsync({ accuracy: Location.Accuracy.Balanced });
```

---

## Share & clipboard (instant native wins)

Share needs **no permission** — the cheapest "native" feeling. Use RN's `Share` for text/links; `expo-sharing` (`Sharing.shareAsync(uri)`) for files (after checking `isAvailableAsync()`).

```tsx
import { Share } from 'react-native';
await Share.share({ message: 'Check this out', url: 'https://app.example/x' });
```

```tsx
import * as Clipboard from 'expo-clipboard';
import * as Haptics from 'expo-haptics';

async function copy(value: string) {
  await Clipboard.setStringAsync(value);
  await Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
  setCopied(true);                       // swap icon → checkmark
  setTimeout(() => setCopied(false), 1500);
}
```

- **Copy is never silent.** Haptic + a momentary "Copied" label or icon swap is mandatory — without feedback the user taps twice.
- For secrets/tokens use **`expo-secure-store`** (`setItemAsync`/`getItemAsync`, Keychain/Keystore-backed) — never `AsyncStorage` for credentials. SecureStore values cap ~2KB.

---

## Deep linking & universal links

Deep links let notifications, share sheets, and the web open your app to a specific screen. With **expo-router**, the file path *is* the URL — minimal config. Set the scheme in `app.json`:

```json
{ "scheme": "myapp",
  "ios": { "associatedDomains": ["applinks:app.example.com"] },
  "android": { "intentFilters": [{ "action": "VIEW", "autoVerify": true,
    "data": [{ "scheme": "https", "host": "app.example.com" }],
    "category": ["BROWSABLE", "DEFAULT"] }] } }
```

- `myapp://order/42` or `https://app.example.com/order/42` → `app/order/[id].tsx`. Universal/App Links (the `https` ones) need the verified domain files (`apple-app-site-association`, `assetlinks.json`) hosted on the domain.
- Use `Linking.createURL('/path')` to build links; `Linking.openSettings()` to bounce to OS settings; `Linking.openURL()` for external. Notification `data.screen` → `router.push(screen)` (see above). See `frameworks/react-native-expo.md` for the expo-router layout backbone.

---

## Dev-build & plugin cheat sheet

| Module | Needs dev build? | app.json config |
| --- | --- | --- |
| `expo-notifications` (remote push) | **Yes** (physical device) | plugin: icon/color/sounds; EAS projectId for token |
| `expo-notifications` (local) | No (works in dev build) | same plugin |
| `expo-image-picker` | No | plugin: photos/camera permission strings |
| `expo-camera` (`CameraView`) | **Yes** for full features | plugin: cameraPermission |
| `expo-location` (foreground) | No | plugin: location permission strings |
| `Share` (RN) / `expo-sharing` | No | none / none |
| `expo-haptics` | No | none |
| `expo-clipboard` | No | none |
| `expo-secure-store` | No | plugin (optional Face ID prompt) |
| `expo-linking` / universal links | No (links); domain files for App Links | scheme + associatedDomains + intentFilters |

Run `npx expo install <module>` (matches SDK 56), then `npx expo prebuild` / rebuild the dev client after adding a config plugin — a JS reload won't pick up native changes.

---

## Quality checklist (native + permissions)

- [ ] No permission requested on app launch — every ask is **in context**, at the moment the feature is reached.
- [ ] A **priming screen** explains the why before the OS dialog fires (you get one shot on iOS).
- [ ] All four states handled: granted / undetermined / denied / limited (iOS Photos).
- [ ] Hard-denied path routes to `Linking.openSettings()` and re-checks status on return.
- [ ] Every capability has a **non-blocking fallback** — no dead-ends when permission is off.
- [ ] Notifications: handler set once, Android channels created, badge cleared on read, tap deep-links via expo-router.
- [ ] System banner suppressed when the user is already on the relevant screen; in-app banner is custom, not an OS clone.
- [ ] Picked images show instantly (optimistic) with determinate upload progress + retry; use `expo-image`.
- [ ] Camera/denied states are styled prompts, never a blank black view.
- [ ] Copy gives haptic + "Copied" feedback; secrets in `expo-secure-store`, not AsyncStorage.
- [ ] Native deps installed via `npx expo install`; config plugins + dev-build rebuilt where required.
