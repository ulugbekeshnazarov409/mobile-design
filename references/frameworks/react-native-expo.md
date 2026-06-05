# React Native / Expo — project-aware

Target: premium RN/Expo UI that **respects the project's existing stack**. RN has no single styling system, so the first job is detection, not generation. Never impose StyleSheet on a NativeWind repo, or NativeWind on a Tamagui repo.

> Read `project-awareness.md` first for the full pre-flight. This file is RN-specific.

## Step 0 — detect the styling system (do this before writing UI)

Check `package.json` + config files and match what's there:

| If the repo has… | Style with… | Rule |
| --- | --- | --- |
| `nativewind` + `tailwind.config.js` + `global.css` | **className only** (Tailwind) | NEVER hand-write `StyleSheet`. Theme via `tailwind.config` + CSS vars / `dark:` variants. Match the `nativewind-ui` skill discipline. |
| `tamagui` / `@tamagui/*` | Tamagui `styled()` + tokens | Use `<YStack>/<XStack>/<Text>`, theme tokens, `$` props. |
| `@shopify/restyle` | restyle `createBox`/`createText` + theme | Use theme-typed components. |
| `dripsy` | Dripsy `sx` + theme | Use `sx` props. |
| none of the above | `StyleSheet.create` + a token module | Centralize tokens; no inline magic numbers. |

If unsure which is active, grep for `className=` (NativeWind), `styled(` from tamagui, `createBox` (restyle) — go with what the existing screens use. **Consistency with the codebase beats personal preference.**

### NativeWind example (className-only)
```tsx
// tokens live in tailwind.config.js (colors.primary, spacing, etc.) — reference them
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

### Plain StyleSheet example (token-backed)
```tsx
import { tokens as t } from '@/theme'; // single source of truth, light+dark maps
const styles = StyleSheet.create({
  btn: { height: 52, borderRadius: 12, backgroundColor: t.primary, alignItems: 'center', justifyContent: 'center' },
  btnText: { color: t.onPrimary, fontSize: 15, fontWeight: '600' },
});
```

## Expo essentials

- **Routing:** Expo Router (file-based). Tab scaffold = `app/(tabs)/_layout.tsx` with `<Tabs>`; stacks via `<Stack>`; modals via `app/(modal)/`. `<Stack.Screen options={{ headerLargeTitle: true }} />` for iOS large titles.
- **Safe area:** `react-native-safe-area-context` — `useSafeAreaInsets()` or `<SafeAreaView>`; pin bottom CTAs above `insets.bottom`.
- **Theme/dark mode:** `useColorScheme()` (from `nativewind` if present, else `react-native`); drive a token object or Tailwind `dark:`.
- **Images:** `expo-image` (`<Image>` with `contentFit`, `placeholder`, `transition`) over RN `Image` for caching + blurhash placeholders.
- **Icons:** `@expo/vector-icons` (Ionicons ≈ iOS feel, MaterialIcons ≈ Android) — keep one set.
- **Fonts:** `expo-font` + `useFonts`; gate render until loaded.
- **Haptics:** `expo-haptics` on key interactions.

## Packages to add when a capability is missing

Prefer existing deps. When genuinely needed, propose the standard choice + install, then wire minimally:

| Need | Package | Install |
| --- | --- | --- |
| Bottom sheet | `@gorhom/bottom-sheet` (+ reanimated, gesture-handler) | `npx expo install @gorhom/bottom-sheet react-native-reanimated react-native-gesture-handler` |
| Animations | `react-native-reanimated` | `npx expo install react-native-reanimated` |
| Gestures | `react-native-gesture-handler` | `npx expo install react-native-gesture-handler` |
| Fast images | `expo-image` | `npx expo install expo-image` |
| Safe area | `react-native-safe-area-context` | `npx expo install react-native-safe-area-context` |
| Tailwind styling | `nativewind` + `tailwindcss` | follow `expo-tailwind-setup` / `nativewind-ui` skill |

Always use `npx expo install` (not bare `npm i`) in Expo projects so versions match the SDK. Mention any required `babel.config.js` / `metro.config.js` / plugin changes.

## Layout idioms

- `FlatList`/`FlashList` (Shopify) for long lists — never `.map()` in a `ScrollView` for big data. Use `ItemSeparatorComponent`, `keyExtractor`, `ListEmptyComponent`, `contentContainerStyle`.
- `Pressable` (not `TouchableOpacity`) for new code: `android_ripple` + `style={({pressed}) => …}`. Hit area ≥ 48dp via `hitSlop` or min sizes.
- Keyboard: `KeyboardAvoidingView` (`behavior={Platform.OS==='ios'?'padding':'height'}`) or `react-native-keyboard-controller` for chat inputs.
- Per-OS: `Platform.select({ ios, android })` for the small differences (shadow vs elevation, FAB only on Android, fonts).

## Animation

- **Reanimated 3:** `useSharedValue`, `useAnimatedStyle`, `withSpring`, `withTiming(v, { easing: Easing.out(Easing.cubic) })`, `Layout`/entering/exiting (`FadeIn`, `SlideInDown`) for list + mount transitions.
- Gestures via `react-native-gesture-handler` (`Gesture.Pan()` + `GestureDetector`).
- Keep durations 150–300ms; springs for interactive feel. Honor `AccessibilityInfo.isReduceMotionEnabled`.

## Per-OS polish

- iOS feel: Ionicons, large titles, sheet detents (gorhom `snapPoints`), no FAB, subtle shadows.
- Android feel: ripple everywhere, FAB for primary creation, elevation, Material icons.
- Both: respect insets, dark mode, dynamic text where possible.

## Anti-slop checklist (RN/Expo)

- Styling matches the repo's system (NativeWind className-only / Tamagui / restyle / token-backed StyleSheet) — no mixing.
- No inline hex/magic numbers; tokens or Tailwind config drive everything; light + dark.
- Long lists use FlatList/FlashList with empty + loading states.
- `Pressable` with ripple + ≥48dp targets; safe-area insets honored.
- `npx expo install` for new deps; required config/plugins noted.
