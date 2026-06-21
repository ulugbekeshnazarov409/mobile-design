# React Native / Expo — project-aware, design-grade

Target: premium RN/Expo UI that **respects the project's existing stack**. RN has no single styling system, so the first job is detection, not generation. Never impose StyleSheet on a NativeWind repo, or NativeWind on a Tamagui repo.

> Read `project-awareness.md` first for the full pre-flight, and `frameworks/react-native-ui-libraries.md` for the full styling-library catalog. This file is the RN/Expo platform + design playbook.
>
> **Current SDK: Expo 56** (June 2026). New Architecture (Fabric/TurboModules) is the default. Use `npx expo install` for every native dep so versions match the SDK — never bare `npm i` for native modules.

---

## Step 0 — detect the styling system (do this before writing UI)

Check `package.json` + config files and match what's there:

| If the repo has… | Style with… | Rule |
| --- | --- | --- |
| `nativewind` + `tailwind.config.js` + `global.css` | **className only** (Tailwind) | NEVER hand-write `StyleSheet`. Theme via `tailwind.config` + CSS vars / `dark:` variants. Match the `nativewind-ui` skill discipline. |
| `tamagui` / `@tamagui/*` | Tamagui `styled()` + tokens | `<YStack>/<XStack>/<Text>`, theme tokens, `$` props. |
| `@shopify/restyle` | restyle `createBox`/`createText` + theme | Theme-typed components. |
| `dripsy` | Dripsy `sx` + theme | `sx` props. |
| `react-native-unistyles` | `StyleSheet.create` from unistyles + themes | Themed StyleSheet, `useStyles`. |
| `react-native-paper` / `@rneui/*` / `@ui-kitten` / `@gluestack-ui` | that component library + its theme | One library, used to the end. |
| none of the above | `StyleSheet.create` + a token module | Centralize tokens; no inline magic numbers. |

If unsure which is active, grep for `className=` (NativeWind), `styled(` (tamagui), `createBox` (restyle), `useStyles` (unistyles) — go with what existing screens use. **Consistency with the codebase beats personal preference. One system per app, used to the end — never mix.**

### NativeWind (className-only)
```tsx
// tokens live in tailwind.config.js (colors.primary, spacing…) — reference them, never raw hex
export function PrimaryButton({ title, onPress }: { title: string; onPress: () => void }) {
  return (
    <Pressable
      onPress={onPress}
      className="h-13 rounded-xl bg-primary items-center justify-center active:opacity-85"
      android_ripple={{ color: 'rgba(255,255,255,0.2)' }}>
      <Text className="text-base font-semibold text-on-primary">{title}</Text>
    </Pressable>
  );
}
```
Dark mode: `dark:` variants + `useColorScheme()` from `nativewind`. Don't fork into StyleSheet.

### Plain StyleSheet (token-backed)
```tsx
import { tokens as t } from '@/theme'; // single source of truth, light+dark maps
const styles = StyleSheet.create({
  btn: { height: 52, borderRadius: 12, backgroundColor: t.primary, alignItems: 'center', justifyContent: 'center' },
  btnText: { color: t.onPrimary, fontSize: 15, fontWeight: '600' },
});
```

---

## Expo Router — file-based navigation (the layout backbone)

Routes are files in `app/`. A `_layout.tsx` defines the navigator for its folder. This *is* your screen hierarchy — design the file tree and the navigators together.

### Project shape (typical tabbed app)
```
app/
  _layout.tsx            # root Stack — fonts/splash gate, providers, theme
  (tabs)/
    _layout.tsx          # bottom Tabs
    index.tsx            # Home tab
    search.tsx           # Search tab
    profile.tsx          # Profile tab
  (auth)/
    _layout.tsx          # auth Stack (logged-out group)
    sign-in.tsx
  product/[id].tsx       # dynamic detail route
  (modal)/
    filter.tsx           # presented modally
```
- **Folder in `(parens)`** = a *route group*: organizes files / applies a shared layout **without** adding a URL segment. Use for `(tabs)`, `(auth)`, `(modal)`.
- **`[id].tsx`** = dynamic route; **`[...rest].tsx`** = catch-all.
- Enable **typed routes** in `app.json` (`experiments.typedRoutes: true`) for autocompleted, type-checked `href`s.

