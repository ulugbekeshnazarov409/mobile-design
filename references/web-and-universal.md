# Web & Universal — one codebase, three platforms

Expo runs on web via `react-native-web`, so the *same* RN tree can ship iOS, Android, and a real website. AI ignores this: it builds mobile-only layouts that render a phone column stretched edge-to-edge on a 1440px monitor, leaves `Pressable` with no hover/focus state, calls `expo-haptics`/native modules that throw in a browser, and skips `<a>`-based links and meta for SEO. This file covers making one Expo codebase look intentional on every width.

> Cross-ref: `references/frameworks/react-native-expo.md` (styling systems, deps), `references/frameworks/expo-router.md` (file-tree nav, static rendering), `references/real-world-constraints.md` (§4 form factors, large screens), `references/accessibility-deep.md` (focus order, keyboard nav). Stack: Expo SDK 56 (mid-2026), Expo Router latest, New Architecture default.

---

## When universal is worth it (decision note)

Universal earns its keep only when the **web build is a real product**, not a checkbox.

| Ship web from the RN codebase when… | Stay mobile-only / build web separately when… |
|---|---|
| Marketing/landing + app share components and brand | Web needs heavy SEO content, blog, CMS (use Next.js) |
| You want share/deep links that open in a browser | Desktop UX differs fundamentally (dense data grids, multi-pane editors) |
| Internal tools / dashboards reused on desktop | App leans on native-only libs (maps, BLE, camera-heavy) with no web path |
| You can budget responsive + hover + keyboard work | "Just to have a web link" — that's a broken phone screenshot, skip it |

Decision: if you ship web, commit to responsive layout + hover/focus + keyboard. A half-done web target is worse than none — it signals "AI forgot the platform."

## Expo web setup (brief)

Web uses the **Metro** bundler and `react-native-web`. Configure render output in `app.json`:

```json
{ "expo": { "web": { "bundler": "metro", "output": "static" } } }
```

| `web.output` | Renders | Use for |
|---|---|---|
| `"single"` | Client-only SPA (one `index.html`) | App shell, gated dashboards — no SEO needed |
| `"static"` | One HTML file **per route** at build, hydrated client-side | Marketing + app — SEO, fast first paint (recommended default) |
| `"server"` | SSR/API routes (`+api.ts`, server rendering) | Dynamic per-request HTML, server data |

`npx expo start --web` for dev; `npx expo export -p web` builds to `dist/`. Static rendering pre-renders each Expo Router route, so every screen needs to render without throwing at module/import time (guard native-only calls — see below).

## Responsive layout

Branch on width, don't hardcode a phone column. `useWindowDimensions()` re-renders on resize (web) and rotation (native) — prefer it over `Dimensions.get()` (one-shot, needs a listener).

```tsx
import { useWindowDimensions, View } from 'react-native';

const BP = { sm: 640, md: 768, lg: 1024, xl: 1280 };

export function useBreakpoint() {
  const { width } = useWindowDimensions();
  return width >= BP.xl ? 'xl' : width >= BP.lg ? 'lg'
    : width >= BP.md ? 'md' : width >= BP.sm ? 'sm' : 'base';
}
```

**Max content width** — the #1 web tell. Never let a phone layout stretch full-width on desktop. Center a capped column:

```tsx
<View style={{ flex: 1, alignItems: 'center' }}>
  <View style={{ width: '100%', maxWidth: 480, paddingHorizontal: 16 }}>
    {/* phone-width content, centered on large screens */}
  </View>
</View>
```

**Reflow columns**, don't scale a fixed grid. Drive column count off the breakpoint:

```tsx
const cols = { base: 1, sm: 2, md: 2, lg: 3, xl: 4 }[useBreakpoint()];
<View style={{ flexDirection: 'row', flexWrap: 'wrap', gap: 16 }}>
  {items.map((it) => (
    <View key={it.id} style={{ width: `${100 / cols}%`, paddingHorizontal: 8 }}>
      <Card {...it} />
    </View>
  ))}
</View>
```

