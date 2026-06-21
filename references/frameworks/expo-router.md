# Expo Router — deep navigation patterns

Goes beyond the basics in `references/frameworks/react-native-expo.md`. AI gets RN navigation wrong three ways: reaching for `@react-navigation` imperatively instead of the file tree, letting Back return to the login screen, and nesting Stacks/Tabs without grasping that each `_layout.tsx` *is* a navigator. The file system **is** the navigator hierarchy — design the tree and the navigators as one thing.

> Stack: Expo SDK 56 (June 2026), Expo Router latest, New Architecture default. Enable typed routes. Pair with `references/screen-patterns.md` for screen content and `references/flow-architecture.md` for flow design.

## File tree → navigator hierarchy

Every folder with a `_layout.tsx` mounts a navigator (`Stack`, `Tabs`, `Drawer`, `Slot`). Nesting folders nests navigators. A screen renders **inside** the nearest ancestor layout.

```
app/
  _layout.tsx              # root Stack — providers, theme, auth gate
  (app)/
    _layout.tsx            # authed Stack (group, no URL segment)
    (tabs)/
      _layout.tsx          # bottom Tabs — nested INSIDE the (app) Stack
      index.tsx            # /  → Home tab
      feed/
        _layout.tsx        # Stack INSIDE the Feed tab → own back history
        index.tsx          # /feed
        [id].tsx           # /feed/123  (pushes over the tab bar)
      profile.tsx          # /profile
    settings.tsx           # /settings — sibling of (tabs): full-screen, no tab bar
  (auth)/_layout.tsx       # logged-out Stack: sign-in.tsx, sign-up.tsx
  (modals)/_layout.tsx     # Stack with presentation:'modal': filter.tsx
  +not-found.tsx           # catch-all for unmatched URLs
```

Key model: **Tabs nested in a Stack** → push routes (`feed/[id]`) cover the whole screen including the tab bar, because they live in a Stack *above* the Tabs. A Stack **inside** one tab keeps that tab's back history independent of the others.

## Route groups `(parens)` and the auth/app split

A folder in parentheses groups files and applies a shared `_layout.tsx` **without adding a URL segment**. `(app)/(tabs)/index.tsx` → `/`, not `/app/tabs`. Use the canonical `(auth)` vs `(app)` split: one group for logged-out screens, one for logged-in. The root layout decides which group is reachable.

```tsx
// app/_layout.tsx — root Stack hosts BOTH groups; the gate redirects between them
import { Stack } from 'expo-router';
import { SessionProvider } from '@/auth';

export const unstable_settings = { initialRouteName: '(app)' };

export default function RootLayout() {
  return (
    <SessionProvider>
      <Stack screenOptions={{ headerShown: false }}>
        <Stack.Screen name="(app)" />
        <Stack.Screen name="(auth)" />
        <Stack.Screen name="(modals)" options={{ presentation: 'modal' }} />
        <Stack.Screen name="+not-found" />
      </Stack>
    </SessionProvider>
  );
}
```

## Protected routes / auth gating

Gate inside a **layout**, not a screen — the layout runs before its children render, so unauthorized content never flashes. Use `<Redirect>` for declarative gating and `router.replace()` after an *action* (login/logout) so Back can't return to the auth flow.

```tsx
// app/(app)/_layout.tsx — guards the entire authed group
import { Redirect, Stack } from 'expo-router';
import { useSession } from '@/auth';

export default function AppLayout() {
  const { session, isLoading } = useSession();
  if (isLoading) return null;                 // splash still up — don't render a half-state
  if (!session) return <Redirect href="/(auth)/sign-in" />;
  return <Stack screenOptions={{ headerShown: false }} />;
}
```

```tsx
// app/(auth)/sign-in.tsx — after a successful login, REPLACE so Back doesn't reopen sign-in
const router = useRouter();
async function onSubmit() {
  await signIn(email, password);
  router.replace('/(app)/(tabs)');            // not push — clears the auth stack
}
// On logout: router.replace('/(auth)/sign-in')
```

SDK 56 also ships **`Stack.Protected`** as a declarative guard composed in a layout:

```tsx
<Stack>
  <Stack.Protected guard={!!session}><Stack.Screen name="(tabs)" /></Stack.Protected>
  <Stack.Protected guard={!session}><Stack.Screen name="(auth)" /></Stack.Protected>
</Stack>
```

## Dynamic, catch-all, optional, and typed routes