### Root layout — splash/font gate + providers
```tsx
// app/_layout.tsx
import { Stack } from 'expo-router';
import { useFonts } from 'expo-font';
import * as SplashScreen from 'expo-splash-screen';
import { SafeAreaProvider } from 'react-native-safe-area-context';
import { GestureHandlerRootView } from 'react-native-gesture-handler';
import { ThemeProvider, DarkTheme, DefaultTheme } from '@react-navigation/native';
import { useColorScheme } from 'react-native';
import { useEffect } from 'react';

SplashScreen.preventAutoHideAsync();

export default function RootLayout() {
  const scheme = useColorScheme();
  const [loaded] = useFonts({ Inter: require('../assets/fonts/Inter.ttf') });
  useEffect(() => { if (loaded) SplashScreen.hideAsync(); }, [loaded]);
  if (!loaded) return null;                       // keep splash up until fonts ready (no FOUT)

  return (
    <GestureHandlerRootView style={{ flex: 1 }}>
      <SafeAreaProvider>
        <ThemeProvider value={scheme === 'dark' ? DarkTheme : DefaultTheme}>
          <Stack>
            <Stack.Screen name="(tabs)" options={{ headerShown: false }} />
            <Stack.Screen name="(modal)/filter" options={{ presentation: 'modal' }} />
          </Stack>
        </ThemeProvider>
      </SafeAreaProvider>
    </GestureHandlerRootView>
  );
}
```
`GestureHandlerRootView` must wrap the whole app once (gestures + bottom sheets need it). `SafeAreaProvider` enables `useSafeAreaInsets()` everywhere.

### Tabs layout — premium bottom nav
```tsx
// app/(tabs)/_layout.tsx
import { Tabs } from 'expo-router';
import { Ionicons } from '@expo/vector-icons';
import { useColorScheme } from 'react-native';

export default function TabsLayout() {
  const scheme = useColorScheme();
  return (
    <Tabs
      screenOptions={{
        tabBarActiveTintColor: '#6E56CF',
        headerShown: false,
        tabBarStyle: { borderTopWidth: 0.5, height: 84, paddingTop: 6 },  // let safe-area pad the bottom
      }}>
      <Tabs.Screen name="index"   options={{ title: 'Home',    tabBarIcon: ({ color, size }) => <Ionicons name="home" color={color} size={size} /> }} />
      <Tabs.Screen name="search"  options={{ title: 'Search',  tabBarIcon: ({ color, size }) => <Ionicons name="search" color={color} size={size} /> }} />
      <Tabs.Screen name="profile" options={{ title: 'Profile', tabBarIcon: ({ color, size }) => <Ionicons name="person" color={color} size={size} /> }} />
    </Tabs>
  );
}
```
3–5 tabs max; selected = filled icon + accent + label. For an iOS-26 glass tab bar, give `tabBarBackground` a `BlurView` (see `glassmorphism-and-materials.md`).

### Navigating
```tsx
import { Link, useRouter, useLocalSearchParams } from 'expo-router';

<Link href="/product/42">Open</Link>
<Link href={{ pathname: '/product/[id]', params: { id: '42' } }} asChild><Pressable>…</Pressable></Link>

const router = useRouter();
router.push('/product/42');     // add to stack
router.replace('/(tabs)');      // swap (post-login — no back to auth)
router.back();                  // pop
router.dismiss();               // close modal stack

// in app/product/[id].tsx
const { id } = useLocalSearchParams<{ id: string }>();
```
- **`push`** vs **`replace`**: after sign-in, `replace` into `(tabs)` so Back doesn't return to the login screen.
- **Auth gating:** `<Redirect href="/(auth)/sign-in" />` from a protected layout when no session.
- **Modals:** a screen in a `presentation: 'modal'` group; close with `router.back()`. iOS gets the card sheet automatically.
- **`unstable_settings.initialRouteName`** in a layout fixes deep-link back behavior (so a deep link still has a parent to pop to).

---

## Safe areas & edge-to-edge

Expo apps render **edge-to-edge** (content behind status/nav bars) — handle insets explicitly or content hides under the notch / home indicator.

```tsx
import { useSafeAreaInsets } from 'react-native-safe-area-context';

const insets = useSafeAreaInsets();
// Pin a bottom CTA above the home indicator:
<View style={{ paddingBottom: insets.bottom + 12 }}>…</View>
// Prefer padding from insets over <SafeAreaView> when a header/scroll already consumes the top.
```
- Use `useSafeAreaInsets()` (padding) for scroll containers + pinned bars; `<SafeAreaView edges={['top']}>` for simple full-screen content.
- `expo-status-bar`: `<StatusBar style="auto" />` — flips light/dark with the theme; set `style="light"` over dark hero images.
- Android edge-to-edge is on by default in recent SDKs — never hardcode a status-bar height; read insets.

