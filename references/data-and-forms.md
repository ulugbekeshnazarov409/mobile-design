# Data & Forms — wiring UI to real async state

AI builds the happy path: it fetches, renders the array, ships. No loading skeleton, no empty state, no error+retry, no double-submit guard, no optimistic update — so the UI flickers, lies, or hangs the moment the network is slow or fails. This file wires UI to **real async state**: every fetch has four states, every mutation is guarded and optimistic, every form is validated and double-submit-proof. RN/Expo-first (TanStack Query v5 + react-hook-form + zod), with cross-framework notes.

> Rule: a screen that touches the network is not "done" until **loading, empty, error, and success** are all visible designs and every submit is **disabled while in flight**. See `interaction-patterns.md` — every screen is a state machine.

---

## Every async surface has 4+ states

Never render `data.map(...)` directly off a fetch. Map each query/mutation to explicit states and tie them to `interaction-patterns.md`:

| State | TanStack flag | UI (don't ship a bare spinner) |
| --- | --- | --- |
| **Loading (first)** | `isPending` | **skeleton** matching the real layout, not a centered spinner |
| **Empty** | `data.length === 0` | icon + one line + a primary action |
| **Error** | `isError` | human message + **Retry** button (`refetch`) |
| **Success** | `isSuccess` | the content |
| **Refreshing** | `isRefetching` | keep old data, show pull-to-refresh spinner |
| **Fetching next** | `isFetchingNextPage` | footer loader in list |

Skeleton over spinner: it preserves layout (no jump), signals structure, and feels faster. Animate per `motion-recipes.md` §8 (shimmer).

```tsx
function Skeleton({ w, h, r = 8 }: { w: number | `${number}%`; h: number; r?: number }) {
  const o = useRef(new Animated.Value(0.4)).current;
  useEffect(() => {
    Animated.loop(Animated.sequence([
      Animated.timing(o, { toValue: 1, duration: 700, useNativeDriver: true }),
      Animated.timing(o, { toValue: 0.4, duration: 700, useNativeDriver: true }),
    ])).start();
  }, [o]);
  return <Animated.View style={{ width: w, height: h, borderRadius: r, backgroundColor: '#E5E7EB', opacity: o }} />;
}

// ListSkeleton: 6 rows mirroring the real row layout (avatar + two text lines), not a centered spinner
function ListSkeleton() {
  return (
    <View style={{ gap: 16, padding: 16 }}>
      {Array.from({ length: 6 }).map((_, i) => (
        <View key={i} style={{ flexDirection: 'row', gap: 12, alignItems: 'center' }}>
          <Skeleton w={44} h={44} r={22} />
          <View style={{ gap: 8, flex: 1 }}>
            <Skeleton w="70%" h={14} /><Skeleton w="40%" h={12} />
          </View>
        </View>
      ))}
    </View>
  );
}
```

---

## TanStack Query v5 — queries

v5 uses `isPending` (no data yet) and `isError`. `isLoading` === `isPending && isFetching`. Branch in this order: error → pending → empty → content.

```tsx
function CardsScreen() {
  const { data, isPending, isError, refetch, isRefetching } = useQuery({
    queryKey: ['cards'],
    queryFn: fetchCards,
    staleTime: 30_000,     // 30s "fresh" — no refetch on remount/focus within window
    gcTime: 5 * 60_000,    // keep cache 5min after unused (was cacheTime in v4)
    retry: 2,
  });

  if (isError) return <ErrorState onRetry={refetch} />;      // message + Retry
  if (isPending) return <ListSkeleton />;                    // first load
  if (data.length === 0) return <EmptyState />;              // empty

  return (
    <FlatList
      data={data}
      keyExtractor={(c) => c.id}
      renderItem={({ item }) => <CardRow card={item} />}
      refreshControl={<RefreshControl refreshing={isRefetching} onRefresh={refetch} />}
    />
  );
}
```

**queryKey design:** array, serializable, hierarchical — `['cards']`, `['card', id]`, `['tx', { cardId, status }]`. Params live in the key so the cache splits per-variant and `invalidateQueries({ queryKey: ['card'] })` matches by prefix.

**staleTime vs gcTime:** `staleTime` = how long data is trusted (skip refetch); `gcTime` = how long unused cache survives in memory. Set `staleTime` per data volatility (balance: 0–10s; static list: minutes).

**Prefetch** before navigation so the next screen opens with data already warm:

```tsx
const qc = useQueryClient();
const onPressRow = (id: string) =>
  qc.prefetchQuery({ queryKey: ['card', id], queryFn: () => fetchCard(id), staleTime: 30_000 });
```

### Infinite feeds

```tsx
const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isPending, isError, refetch } =
  useInfiniteQuery({
    queryKey: ['feed'],
    queryFn: ({ pageParam }) => fetchFeed(pageParam),
    initialPageParam: 0,
    getNextPageParam: (last) => last.nextCursor ?? undefined, // undefined => hasNextPage=false
  });

const items = data?.pages.flatMap((p) => p.items) ?? [];

<FlatList
  data={items}
  onEndReachedThreshold={0.5}
  onEndReached={() => { if (hasNextPage && !isFetchingNextPage) fetchNextPage(); }}
  ListFooterComponent={isFetchingNextPage ? <Skeleton w="100%" h={56} /> : null}
  refreshControl={<RefreshControl refreshing={false} onRefresh={refetch} />}
/>
```

Render the end-of-list state when `!hasNextPage`; show inline retry if a page load fails.

---

## Mutations — guarded, optimistic, acknowledged

`useMutation` gives `isPending` for the in-flight button state. Disable + spinner the CTA so a second tap is impossible (double-submit guard).

```tsx
function LikeButton({ post }: { post: Post }) {
  const qc = useQueryClient();
  const m = useMutation({
    mutationFn: () => api.like(post.id),
    onMutate: async () => {
      await qc.cancelQueries({ queryKey: ['feed'] });          // stop races
      const prev = qc.getQueryData(['feed']);                  // snapshot for rollback
      qc.setQueryData(['feed'], (old) => toggleLike(old, post.id)); // optimistic
      return { prev };
    },
    onError: (_e, _v, ctx) => qc.setQueryData(['feed'], ctx?.prev), // rollback visibly
    onSettled: () => qc.invalidateQueries({ queryKey: ['feed'] }),  // reconcile with server
    onSuccess: () => Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light),
  });
  return <Pressable onPress={() => m.mutate()} disabled={m.isPending} />;
}
```

Submit button pattern (disable + spinner = no double-submit):

```tsx
<Pressable onPress={handleSubmit(onSubmit)} disabled={mutation.isPending || !isValid}>
  {mutation.isPending ? <ActivityIndicator color="#fff" /> : <Text>Send</Text>}
</Pressable>
```

**Success moment:** acknowledge before navigating — success haptic (`Haptics.notificationAsync(Success)`) + toast, *then* route (see `interaction-patterns.md`). Silent success reads as a no-op.

**Idempotency:** disabling the CTA blocks the local double-tap, but a retried/network-replayed request can still hit the server twice. For money/orders send a client-generated `Idempotency-Key` (UUID per attempt) so the server dedupes. UI guard + server idempotency together.

---

## Forms — react-hook-form + zod

zod is the single source of truth for shape, types (`z.infer`), and messages. `zodResolver` bridges it; `Controller` adapts RN inputs (they are uncontrolled-unfriendly).

```tsx
const schema = z.object({
  fullName: z.string().min(2, 'Ism kamida 2 belgi'),
  phone: z.string().regex(/^\+998 \d{2} \d{3} \d{2} \d{2}$/, 'Telefon noto‘g‘ri'),
  amount: z.coerce.number().positive('Summani kiriting'),
});
type Form = z.infer<typeof schema>;

const { control, handleSubmit, formState: { errors, isValid } } = useForm<Form>({
  resolver: zodResolver(schema),
  mode: 'onBlur',            // validate on blur; 'onChange' for live; 'onTouched' is a good default
  defaultValues: { fullName: '', phone: '', amount: 0 },
});

<Controller
  control={control}
  name="phone"
  render={({ field: { value, onChange, onBlur, ref } }) => (
    <TextInput
      ref={ref}
      value={value}
      onChangeText={onChange}
      onBlur={onBlur}
      keyboardType="phone-pad"
      returnKeyType="next"
      onSubmitEditing={() => amountRef.current?.focus()}  // focus next field
    />
  )}
/>
{errors.phone && <Text style={styles.err}>{errors.phone.message}</Text>}  // inline, per-field
```

Form rules:
- **Inline errors** under the field, shown after blur/submit — never an alert dump of all errors.
- **Focus the next field** via refs + `returnKeyType="next"` / `onSubmitEditing`; last field `returnKeyType="done"`.
- **Submit disabled until `isValid`** (and during `mutation.isPending`).
- **Keyboard handling:** wrap in `KeyboardAvoidingView` / `react-native-keyboard-controller` — see `frameworks/react-native-expo.md` (Keyboard section). Don't let the keyboard cover the submit button.
- **Never lose input** on validation/network error — preserve typed values (`interaction-patterns.md`).
- **Validation timing:** `onBlur`/`onTouched` for calm forms; `onChange` only for live constraints (password strength, available-username). Avoid yelling on first keystroke.

---

## Field patterns

| Field | Behavior | Reference |
| --- | --- | --- |
| **Phone** | fixed `+998` affix, mask `00 000 00 00`, `phone-pad`, 9-digit validate | `locale-uz.md` (Phone) |
| **Card** | 4-4-4-4 grouping, BIN brand-detect as typed, swap logo, Luhn+length | `locale-uz.md` (Cards) |
| **OTP** | N single-digit boxes, auto-advance, paste-fill, auto-submit on last, resend timer | `interaction-patterns.md` (Auth) |
| **Currency** | numeric keypad, space grouping, no decimals, `so'm` suffix | `locale-uz.md` (Currency) |

```tsx
// Phone mask -> "+998 90 123 45 67"
const maskPhone = (raw: string) => {
  const d = raw.replace(/\D/g, '').replace(/^998/, '').slice(0, 9);
  const p = ['+998', d.slice(0, 2), d.slice(2, 5), d.slice(5, 7), d.slice(7, 9)];
  return p.filter(Boolean).join(' ');
};

// OTP: N number-pad boxes; on input -> focus next, on last cell complete -> auto-submit;
// Backspace (onKeyPress) -> focus prev; support paste-fill by spreading a multi-char value.
const setCell = (i: number, t: string) => {
  const next = [...vals]; next[i] = t.slice(-1); setVals(next);
  if (t && i < length - 1) refs.current[i + 1]?.focus();
  if (next.join('').length === length) onComplete(next.join(''));
};
```

Card input: detect brand from BIN as the user types and swap the logo — see `cardBrand()` in `locale-uz.md`; treat Uzcard/Humo as valid (prefix+length), not "unknown".

---

## Error taxonomy — handle by kind

| Kind | Detect | UI response |
| --- | --- | --- |
| **Network/offline** | fetch throws / no connectivity | offline banner + Retry; queue mutation, don't lose input |
| **Validation (4xx)** | server 400/422 with field errors | map back to fields via `setError`, inline — not a toast |
| **Server (5xx)** | 500/timeout | generic "Nimadir noto‘g‘ri ketdi" + Retry; log; don't blame user |
| **Auth (401/403)** | expired/forbidden | re-auth flow / silent token refresh, then retry |

```tsx
onError: (e) => {
  if (e instanceof ZodFieldError) e.fields.forEach((f) => setError(f.path, { message: f.msg }));
  else if (e.code === 'NETWORK') showOfflineBanner();
  else toast.error('Nimadir noto‘g‘ri ketdi');
}
```

**Offline + optimistic:** optimistic mutations make the app feel offline-capable — UI updates instantly, reconciles via `onSettled` invalidate. Persist the query cache (`@tanstack/query-async-storage-persister`) so the app opens to last-known data instead of a blank skeleton.

---

## Cross-framework note

The pattern is identical everywhere — **a single state object the UI exhaustively switches on**, never raw data.

```kotlin
// Compose — sealed UiState in a StateFlow, collected lifecycle-aware
sealed interface UiState { data object Loading: UiState; data object Empty: UiState
  data class Error(val msg: String): UiState; data class Content(val cards: List<Card>): UiState }
val state by viewModel.uiState.collectAsStateWithLifecycle()
when (val s = state) {
  UiState.Loading -> CardsSkeleton()
  is UiState.Error -> ErrorRetry(s.msg, onRetry = viewModel::reload)
  UiState.Empty -> EmptyState()
  is UiState.Content -> CardsList(s.cards)
}
```

```swift
// SwiftUI — @Observable + async/await, drive load from .task, switch on phase
@Observable final class CardsVM { var phase: Phase = .loading
  func load() async { phase = .loading; do { phase = .loaded(try await api.cards()) } catch { phase = .failed } } }
enum Phase { case loading, loaded([Card]), failed }
// View: .task { await vm.load() }; switch vm.phase { case .loading: Skeleton() ... }
```

```dart
// Flutter — Riverpod AsyncValue.when covers all states (or FutureBuilder)
ref.watch(cardsProvider).when(
  loading: () => const CardsSkeleton(),
  error: (e, _) => ErrorRetry(onRetry: () => ref.invalidate(cardsProvider)),
  data: (cards) => cards.isEmpty ? const EmptyState() : CardsList(cards));
```

---

## Quality checklist (data & forms)

- [ ] Every fetch renders **loading (skeleton, not bare spinner) / empty / error+Retry / success** — branched explicitly.
- [ ] `useQuery` uses `isPending`/`isError`; `queryKey` is hierarchical + serializable; `staleTime`/`gcTime` set per volatility.
- [ ] Pull-to-refresh wired (`refetch` + `RefreshControl`); feeds use `useInfinitePage` end-loader + end-of-list state.
- [ ] Mutations: CTA disabled + spinner while `isPending` (no double-submit); money sends an idempotency key.
- [ ] Optimistic updates via `onMutate` snapshot → `onError` rollback → `onSettled` invalidate.
- [ ] Success acknowledged (haptic + toast) before navigating.
- [ ] Forms: zod schema + `zodResolver`, `Controller` per RN input, inline per-field errors, submit disabled until `isValid`.
- [ ] Keyboard handled (avoiding view), next-field focus wired, input never lost on error.
- [ ] Phone/card/OTP/currency fields follow `locale-uz.md` masks + brand detection.
- [ ] Errors handled by taxonomy (network vs validation vs server vs auth); offline cache persisted.