**NativeWind** gives responsive variants directly — `sm:`/`md:`/`lg:`/`xl:` map to the same min-width breakpoints, so prefer them when the repo uses NativeWind:

```tsx
<View className="px-4 sm:px-6 lg:px-8 max-w-screen-sm lg:max-w-screen-lg mx-auto">
  <View className="flex-row flex-wrap gap-4">
    <Card className="w-full md:w-1/2 lg:w-1/3" />
  </View>
</View>
```

## Platform-adaptive behavior

`Platform.OS === 'web'` and `Platform.select` split behavior. Web gets hover/focus-visible, cursor, keyboard nav; native gets touch + haptics.

```tsx
import { Platform } from 'react-native';
const styles = Platform.select({
  web:    { cursor: 'pointer', userSelect: 'none', transitionDuration: '150ms' },
  default:{},
});
```

`react-native-web` extends `Pressable` state with **`hovered`** and **`focused`** (no-ops on native, so the same component is safe everywhere):

```tsx
import { Pressable, Text, Platform } from 'react-native';
import * as Haptics from 'expo-haptics';

<Pressable
  accessibilityRole="button"
  onPress={() => {
    if (Platform.OS !== 'web') Haptics.selectionAsync(); // guard native-only
    onSubmit();
  }}
  style={({ pressed, hovered, focused }) => [
    { padding: 12, borderRadius: 12, backgroundColor: '#1f6feb' },
    Platform.OS === 'web' && { cursor: 'pointer', transitionDuration: '150ms' },
    hovered && { backgroundColor: '#388bfd' },
    focused && { outlineWidth: 2, outlineColor: '#388bfd', outlineStyle: 'solid' }, // focus-visible ring
    pressed && { opacity: 0.85, transform: [{ scale: 0.98 }] },
  ]}
>
  <Text style={{ color: 'white', textAlign: 'center' }}>Continue</Text>
</Pressable>
```

For richer hover/focus interactions, `onHoverIn`/`onHoverOut` (Pressable, web-only firing) or the `@react-native-aria/interactions` `useHover`/`useFocus` hooks give controlled state. Keep the **focus ring** — never `outlineStyle: 'none'` without a visible replacement.

## Web-specific concerns

- **Real anchors**: Expo Router's `<Link href>` renders a real `<a href>` on web — right-click "open in new tab", middle-click, and crawlers all work. Don't reimplement nav with `onPress` + `router.push` for primary links; you lose the anchor. Use `<Link asChild>` to wrap a custom Pressable while keeping the `<a>`.
- **SEO / meta**: with `output: "static"` each route pre-renders. Set `<title>`/meta per route — `expo-router` exposes `+html.tsx` for the document shell and you can render `<title>`/`<meta>` via React (e.g. `react-native-helmet-async` or route `<Head>`). No meta = no search/social previews.
- **Scroll restoration**: browsers restore scroll on back; ensure scroll containers use the page (`ScrollView` on web maps to a div) so the browser, not a nested view, owns scroll position. Avoid a single 100vh container that traps scrolling.
- **Text selection**: long-form/marketing text should be selectable — don't blanket `userSelect: 'none'`. Apply it only to buttons/labels/chrome.
- **No native modules on web**: `expo-haptics`, `expo-local-authentication`, BLE, etc. throw or no-op. Guard with `Platform.OS !== 'web'`, or fork files with `.web.tsx` / `.native.tsx` extensions (Metro picks the right one) so web bundles never import the native module.

## Inputs, keyboard & forms

Web users tab through fields and press Enter — both ignored by default RN. See `references/accessibility-deep.md` for full focus-order/screen-reader handling.

- **Focus order**: DOM/source order = tab order. Lay out fields top-to-bottom in source; avoid absolute repositioning that scrambles tabbing. Keep visible `focused` rings (above).
- **Enter to submit**: `TextInput` supports `onSubmitEditing` (fires on Enter); set `returnKeyType="next"`/`"done"` and chain `ref.focus()` to the next field, submit on the last.
- **Keyboard activation**: `Pressable`/`accessibilityRole="button"` are focusable and Enter/Space-activatable on web via react-native-web — prefer them over bare `View + onPress` (not keyboard-reachable).

