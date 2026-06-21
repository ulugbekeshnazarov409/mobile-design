# State & Architecture — structure that scales

AI-generated apps tend to dump every screen into one giant component, prop-drill the theme and auth session through ten levels, stuff server data into a global store, and hardcode API keys into the bundle. This file is the antidote: a clean feature-folder layout, the right home for each *kind* of state, and the provider/secret patterns a real app needs. React Native / Expo first, with short cross-framework notes.

> The single most important call: **classify your state before you store it.** Server data, client/UI state, and navigation state have different owners. Put server data in TanStack Query, not Zustand.

---

## Three kinds of state — different owners

Most "should this go in Redux/Zustand?" confusion dissolves once you classify the state. Don't mirror the server into a client store — that's how data goes stale.

| Kind | Examples | Owner | Why |
| --- | --- | --- | --- |
| **Server state** | user profile, feed, orders, anything fetched | **TanStack Query** | async cache: dedupe, refetch, stale-while-revalidate, retries, pagination — you don't re-implement this |
| **Client / UI state** | filters not yet applied, multi-step draft, "is sheet open", optimistic toggles | **Zustand** (or `useState`/`useReducer` if local) | ephemeral, app-owned, synchronous |
| **Navigation state** | current route, params, back stack | **Expo Router** (the URL) | the router *is* the state machine; don't shadow it in a store |
| **Cross-cutting config** | theme, auth session, locale | **React Context** | read-everywhere, rarely-changing; provider at the root |

Rules of thumb:
- Server data lives in Query and is **read via `useQuery`**, never copied into Zustand. If you find yourself `setUser(data)` after a fetch, delete the store slice and use the query.
- If state is used by one screen, keep it local (`useState`). Promote to Zustand only when ≥2 unrelated screens need it.
- Theme/auth/locale change rarely and are read everywhere → Context, not a store (no selector overhead, no extra dep). See `references/data-and-forms.md` for the Query/mutation/form side.

---

## Folder structure (a real app, not one file)

Feature-first: each feature owns its components, hooks, and api calls; shared primitives and infra live at the top. `app/` is *only* routes — thin screens that compose features.

```
app/                          # Expo Router routes ONLY (thin screens)
  _layout.tsx                 # root Stack: providers, splash/font gate, theme
  (tabs)/
    _layout.tsx               # bottom Tabs
    index.tsx                 # composes features/feed
    profile.tsx
  (auth)/
    _layout.tsx               # logged-out group
    sign-in.tsx
  (modal)/filter.tsx
src/
  features/
    feed/
      components/FeedCard.tsx
      hooks/useFeed.ts        # wraps useQuery
      api/feed.api.ts         # endpoint fns
    auth/
      AuthProvider.tsx
      hooks/useSession.ts
      api/auth.api.ts
  components/ui/              # design-system primitives (Button, Text, Card)
  theme/                      # tokens + ThemeProvider + useTheme
    tokens.ts
    ThemeProvider.tsx
  lib/                        # api client, query client, utils, storage
    apiClient.ts
    queryClient.ts
  store/                      # Zustand client/UI stores
    useUiStore.ts
```

Discipline: a screen in `app/` imports from `features/*` and `components/ui/*` — it should not contain raw `fetch`, inline `StyleSheet` of brand colors, or business logic. Cross-feature sharing goes through `components/ui`, `lib`, or a feature's public exports — never reach into another feature's internals.

---

## Zustand store — typed, sliced, selected

Use Zustand for **client/UI state only**. Type the store, keep actions inside it, and read with **selectors** so components re-render only when their slice changes.

```tsx
// src/store/useUiStore.ts
import { create } from 'zustand';

type UiState = {
  filtersOpen: boolean;
  activeFilters: string[];
  setFiltersOpen: (open: boolean) => void;
  toggleFilter: (id: string) => void;
  reset: () => void;
};

export const useUiStore = create<UiState>((set) => ({
  filtersOpen: false,
  activeFilters: [],
  setFiltersOpen: (open) => set({ filtersOpen: open }),
  toggleFilter: (id) =>
    set((s) => ({
      activeFilters: s.activeFilters.includes(id)
        ? s.activeFilters.filter((x) => x !== id)
        : [...s.activeFilters, id],
    })),
  reset: () => set({ filtersOpen: false, activeFilters: [] }),
}));
```

