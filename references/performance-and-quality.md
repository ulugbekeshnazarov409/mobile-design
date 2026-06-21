# Performance & Quality — 60fps as polish

Performance is a design property, not an afterthought. A screen with perfect spacing, color, and motion still reads as *cheap* the instant it stutters: dropped frames on scroll, a thumbnail that pops in at full-res, a tap that lags because the whole tree re-rendered, a 3-second white launch. The eye reads jank as low quality before it reads anything else. This file is the perf-as-polish layer — keep every frame under the **16ms budget** (≈60fps; 8ms for 120Hz ProMotion).

> One rule above all: **work that touches the screen each frame belongs off the main/UI thread.** Lists virtualize, images decode async, animations run on the compositor, expensive effects are capped. Jank is almost always one of these four failing.

---

## The frame budget

| Refresh | Budget / frame | Miss = |
| --- | --- | --- |
| 60Hz | **16.6ms** | one dropped frame, visible stutter |
| 120Hz (ProMotion / high-refresh Android) | **8.3ms** | smoothness regression |

Everything below is in service of staying inside that budget. Measure before optimizing — guess wrong and you waste the polish pass.

---

## 1. Lists — virtualize, never render eagerly

The single biggest source of jank. Rendering a 500-row list with `.map()` builds 500 views up front: slow mount, huge memory, scroll death. Virtualize so only visible (+ a small window) rows exist.

**RN / Expo** — `@shopify/flash-list` (preferred) or `FlatList`:
```tsx
import { FlashList } from '@shopify/flash-list';

const renderItem = useCallback(({ item }: { item: Row }) => <RowItem row={item} />, []);
const keyExtractor = useCallback((r: Row) => r.id, []);

<FlashList
  data={rows}
  renderItem={renderItem}            // stable ref — NOT an inline arrow
  keyExtractor={keyExtractor}
  estimatedItemSize={88}             // FlashList needs this to recycle well
  getItemType={(r) => r.kind}        // distinct layouts → separate recycle pools
  ItemSeparatorComponent={Separator}
  ListEmptyComponent={Empty}
/>
```
`FlatList` equivalents: `getItemLayout={(d, i) => ({ length: H, offset: H * i, index: i })}` (skips measurement for fixed-height rows), `removeClippedSubviews`, `windowSize`, `maxToRenderPerBatch`. Row component wrapped in `React.memo`. **Never `.map()` real data inside a `ScrollView`.** See `frameworks/react-native-expo.md` §Lists.

**Compose** — `LazyColumn` / `LazyRow` / `LazyVerticalGrid` with stable keys + `contentType`:
```kotlin
LazyColumn {
    items(rows, key = { it.id }, contentType = { it.kind }) { row ->
        RowItem(row)   // stable key = correct item animation + reuse; contentType = layout reuse across types
    }
}
```
Keys keep `animateItem()` and recomposition correct on reorder/insert; `contentType` lets Compose reuse layout nodes between same-typed items.

**SwiftUI** — `List` recycles for free; use `LazyVStack`/`LazyHStack` inside `ScrollView` so rows build on demand:
```swift
ScrollView { LazyVStack(spacing: 0) { ForEach(rows) { RowItem(row: $0) } } }
```
Give `ForEach` a stable `id`. Plain `VStack` builds everything eagerly — only for short, fixed content.

**Flutter** — `ListView.builder` / `GridView.builder` / `CustomScrollView` + `SliverList`:
```dart
ListView.builder(
  itemCount: rows.length,
  itemExtent: 88,                    // fixed extent → cheaper layout, smoother scroll
  itemBuilder: (_, i) => RowItem(row: rows[i]),
);
```
Never a `Column` of N children inside a `SingleChildScrollView` for real data — it builds all N.

---

## 2. Re-render control — stop the storm

A tap should repaint the thing that changed, not the whole screen.