```tsx
<TextInput
  returnKeyType="next"
  onSubmitEditing={() => passwordRef.current?.focus()}
  blurOnSubmit={false}
/>
<TextInput ref={passwordRef} returnKeyType="done" onSubmitEditing={submit} />
```

## Images & fonts on web

- **`expo-image`** works on web (renders `<img>`, supports `contentFit`, `transition`, blurhash placeholder). Prefer it over RN `Image` for consistent behavior. Give remote images explicit width/height to avoid layout shift (CLS).
- **Fonts**: `expo-font` / `useFonts` loads fonts on web too. Gate render until loaded to avoid FOUT, and call `SplashScreen.preventAutoHideAsync()` only where it's safe (it's a no-op on web). For best web font perf, also ensure the static build can preload — fonts are emitted to `dist/assets`.

## Pitfalls — works native, breaks web (and vice versa)

| Thing | Symptom | Fix |
|---|---|---|
| `expo-haptics`, `expo-local-authentication`, BLE, `react-native-maps` | Throws / blank on web | Guard `Platform.OS !== 'web'` or `.web.tsx` fork |
| `100vh` / full-height container | Mobile-browser URL bar overlaps; scroll traps | Use `flex: 1` + page scroll, or `100dvh` in web CSS |
| `position: 'fixed'` overlays | RN native ignores `fixed`; web differs from `absolute` | Use `absolute` for native; branch styles for sticky web headers |
| `react-native-gesture-handler` gestures | Pan/swipe feel off or dead on web (pointer vs touch) | Test on web; provide hover/click affordances as fallback |
| `Pressable` with no hover/focus | "Dead" feel on desktop, invisible keyboard focus | Add `hovered`/`focused` styles + cursor |
| Full-width phone layout | Stretched, lost on wide screens | `maxWidth` + center; reflow columns |
| `onPress`-only nav links | No `<a>`, no SEO, no open-in-new-tab | `<Link href>` / `<Link asChild>` |
| `Dimensions.get()` once | Stale after browser resize | `useWindowDimensions()` |
| Shadows (`shadow*` props) | Differ between native and web box-shadow | Verify on web; `boxShadow` via web style if needed |
| `KeyboardAvoidingView` | No-op on web (no soft keyboard) | Harmless, but don't rely on it for web layout |

## Quality checklist (web & universal)

- [ ] Web target is a *deliberate* product, not a stretched phone screenshot; if shipping web, responsive + hover + keyboard are all done.
- [ ] `app.json` web `output` chosen intentionally (`static` for SEO, `single` for app-only); each route renders without throwing at import time.
- [ ] Content has a `maxWidth` and is centered on large screens; grids reflow column count by breakpoint (not a scaled fixed grid).
- [ ] `useWindowDimensions()` (not one-shot `Dimensions`) drives responsive logic; NativeWind `sm:`/`md:`/`lg:` used when the repo is NativeWind.
- [ ] Every interactive element has `hovered` + `focused` (focus-visible) styles and `cursor: 'pointer'` on web; native gets touch + guarded haptics.
- [ ] Primary nav uses `<Link href>` (real `<a>`); per-route `<title>`/meta set under static rendering.
- [ ] Native-only modules (`expo-haptics`, auth, maps, BLE) guarded with `Platform.OS` or `.web.tsx` forks — web build never imports them.
- [ ] Forms: source order = tab order; Enter submits / advances fields; buttons are `Pressable`/role button (keyboard-activatable), focus ring visible.
- [ ] `expo-image` + `expo-font` verified on web (no FOUT, no layout shift); long text is selectable, chrome/buttons are not.
- [ ] No `100vh` scroll traps or `position: fixed` assumptions; scroll restoration works on back; tested at 375px, 768px, and 1440px.