```tsx
// Read with a selector — this component re-renders ONLY when filtersOpen changes
const filtersOpen = useUiStore((s) => s.filtersOpen);
const setFiltersOpen = useUiStore((s) => s.setFiltersOpen);
// ❌ const store = useUiStore();  // subscribes to everything → re-renders on any change
```

For larger stores, split into slices (one `create` per concern, or slice functions combined) rather than one mega-store. When selecting multiple fields, use `useShallow` from `zustand/react/shallow` to avoid a new-object re-render.

**Context vs Zustand vs Query — pick by question:** *Is it from the server?* → Query. *Does it change often and need fine-grained subscriptions?* → Zustand. *Is it rarely-changing config read everywhere (theme/auth/locale)?* → Context.

---

## Auth / session pattern

A Context provider holds the session; tokens go in **`expo-secure-store`** (Keychain / Keystore), never `AsyncStorage` (unencrypted). A protected Expo Router layout redirects when there's no session — see `references/frameworks/react-native-expo.md` for the router mechanics.

```tsx
// src/features/auth/AuthProvider.tsx
import { createContext, useContext, useEffect, useState } from 'react';
import * as SecureStore from 'expo-secure-store';

type Session = { accessToken: string; userId: string } | null;
const AuthContext = createContext<{
  session: Session; loading: boolean;
  signIn: (s: NonNullable<Session>) => Promise<void>;
  signOut: () => Promise<void>;
}>(null!);

export function AuthProvider({ children }: { children: React.ReactNode }) {
  const [session, setSession] = useState<Session>(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {                                  // restore on cold start
    SecureStore.getItemAsync('session').then((raw) => {
      if (raw) setSession(JSON.parse(raw));
      setLoading(false);
    });
  }, []);

  const signIn = async (s: NonNullable<Session>) => {
    await SecureStore.setItemAsync('session', JSON.stringify(s));  // encrypted store
    setSession(s);
  };
  const signOut = async () => { await SecureStore.deleteItemAsync('session'); setSession(null); };

  return <AuthContext.Provider value={{ session, loading, signIn, signOut }}>{children}</AuthContext.Provider>;
}
export const useSession = () => useContext(AuthContext);
```

```tsx
// app/(tabs)/_layout.tsx — protected group gate
import { Redirect, Stack } from 'expo-router';
import { useSession } from '@/features/auth/AuthProvider';

export default function ProtectedLayout() {
  const { session, loading } = useSession();
  if (loading) return null;                          // keep splash up
  if (!session) return <Redirect href="/(auth)/sign-in" />;
  return <Stack screenOptions={{ headerShown: false }} />;
}
```

**Refresh flow:** put it in the api client (`lib/apiClient.ts`), not components. On a `401`, run a single in-flight refresh (dedupe concurrent calls with a shared promise), write the new token to SecureStore, retry the request once; if refresh fails, `signOut()`. Never keep refresh tokens in JS-readable globals.

---

## Theme provider — tokens via hook, not props

Define a token object, map it for light/dark, drive it off `useColorScheme()`, and expose it through a `useTheme()` hook so no screen prop-drills colors. The token shape comes from `references/design-tokens-starter.md`.

```tsx
// src/theme/ThemeProvider.tsx
import { createContext, useContext } from 'react';
import { useColorScheme } from 'react-native';
import { tokens } from './tokens';

const themes = {
  light: { bg: tokens.color.neutral0, text: tokens.color.neutral900, accent: tokens.color.brand500, ...tokens },
  dark:  { bg: tokens.color.neutral900, text: tokens.color.neutral0, accent: tokens.color.brand400, ...tokens },
};
type Theme = typeof themes.light;
const ThemeContext = createContext<Theme>(themes.light);

export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const scheme = useColorScheme();                   // 'light' | 'dark' | null
  return <ThemeContext.Provider value={themes[scheme ?? 'light']}>{children}</ThemeContext.Provider>;
}
export const useTheme = () => useContext(ThemeContext);
```

Now screens read `const t = useTheme();` and use `t.bg`, `t.spacing.md`, `t.radius.lg` — one place to restyle the whole app. Wrap the tree once in `app/_layout.tsx` (alongside `AuthProvider` and the Query client provider).

---