---

## Images, icons, fonts, splash

| Concern | Use | Why |
| --- | --- | --- |
| Images | `expo-image` `<Image>` | Caching, `contentFit`, `placeholder={blurhash}`, `transition` fade — premium load feel. Avoid RN core `Image` for content. |
| Icons | `@expo/vector-icons` (Ionicons ≈ iOS, MaterialIcons ≈ Android) or **`expo-symbols`** (`<SymbolView>`) for true SF Symbols on iOS | One icon set per app; match stroke weight to text. |
| Fonts | `expo-font` `useFonts` | Gate render until loaded (see root layout) — kills the flash of system font. |
| Splash | `expo-splash-screen` | `preventAutoHideAsync()` → `hideAsync()` after fonts/data ready. Configure icon/color in `app.json` `expo-splash-screen` plugin. |

```tsx
import { Image } from 'expo-image';
<Image
  source={uri}
  placeholder={{ blurhash }}
  contentFit="cover"
  transition={200}
  style={{ width: '100%', aspectRatio: 16 / 9, borderRadius: 16 }}
/>
```

---

## Lists & performance (where RN UIs get janky)

- **Long lists:** `@shopify/flash-list` (`<FlashList>`) or `FlatList` — never `.map()` in a `ScrollView` for real data. Provide `keyExtractor`, `ItemSeparatorComponent`, `ListEmptyComponent`, `ListHeaderComponent`, `contentContainerStyle`. FlashList wants a stable `estimatedItemSize`.
- **Memoize rows:** `React.memo` the item; stable `renderItem`/`keyExtractor` (no inline closures recreated each render).
- **Images in lists:** `expo-image` with `recyclingKey` and fixed dimensions — unsized images cause layout thrash.
- **New Architecture (default in SDK 56):** Fabric renderer + TurboModules; most libs support it, but verify a native lib lists New-Arch support before adding.
- **Don't animate layout on the JS thread:** drive motion through Reanimated worklets (UI thread) — see below.

---

## Motion & gestures (Reanimated 3 + Gesture Handler)

```tsx
import Animated, { useSharedValue, useAnimatedStyle, withSpring, withTiming, Easing, FadeIn, SlideInDown } from 'react-native-reanimated';
import { Gesture, GestureDetector } from 'react-native-gesture-handler';

// Press scale (tactile feedback)
const scale = useSharedValue(1);
const style = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }));
const tap = Gesture.Tap().onBegin(() => (scale.value = withSpring(0.96))).onFinalize(() => (scale.value = withSpring(1)));
<GestureDetector gesture={tap}><Animated.View style={style}>…</Animated.View></GestureDetector>

// Mount/exit transitions on list items
<Animated.View entering={FadeIn.duration(200)} exiting={SlideInDown}>…</Animated.View>
```
- Timing 150–300ms, `Easing.out(Easing.cubic)` for entrances; springs for interactive/dragged elements. **Every enter has an exit.**
- Honor Reduce Motion: `AccessibilityInfo.isReduceMotionEnabled()` → swap to instant/opacity. See `motion-recipes.md` + `gestures-and-haptics.md`.

### Haptics (commit feedback)
```tsx
import * as Haptics from 'expo-haptics';
Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);          // press / toggle
Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success); // success / error
```
Map haptics by action (light=tap, medium=commit, success/error=outcomes); never fire on every render. iOS feels these strongly; Android varies — don't depend on them for meaning.

---

## Material & depth helpers

- **Blur / glass:** `expo-blur` `<BlurView intensity tint />` for floating bars/sheets — disciplined, with a solid fallback (Android blur is weaker). Full rules in `glassmorphism-and-materials.md`.
- **Gradients:** `expo-linear-gradient` `<LinearGradient colors={[…]} />` — accent/scrim use, ≤ subtle; not stock gradients on every card.
- **Bottom sheets:** `@gorhom/bottom-sheet` (needs reanimated + gesture-handler + `GestureHandlerRootView`). `snapPoints`, `BottomSheetBackdrop`, dynamic sizing. The right tool for sheets over maps/content.
- **Shadows:** iOS uses `shadowColor/Opacity/Radius/Offset`; Android uses `elevation`. `Platform.select` or a token that maps both — soft layered shadows, not `#000` 0.5.

---

## Keyboard, inputs, platform polish