| File | Matches | Param shape |
| --- | --- | --- |
| `[id].tsx` | `/post/42` | `{ id: '42' }` |
| `[...slug].tsx` | `/docs/a/b/c` | `{ slug: ['a','b','c'] }` |
| `[[...slug]].tsx` | `/docs` **and** `/docs/a/b` | `{ slug?: string[] }` (optional catch-all) |
| `(group)/x.tsx` | `/x` | group adds no segment |

Enable typed routes (`app.json` → `{ "expo": { "experiments": { "typedRoutes": true } } }`) for autocompleted, build-time-checked `href`s:

```tsx
<Link href={{ pathname: '/post/[id]', params: { id: '42' } }}>Open</Link>
// app/post/[id].tsx
const { id } = useLocalSearchParams<{ id: string }>();
```

`useLocalSearchParams` vs `useGlobalSearchParams`: **local** returns params for the screen owning this segment and won't re-render on unrelated navigation — use for screen data (default). **global** returns the focused route's params and re-renders on *any* navigation — only for analytics/global concerns.

## Modals & sheets

`presentation` on a Stack screen controls how it animates in.

| `presentation` | Look |
| --- | --- |
| `'modal'` | iOS card sheet that slides up, drag-to-dismiss |
| `'formSheet'` | iOS resizable sheet with detents |
| `'transparentModal'` | overlay; backdrop stays visible (use for custom dialogs) |
| `'containedModal'` / `'fullScreenModal'` | platform full-screen takeover |

```tsx
// app/(modals)/_layout.tsx — iOS detents via native form sheet
<Stack screenOptions={{ presentation: 'formSheet' }}>
  <Stack.Screen name="filter" options={{
    sheetAllowedDetents: [0.4, 0.9],   // fractions of screen height
    sheetGrabberVisible: true,
    sheetCornerRadius: 24,
  }} />
</Stack>

// Dismissal (modal-aware — prefer over back() when several deep):
const router = useRouter();
router.dismiss();          // pop top modal
router.dismiss(2);         // pop 2 levels
router.dismissAll();       // close whole modal stack, back to presenter
router.dismissTo('/feed'); // pop back to a route
router.canDismiss();       // guard before dismissing
```

For sheets *over content* (maps, drag/snap, backdrop blur) prefer `@gorhom/bottom-sheet` — it's a component, not navigation. Native `formSheet` is right when the sheet is a distinct screen with its own URL.

## Deep linking, universal links, and the back stack

```json
// app.json — scheme = custom URL; associatedDomains/intentFilters = universal/app links
{ "expo": {
  "scheme": "myapp",
  "ios": { "associatedDomains": ["applinks:myapp.com"] },
  "android": { "intentFilters": [{ "action": "VIEW", "data": [{ "scheme": "https", "host": "myapp.com" }], "category": ["BROWSABLE", "DEFAULT"] }] }
}}
```

The dangerous bug: a deep link straight into `/feed/123` mounts that screen with **no parent to pop back to** — Back exits the app. Fix it with `unstable_settings.initialRouteName` in the layout so Expo Router rebuilds the back stack with an anchor.

```tsx
// app/(app)/(tabs)/feed/_layout.tsx
export const unstable_settings = { initialRouteName: 'index' }; // deep link to [id] anchors on /feed
```

Catch unmatched URLs with **`+not-found.tsx`** (the `+` prefix marks special routes). Read links at runtime with `expo-linking`: `Linking.useLinkingURL()` → current deep link; `Linking.parse(url)` → `{ hostname, path, queryParams }`.

## Headers

Set per screen with `Stack.Screen options`, or dynamically from inside the screen with `useNavigation().setOptions`.

```tsx
// Static, in a layout
<Stack.Screen name="index" options={{
  headerLargeTitle: true,                       // iOS large-title that collapses on scroll
  headerTransparent: true,                      // header floats over a hero image
  headerBlurEffect: 'systemThinMaterial',       // iOS blur behind a transparent header
  headerSearchBarOptions: { placeholder: 'Search', onChangeText: (e) => {} },
  headerRight: () => <IconButton name="bell" onPress={openInbox} />,
}} />

// Dynamic — title/actions that depend on loaded data
const navigation = useNavigation();
useLayoutEffect(() => {
  navigation.setOptions({ title: post?.title ?? 'Loading…', headerRight: () => <Share data={post} /> });
}, [navigation, post]);
```

For `headerLargeTitle` to collapse, the screen's scroll view must be the first child with `contentInsetAdjustmentBehavior="automatic"`.