## Env & secrets — nothing private in the bundle

The JS bundle ships to the device and is fully readable. **Only public, non-sensitive values** may live in it, prefixed `EXPO_PUBLIC_*`. Real secrets (DB creds, third-party private keys, signing secrets) stay server-side behind your own API.

```ts
// app.config.ts — typed config; reads .env at build time
import { ExpoConfig } from 'expo/config';

export default (): ExpoConfig => ({
  name: 'MyApp',
  slug: 'myapp',
  extra: {
    apiUrl: process.env.EXPO_PUBLIC_API_URL,         // public base URL — fine in bundle
  },
});
```

```ts
// usage — safe public values only
const API_URL = process.env.EXPO_PUBLIC_API_URL!;
// ❌ const STRIPE_SECRET = process.env.STRIPE_SECRET_KEY;  // NEVER — ships to every device
```

Rules: never hardcode an API key/secret in a `.tsx` file; never put a `*_SECRET` in `EXPO_PUBLIC_*`; route privileged calls through a backend that holds the secret; put runtime session tokens in `expo-secure-store`, not env.

---

## Design-system layer — `components/ui/`

Screens compose **tokens, never raw values**. Wrap primitives once in `components/ui/` so restyling happens in one file and every screen stays consistent.

```tsx
// src/components/ui/Button.tsx
import { Pressable, Text, PressableProps } from 'react-native';
import { useTheme } from '@/theme/ThemeProvider';

export function Button({ label, ...props }: { label: string } & PressableProps) {
  const t = useTheme();
  return (
    <Pressable
      style={({ pressed }) => ({
        backgroundColor: t.accent,
        paddingVertical: t.spacing.md,
        paddingHorizontal: t.spacing.lg,
        borderRadius: t.radius.lg,
        opacity: pressed ? 0.85 : 1,
      })}
      {...props}
    >
      <Text style={{ color: t.color.neutral0, fontWeight: '600' }}>{label}</Text>
    </Pressable>
  );
}
```

Build `Button`, `Text`, `Card`, `Input` this way; screens import these, not raw `Pressable` with inline hex. Payoff: a brand restyle touches `tokens.ts` + `components/ui/` only — zero screen edits.

---

## Cross-framework notes (same principles)

- **Jetpack Compose (Kotlin):** state in a `ViewModel` exposed as `StateFlow`, collected with `collectAsStateWithLifecycle()`; inject the repository/api client with **Hilt** (`@HiltViewModel`, `@Inject`); theme via `MaterialTheme` + a `CompositionLocal` for custom tokens.
```kotlin
@HiltViewModel
class FeedViewModel @Inject constructor(repo: FeedRepository) : ViewModel() {
  val state: StateFlow<FeedUiState> = repo.feed.map(::toUiState)
    .stateIn(viewModelScope, SharingStarted.WhileSubscribed(5_000), FeedUiState.Loading)
}
```
- **SwiftUI (Swift):** state in an `@Observable` class; inject app-wide services (session, theme) via `.environment(...)`, read with `@Environment`; secrets in Keychain, not `UserDefaults`. `@Observable final class Session { var token: String? }`
- **Flutter (Dart):** **Riverpod** splits server (`FutureProvider`/`AsyncNotifier`) from client state (`NotifierProvider`); theme via `ThemeData` + `Theme.of(context)`; read with `ref.watch`, never global mutable singletons.

---

## Quality checklist (architecture)

- [ ] State classified: server data in TanStack Query, UI/client in Zustand, navigation in Expo Router — no server data copied into a store.
- [ ] Feature-first folders (`features/<feature>/` with components+hooks+api); `app/` holds thin route screens only.
- [ ] Zustand stores are typed and read via selectors (no whole-store subscriptions); slices for large stores.
- [ ] Theme & auth session exposed through Context/hooks — no prop-drilling of colors or session.
- [ ] Auth tokens in `expo-secure-store` (not AsyncStorage); protected layout redirects; refresh deduped in the api client.
- [ ] Theme driven by `useColorScheme()` with light/dark token maps; screens read `useTheme()`, not hex.
- [ ] No secrets in the bundle: only `EXPO_PUBLIC_*` public values; private keys stay server-side; no hardcoded keys.
- [ ] Screens compose `components/ui/` primitives and tokens, never raw values — one place to restyle.