**RN** — the inline-object trap is the #1 cause of wasted renders:
```tsx
const Row = React.memo(function Row({ row }: { row: Row }) { /* ... */ });

// BAD: new object/array/fn each render → memo defeated, children re-render
<Row row={row} style={{ padding: 12 }} onPress={() => open(row.id)} />

// GOOD: hoisted StyleSheet + stable callback
const styles = StyleSheet.create({ row: { padding: 12 } });
const onPress = useCallback(() => open(row.id), [row.id]);
<Row row={row} style={styles.row} onPress={onPress} />
```
- `useMemo` for derived/expensive values; `useCallback` for handlers passed to memoized children.
- Store reads via **selectors** (Zustand `useStore(s => s.count)`, Redux `useSelector`) so only subscribers of that slice re-render — never read the whole store object.
- Lift state down: keep fast-changing state (text input, animation) in the smallest component that needs it.

**Compose** — recomposition is cheap *if scoped*; keep params stable:
```kotlin
@Immutable data class RowUi(val id: String, val title: String)   // stable → skippable

@Composable fun Counter(items: List<RowUi>) {
    val total by remember(items) { derivedStateOf { items.sumOf { it.price } } }  // recompute only when result changes
    Text("$total")
}
```
- Mark UI models `@Immutable`/`@Stable`; prefer `ImmutableList` (kotlinx.collections.immutable) — a raw `List` param is treated as unstable and kills skipping.
- `remember` expensive computations; `derivedStateOf` when many state reads collapse into fewer emissions (scroll offset → "isScrolled").
- Defer state reads to the narrowest scope (read in a lambda / `Modifier.graphicsLayer { }` / `drawBehind`) so only that layer recomposes — not the whole composable. Use **Layout Inspector recomposition counts** to find offenders.

**SwiftUI** — narrow what observes change:
```swift
@Observable final class Model { var query = ""; var results: [Row] = [] }  // only views reading a property invalidate on its change
struct RowItem: View, Equatable { let row: Row; var body: some View { /* ... */ } }
// RowItem().equatable()  → SwiftUI skips body when row is unchanged
```
With `@Observable` (iOS 17+), a view re-evaluates only for the exact properties it reads — far tighter than `@ObservableObject`/`@Published`. Split big views; pass value types; mark cheap leaf views `Equatable`.

**Flutter** — `const` everything constructible at compile time, and isolate repaints:
```dart
const SizedBox(height: 12);                 // const subtree → skipped on rebuild
RepaintBoundary(child: ExpensiveChart());   // confine repaints to this layer
```
- `const` constructors stop rebuilds propagating; make widgets `const`-friendly (final fields).
- `RepaintBoundary` around animated/independently-repainting subtrees so a parent rebuild doesn't repaint them.
- Split `build` into small widgets; `ValueListenableBuilder`/`Selector` rebuild only the leaf, not the screen.

---

## 3. Images — the silent jank source

Wrong-sized images thrash layout and blow memory; full-res into a 64px thumbnail decodes megabytes for nothing.

| Do | Don't |
| --- | --- |
| Request/decode at display size (server resize or `resizeMode` + fixed dims) | Load 4000px source into a list thumbnail |
| Cache decoded images (memory + disk) | Re-decode every scroll |
| Blurhash/placeholder + fade-in | White flash → pop |
| Prefetch above-the-fold / next page | Lazy-load the hero the user is staring at |

**RN / Expo** — always `expo-image` for content (caching, blurhash, `recyclingKey`), never core `Image`:
```tsx
import { Image } from 'expo-image';
<Image
  source={thumbUrl}                  // ask the backend for a thumbnail URL, not the original
  placeholder={{ blurhash }}
  contentFit="cover"
  transition={200}
  recyclingKey={item.id}             // correct reuse inside FlashList
  cachePolicy="memory-disk"
  style={{ width: 64, height: 64, borderRadius: 12 }}
/>
// Prefetch above the fold:  Image.prefetch(urls)
```
Full image rules in `frameworks/react-native-expo.md` §Images.