- **Keyboard:** `KeyboardAvoidingView` (`behavior={Platform.OS === 'ios' ? 'padding' : 'height'}`) for forms; **`react-native-keyboard-controller`** for chat inputs / sticky composer (smoother, interactive dismiss).
- **Pressable, not TouchableOpacity** for new code: `android_ripple={{ color }}` + `style={({ pressed }) => …}`; hit area ≥ 48dp via `hitSlop` or min sizes.
- **Per-OS:** `Platform.select({ ios, android })` for the real differences — iOS: Ionicons, large titles (`headerLargeTitle`), sheet detents, no FAB, subtle shadows; Android: ripple, FAB for primary create, elevation, Material icons. Adapt the *soul* (`platform-personality.md`), don't force one look.

---

## EAS & app config (what affects shipping the design)

- **app.json / app.config.js** — app metadata + **config plugins** (native capability without ejecting). Splash, icon, fonts, permissions, status-bar all configured here.
- **Development build** (`eas build --profile development`) vs **Expo Go**: any custom native module (maps, some blur/haptics edge cases) needs a **dev build**, not Expo Go. Tell the user when a chosen package forces this.
- **EAS Update** (`eas update`) — OTA push JS/asset changes (incl. UI tweaks) without a store release; pairs with `expo-updates`.
- **`npx expo prebuild`** — generates native `ios/`/`android/` when manual native config is needed; otherwise stay managed.
- `eas build` / `eas submit` build + ship to the stores from the cloud.

> Design impact: if a screen needs a native module (e.g. maps, advanced blur), say so up front — it implies a dev build, not Expo Go, and possibly a config plugin in `app.json`.

---

## Add-a-package policy (only on a real gap, the Expo way)

| Need | Package | Install |
| --- | --- | --- |
| Bottom sheet | `@gorhom/bottom-sheet` (+ reanimated, gesture-handler) | `npx expo install @gorhom/bottom-sheet react-native-reanimated react-native-gesture-handler` |
| Animations | `react-native-reanimated` | `npx expo install react-native-reanimated` |
| Gestures | `react-native-gesture-handler` | `npx expo install react-native-gesture-handler` |
| Fast images | `expo-image` | `npx expo install expo-image` |
| Safe area | `react-native-safe-area-context` | `npx expo install react-native-safe-area-context` |
| Blur / glass | `expo-blur` | `npx expo install expo-blur` |
| Gradients | `expo-linear-gradient` | `npx expo install expo-linear-gradient` |
| Haptics | `expo-haptics` | `npx expo install expo-haptics` |
| SF Symbols (iOS) | `expo-symbols` | `npx expo install expo-symbols` |
| Big lists | `@shopify/flash-list` | `npx expo install @shopify/flash-list` |
| Keyboard (chat) | `react-native-keyboard-controller` | `npx expo install react-native-keyboard-controller` |
| Maps | `react-native-maps` (or Yandex SDK via dev build) | `npx expo install react-native-maps` |
| Tailwind styling | `nativewind` + `tailwindcss` | follow `expo-tailwind-setup` / `nativewind-ui` skill |

Prefer existing deps. Always `npx expo install` (matches SDK versions). Mention any required `babel.config.js` (reanimated plugin) / `metro.config.js` / `app.json` plugin changes — and whether it forces a **dev build**.

---

## Anti-slop checklist (RN/Expo)

- [ ] Styling matches the repo's system (NativeWind className-only / Tamagui / restyle / unistyles / token-backed StyleSheet) — no mixing.
- [ ] No inline hex / magic numbers; tokens or Tailwind config drive everything; light + dark both shipped.
- [ ] Navigation via Expo Router file tree; groups for tabs/auth/modals; `replace` (not `push`) after login; typed routes on.
- [ ] Safe-area insets honored (edge-to-edge); bottom CTAs above `insets.bottom`; `expo-status-bar` style matches background.
- [ ] Fonts/splash gated (no FOUT); `expo-image` for content with blurhash + `transition`; one icon set.
- [ ] Long lists use FlashList/FlatList with `keyExtractor` + empty/loading states; rows memoized.
- [ ] Motion via Reanimated worklets (UI thread); press scale + `expo-haptics` on commits; every enter has an exit; Reduce Motion handled.
- [ ] `Pressable` with ripple + ≥48dp targets; keyboard handled for forms/chat.
- [ ] New deps via `npx expo install`; required plugins/config noted; dev-build requirement flagged when a native module is used.