## Tabs: badges, custom bar, hiding, removing

```tsx
// app/(app)/(tabs)/_layout.tsx
<Tabs screenOptions={{ tabBarActiveTintColor: '#6E56CF', headerShown: false }}>
  <Tabs.Screen name="index" options={{ title: 'Home', tabBarIcon: ({ color, size }) => <Ionicons name="home" color={color} size={size} /> }} />
  <Tabs.Screen name="inbox" options={{ title: 'Inbox', tabBarBadge: unread > 0 ? unread : undefined, tabBarBadgeStyle: { backgroundColor: '#E5484D' } }} />
  <Tabs.Screen name="hidden-detail" options={{ href: null }} /> {/* href:null = routable but hidden from the bar */}
</Tabs>
```

- **Hide tab bar on a pushed screen:** keep detail routes as siblings of `(tabs)` in the parent Stack so they cover the bar naturally, or set `tabBarStyle: { display: 'none' }` on that screen.
- **Custom tab bar:** pass `tabBar={(props) => <MyTabBar {...props} />}` and drive it from `props.state`/`props.navigation` — required for a glass/floating bar (see `references/glassmorphism-and-materials.md`).
- 3–5 tabs; selected = filled icon + accent + label.

## Navigation lifecycle hooks

| Hook | Use it for |
| --- | --- |
| `useFocusEffect(cb)` | run/cleanup when screen gains/loses focus — refetch, start/stop subscriptions, reset state |
| `useIsFocused()` | boolean to pause video/animation/polling while off-screen |
| `usePathname()` | current URL string, e.g. `/feed/42` — for active-state logic |
| `useSegments()` | array of route segments, e.g. `['(app)','(tabs)','feed']` — read the group to branch on auth |
| `useRootNavigationState()` | check the navigator is mounted before redirecting on first launch |

```tsx
useFocusEffect(useCallback(() => {
  const sub = subscribeToOrders();
  return () => sub.remove();                       // cleanup on blur — must wrap in useCallback
}, []));

const segments = useSegments();                    // branch on active group, e.g. for analytics/guards
const inAuthGroup = segments[0] === '(auth)';
```

## Transitions & animations

Expo Router's Stack is a **native stack** (react-native-screens) — transitions run on the native thread, so prefer it over a JS stack for 60fps push/pop. Tune per screen:

```tsx
<Stack.Screen
  name="detail"
  options={{ animation: 'slide_from_right', gestureEnabled: true, fullScreenGestureEnabled: true }}
/>
// animation: 'default' | 'fade' | 'slide_from_bottom' | 'flip' | 'none'
```

Expo Router has no built-in shared-element API: for hero continuity, drive a `react-native-reanimated` shared transition or layout animation in the screen body — animate the content, not the navigator. Native stack `animation` for the frame; Reanimated for in-screen motion. Honor Reduce Motion via `animation: 'fade' | 'none'`.

## Quality checklist (navigation)

- [ ] File tree mirrors the intended navigator hierarchy; each `_layout.tsx` is the right navigator (Stack/Tabs); detail routes placed to cover or keep the tab bar deliberately.
- [ ] `(auth)` vs `(app)` groups split logged-out/in; gating done in a **layout** with `<Redirect>` / `Stack.Protected`, never a flashing screen-level check.
- [ ] Post-login/logout uses `router.replace`, not `push` — Back never returns to auth.
- [ ] Typed routes enabled; `href`s are object form with typed params; `useLocalSearchParams` for screen data, `useGlobalSearchParams` only for global concerns.
- [ ] Modals use the right `presentation`; closed with `dismiss`/`dismissAll`/`dismissTo`; sheets-over-content use `@gorhom/bottom-sheet`, sheet-screens use `formSheet` detents.
- [ ] `scheme` + universal/app links configured; `unstable_settings.initialRouteName` set so deep links have a back anchor; `+not-found.tsx` present.
- [ ] Headers via `Stack.Screen options` or `useNavigation().setOptions`; `headerLargeTitle`/`headerTransparent`/`headerSearchBarOptions` used where they fit; large-title scroll wired correctly.
- [ ] Tabs limited to 3–5; badges, `href: null`, custom `tabBar`, and tab-bar hiding applied intentionally.
- [ ] Lifecycle handled: `useFocusEffect` cleans up subscriptions; off-screen work paused via `useIsFocused`.
- [ ] Native-stack `animation`/`gestureEnabled` tuned per screen; Reduce Motion downgrades to fade/none.