- **Compose:** Coil `AsyncImage` — set `.size(...)` in the request, `placeholder`/`crossfade(true)`; Coil caches mem+disk by default.
- **SwiftUI:** `AsyncImage` for simple cases; for lists prefer Kingfisher/Nuke (`AsyncImage` doesn't cache aggressively and can re-fetch). Downsample large sources before display.
- **Flutter:** `Image.network` with `cacheWidth`/`cacheHeight` (decode at target size), or `cached_network_image` for disk cache + placeholder/fade.

---

## 4. Animations on the right thread

A `setState`-per-frame loop animates on the JS/UI thread and competes with everything else → guaranteed jank. Drive motion on the **UI/compositor thread** instead. (Recipes & timings: `motion-recipes.md`.)

**RN** — Reanimated worklets run on the UI thread; nothing crosses the bridge per frame:
```tsx
const x = useSharedValue(0);
const style = useAnimatedStyle(() => ({ transform: [{ translateX: x.value }] })); // runs on UI thread
x.value = withSpring(120);
// NEVER: setInterval(() => setX(v => v + 1), 16) — JS-thread loop, drops frames
```
Prefer transform/opacity (compositor-only). Animating `width`/`height`/`top` triggers layout each frame — expensive; use `transform: scale/translate` instead.

- **Compose:** `animate*AsState` / `Animatable` / `updateTransition` drive the value off recomposition; animate via `graphicsLayer { }` (scale/translate/alpha) to skip relayout. Avoid animating in a way that recomposes large subtrees every frame.
- **SwiftUI:** `withAnimation` / `.animation(_, value:)`; transforms (`.scaleEffect`, `.offset`, `.opacity`) are cheap. Avoid animating layout-affecting values (frame size, stack spacing) when a transform achieves the same look.
- **Flutter:** `AnimatedBuilder`/implicit `Animated*` widgets; wrap the animated subtree in `RepaintBoundary`. Transform/opacity over layout-driving properties.

**Across all:** opacity + transform = compositor (cheap). Size/position/layout = relayout (costly). Reach for the former.

---

## 5. Startup & bundle — first frame fast

| Lever | RN / Expo | Compose | SwiftUI | Flutter |
| --- | --- | --- | --- | --- |
| Lazy routes/screens | Expo Router code-splits per route; `React.lazy` for heavy modals | nav graph loads per-destination | lazy `NavigationStack` destinations | deferred routes / `--split-debug-info` |
| Hold splash until ready | `expo-splash-screen` `hideAsync()` after fonts/data | keep splash to first compose | LaunchScreen storyboard → ready | native splash until first frame |
| Defer heavy work | run off first frame (`InteractionManager.runAfterInteractions`) | `LaunchedEffect` after first compose | `.task` post-appear | `addPostFrameCallback` |
| Engine / tree-shake | **Hermes** (default), tree-shaking, avoid huge libs | R8 minify + shrink | dead-code strip, whole-module opt | tree-shaking, `--obfuscate` |

```tsx
// RN: gate render on fonts so there's no flash of system font, then defer non-critical work
const [loaded] = useFonts({ Inter: require('../assets/fonts/Inter.ttf') });
useEffect(() => { if (loaded) SplashScreen.hideAsync(); }, [loaded]);
if (!loaded) return null;
useEffect(() => { InteractionManager.runAfterInteractions(() => warmCaches()); }, []);
```

- **Asset budget:** ship right-sized images (WebP/AVIF), subset fonts to used weights, lazy-load below-the-fold media. A 2MB hero PNG delays first paint.
- **Don't import the world:** pull single functions (`import debounce from 'lodash/debounce'`, not all of `lodash`); audit bundle with `npx expo-atlas` / source-map-explorer. Confirm **Hermes** is on (RN) — faster startup, lower memory than JSC.

---

## 6. Expensive effects — blur, shadow, gradient, overdraw

Glass and depth are premium *until* they tank the frame rate. Real-time blur, large shadows, and stacked translucency each cost GPU and cause **overdraw** (the same pixel painted many times). See `glassmorphism-and-materials.md` for the disciplined usage.

- **Cap blur layers:** ≤ 1–2 per screen; never a scrolling list of blurred items. Blur is per-frame GPU work.
- **Shadows are not free:** RN iOS shadows on many items are costly — prefer a single elevated card; Compose/Flutter `elevation` is cheaper than a hand-rolled blurred shadow. Avoid huge `shadowRadius` over big areas.
- **Reduce overdraw:** don't stack opaque background on opaque background. Flat-fill behind a card instead of layering 4 translucent rects. (Android: enable *Debug GPU overdraw* to see it.)
- **Offscreen rendering is a trap (iOS):** rounded corners + shadow + masks can force offscreen passes. `RoundedRectangle` clip + `compositingGroup()` deliberately; profile with Instruments Core Animation.
- **Gradients:** a couple are fine; a gradient on every card multiplies fill cost — use flat fills as the default (`anti-patterns.md`).

---

## 7. Measuring — find the frame, don't guess

| Stack | Tools | Look for |
| --- | --- | --- |
| RN / Expo | React DevTools **Profiler**, Perf Monitor (dev menu), Hermes sampling profiler, Flipper / Reactotron | wasted renders, JS-thread FPS vs UI-thread FPS, long tasks |
| Compose | **Layout Inspector** (recomposition counts), **Macrobenchmark** + `FrameTimingMetric`, Composition tracing, Perfetto | high recomposition counts, jank frames, startup time |
| SwiftUI | **Instruments**: Time Profiler, **SwiftUI** template (view body / update cost), Core Animation, Hangs | hot `body` evaluations, hitches, offscreen passes |
| Flutter | **DevTools** Performance/Timeline (UI vs **raster** thread), `--profile` mode, `debugProfileBuildsEnabled`, repaint rainbow | which thread blows 16ms, expensive build/paint |

Always profile in **release/profile mode on a real mid-tier device** — debug builds and simulators lie (debug Flutter/RN are far slower; simulators have desktop GPUs). Find the over-budget frame first, then fix that.

---

## 8. Memory & leaks

Leaks degrade slowly — fine on the demo, janky after 10 minutes of use.

- **Clean up subscriptions/listeners/timers:** RN `useEffect` return clears listeners/intervals (`AccessibilityInfo`, `AppState`, `Dimensions`, sockets). Compose `DisposableEffect { onDispose { } }`. SwiftUI cancel tasks in `.onDisappear` / structured `.task`. Flutter `dispose()` controllers (`AnimationController`, `ScrollController`, `TextEditingController`, streams).
- **Bound the image cache:** disk + memory caps so a long scroll session doesn't grow unbounded (`expo-image` manages this; Coil/Kingfisher/Nuke expose cache limits — set them).
- **Large lists = bounded memory only if virtualized** (§1). An eager list holds every row + image in memory at once.
- **Avoid retain cycles / captured closures:** RN stale closures capturing large state; SwiftUI `[weak self]` in escaping closures; Flutter listeners not removed. Surface real leaks as actionable diagnostics, not silent — see `error-handling-and-diagnostics.md`.

---

## Quality checklist (performance)

- [ ] Every long list is virtualized (FlashList/FlatList, LazyColumn, List/LazyVStack, ListView.builder) with stable keys/`keyExtractor` + `estimatedItemSize`/`contentType`/`itemExtent`; no `.map()`-in-ScrollView, no eager `Column`.
- [ ] Rows memoized; no inline objects/styles/closures passed to memoized children; store reads are selector-scoped.
- [ ] Re-render scoped: Compose params `@Immutable` + `derivedStateOf`/`remember`; SwiftUI narrow `@Observable` + `Equatable` leaves; Flutter `const` + `RepaintBoundary`; RN `memo`/`useMemo`/`useCallback`.
- [ ] Images sized to display, cached (mem+disk), blurhash/placeholder + fade, above-the-fold prefetched; never full-res into thumbnails (`expo-image` in RN).
- [ ] Animations run on UI/compositor thread (Reanimated worklets / `animate*AsState` / `withAnimation` / `Animated*`); transform+opacity over layout-driving props; no `setState`-per-frame loops.
- [ ] Startup: splash held until fonts/data ready, heavy work deferred off first frame, lazy routes, Hermes on (RN), assets right-sized, no whole-library imports.
- [ ] Expensive effects capped: ≤1–2 blur layers, overdraw minimized, shadows restrained, offscreen rendering checked (see `glassmorphism-and-materials.md`).
- [ ] Profiled in release/profile on a real mid-tier device; every frame within the 16ms (60fps) / 8ms (120Hz) budget.
- [ ] Listeners/controllers/timers cleaned up; image cache bounded; no leaks that degrade after extended use.
