# Advanced Components — composite controls done right

The basics live in `components.md` (buttons, fields, cards, app bars, nav, FAB, sheet, dialog, avatar, badge). This file covers the **composite** controls AI most often fakes or fumbles: pickers, OTP, toast systems, sliders, search, sticky CTAs. Each needs all interaction states, ≥44/48dp hit targets, real tokens (never magic numbers), and a haptic at the commit moment. Cross-refs: `components.md`, `data-and-forms.md`, `motion-recipes.md`, `locale-uz.md`, `design-tokens-starter.md`.

> Rule of thumb: if it has selection, a threshold, or an async result, it commits — and a commit fires a haptic (`gestures-and-haptics.md`). Use platform-native pickers when one exists; only hand-roll when the native control can't do the job.

---

## Date / calendar picker

For picking a day, a range, or date+time. **Always prefer the native picker** — users know it and it handles locale, accessibility and time zones. Build a custom sheet calendar only for ranges or branded booking flows. Format display per locale: UZ/CIS uses `DD.MM.YYYY`, 24h (`locale-uz.md`).

Spec: trigger is a field-style row (label + selected value + calendar icon, ≥48dp). Sheet/dialog presents the calendar; selected day = `primary` circle, today = ringed, disabled days dimmed. Range = filled track between endpoints, rounded caps. Confirm with a button (don't commit on every tap) and light haptic on confirm.

| Need | iOS | Android | RN/Expo |
| --- | --- | --- | --- |
| Single date | `DatePicker` (`.graphical`) | `DatePicker` (M3) | `@react-native-community/datetimepicker` |
| Date + time | same, `.dateAndTime` | `DatePicker` + `TimePicker` | same, `mode="datetime"` (iOS) / two prompts (Android) |
| Range | custom sheet calendar | `DateRangePicker` (M3) | custom sheet (`react-native-calendars`) |

**RN/Expo** (native single picker)
```tsx
import DateTimePicker from '@react-native-community/datetimepicker';
// trigger is a field row: <Text>{fmtDate(date)}</Text> + calendar icon, ≥48dp
{open && (
  <DateTimePicker
    value={date} mode="date" display={Platform.OS === 'ios' ? 'inline' : 'default'}
    onChange={(e, d) => {
      setOpen(Platform.OS === 'ios');           // Android dialog closes itself
      if (e.type === 'set' && d) { setDate(d); Haptics.selectionAsync(); }
    }}
  />
)}
```

**Compose** (M3 dialog)
```kotlin
val state = rememberDatePickerState()
if (show) DatePickerDialog(
  onDismissRequest = { show = false },
  confirmButton = { TextButton(onClick = { onPick(state.selectedDateMillis); show = false }) { Text("OK") } },
  dismissButton = { TextButton(onClick = { show = false }) { Text("Cancel") } },
) { DatePicker(state = state) }        // DateRangePicker(rememberDateRangePickerState()) for ranges
```

**SwiftUI** — `DatePicker("Date", selection: $date, displayedComponents: .date).datePickerStyle(.graphical)`. Ranges: gate two `DatePicker`s or use a custom grid; SwiftUI has no built-in range picker.

---

## OTP / code input

For SMS/email verification — the dominant login in the UZ/CIS market (`locale-uz.md`). Spec: N separate boxes (usually 4–6), one digit each, ≥44dp tall with comfortable gap; active box has `primary` border + caret; filled boxes show the digit large/tabular. Auto-advance on entry, backspace moves to the previous box, **paste fills all boxes**, and SMS autofill is wired. Numeric keypad. On the final digit, auto-submit, fire success/error haptic, and shake + error color on wrong code.

Autofill hooks: iOS `textContentType(.oneTimeCode)`; Android SMS Retriever / autofill hint `smsOTPCode`; RN single hidden `TextInput` with `textContentType="oneTimeCode"` (iOS) and `autoComplete="sms-otp"` (Android) drives the visual boxes.

**RN/Expo** (one hidden input drives the boxes — most robust for paste + autofill)
```tsx
const LEN = 6;
const onChange = (v: string) => {
  const next = v.replace(/\D/g, '').slice(0, LEN);
  setCode(next);
  if (next.length === LEN) { Keyboard.dismiss(); Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success); onComplete(next); }
};
<Pressable style={styles.row} onPress={() => ref.current?.focus()}>
  {Array.from({ length: LEN }).map((_, i) => (
    <View key={i} style={[styles.box, i === code.length && { borderColor: t.primary, borderWidth: 2 }, error && { borderColor: t.error }]}>
      <Text style={styles.digit}>{code[i] ?? ''}</Text>
    </View>
  ))}
  <TextInput ref={ref} value={code} onChangeText={onChange}
    keyboardType="number-pad" textContentType="oneTimeCode" autoComplete="sms-otp"
    maxLength={LEN} style={StyleSheet.absoluteFill} caretHidden />
</Pressable>
// box: { width:44, height:52, borderRadius:t.radius.sm, borderWidth:1, borderColor:t.outline, alignItems:'center', justifyContent:'center', backgroundColor:t.surfaceContainer }
```

**SwiftUI** — one `TextField` with `.keyboardType(.numberPad).textContentType(.oneTimeCode)` masked by an `HStack` of digit cells (same hidden-field trick). **Compose** — `BasicTextField` with `KeyboardType.NumberPad` + `decorationBox` rendering the cells; read autofill via `AutofillType.SmsOtpCode`.

---

## Segmented control / tabs-pill

Exclusive choice among 2–5 short options in a pill track. iOS = segmented control; Material = a pill-style tab row. For >5 options or longer labels, use scrollable tabs or chips instead.

Spec: rounded `full`/`lg` track in `surfaceContainerHigh`; selected segment = elevated `surface` thumb (iOS) or `secondaryContainer` pill (M3) that **animates** between positions (spring, `motion-recipes.md`); equal-width segments; each segment ≥44dp tall and tappable full-height. Selection fires `selectionAsync()` haptic.

**SwiftUI** — `Picker("", selection: $tab) { Text("Day").tag(0); Text("Week").tag(1) }.pickerStyle(.segmented)`.

**Compose** (M3) — `SingleChoiceSegmentedButtonRow { options.forEachIndexed { i, l -> SegmentedButton(selected = i == sel, onClick = { sel = i }, shape = SegmentedButtonDefaults.itemShape(i, options.size)) { Text(l) } } }`.

**RN/Expo** (animated thumb)
```tsx
function Segmented({ items, value, onChange }: { items: string[]; value: number; onChange: (i: number) => void }) {
  const [w, setW] = useState(0);
  const x = useSharedValue(0);
  useEffect(() => { x.value = withSpring((w / items.length) * value, { damping: 18, stiffness: 220 }); }, [value, w]);
  const thumb = useAnimatedStyle(() => ({ transform: [{ translateX: x.value }] }));
  return (
    <View style={styles.track} onLayout={e => setW(e.nativeEvent.layout.width)}>
      <Animated.View style={[styles.thumb, { width: w / items.length }, thumb]} />
      {items.map((l, i) => (
        <Pressable key={l} style={styles.seg} hitSlop={6} onPress={() => { onChange(i); Haptics.selectionAsync(); }}>
          <Text style={[styles.segLabel, i === value && { color: t.onSurface, fontWeight: '600' }]}>{l}</Text>
        </Pressable>
      ))}
    </View>
  );
}
// track: row, bg surfaceContainerHigh, radius full, padding 3, height 36
// thumb: absolute inset 3, radius full, bg surface, elevation[1] · seg: flex:1, centered
```

---

## Stepper / quantity control

`[ − ] N [ + ]` for counts (cart qty, passengers). Spec: two ≥44dp icon buttons + a tabular numeric value between; `−` disabled (dimmed, not gone) at min, `+` at max; light haptic per step, a slightly stronger one at a boundary. Long-press to repeat is a nice touch. Don't let users type junk — keep it button-driven (or a numeric field validated to range).

**RN/Expo**
```tsx
function Stepper({ value, min = 0, max = 99, onChange }: { value: number; min?: number; max?: number; onChange: (n: number) => void }) {
  const step = (d: 1 | -1) => {
    const n = Math.min(max, Math.max(min, value + d));
    if (n === value) return;
    onChange(n);
    Haptics.impactAsync(n === min || n === max ? Haptics.ImpactFeedbackStyle.Medium : Haptics.ImpactFeedbackStyle.Light);
  };
  return (
    <View style={styles.stepper}>
      <Pressable disabled={value <= min} onPress={() => step(-1)} hitSlop={8} style={[styles.qbtn, value <= min && { opacity: 0.4 }]}><Ionicons name="remove" size={20} color={t.onSurface} /></Pressable>
      <Text style={styles.qval}>{value}</Text>
      <Pressable disabled={value >= max} onPress={() => step(1)} hitSlop={8} style={[styles.qbtn, value >= max && { opacity: 0.4 }]}><Ionicons name="add" size={20} color={t.onSurface} /></Pressable>
    </View>
  );
}
// qbtn: 44×44 centered · qval: minWidth 32, centered, tabular-nums, 16/600
```

**Compose** — `Row { IconButton(onClick = { dec() }, enabled = v > min) { Icon(Icons.Default.Remove, null) }; Text("$v"); IconButton(onClick = { inc() }, enabled = v < max) { Icon(Icons.Default.Add, null) } }`. **SwiftUI** — `Stepper("Qty: \(value)", value: $value, in: min...max)` for a labeled row, or a custom `HStack` for the boxed look.

---

## Toast / Snackbar SYSTEM

Not a one-off view — a **system**. Mistake AI makes: re-renders a `<Toast>` per screen. Correct: **one host** mounted at the app root + an **imperative API** (`toast.success(...)`, `toast.error(...)`) callable from anywhere. The host owns a **queue** (one at a time, or stack max ~3), auto-dismiss timer (~3–4s, pausable on press), optional action button, swipe-to-dismiss, and respects the **safe area**.

Spec: pill/rounded `md` card, `inverseSurface` (neutral) or tonal success/error bg, on-color text; leading status icon; enters from the bottom (above any sticky bar / tab bar + bottom inset) or top; spring in, fade+slide out. Success = subtle, error = stickier (longer/persistent) + error haptic.

| Variant | Bg token | Icon | Haptic | Default ms |
| --- | --- | --- | --- | --- |
| neutral | `inverseSurface` | — | none | 3000 |
| success | `successContainer` | check | Success | 2500 |
| error | `errorContainer` | alert | Error | 5000 / sticky |

**RN/Expo** (single host + imperative store)
```tsx
// toast.ts — global queue, no per-screen state
type T = { id: number; msg: string; variant?: 'neutral' | 'success' | 'error'; action?: { label: string; onPress: () => void } };
let listeners: ((q: T[]) => void)[] = []; let queue: T[] = []; let seq = 0;
const emit = () => listeners.forEach(l => l(queue));
export const toast = {
  show(msg: string, opts: Partial<T> = {}) {
    const item = { id: ++seq, msg, ...opts }; queue = [...queue, item]; emit();
    if (opts.variant === 'success') Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
    if (opts.variant === 'error') Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);
    setTimeout(() => { queue = queue.filter(t => t.id !== item.id); emit(); }, opts.variant === 'error' ? 5000 : 3000);
  },
  success: (m: string) => toast.show(m, { variant: 'success' }),
  error: (m: string) => toast.show(m, { variant: 'error' }),
  subscribe(fn: (q: T[]) => void) { listeners.push(fn); return () => { listeners = listeners.filter(l => l !== fn); }; },
};

// ToastHost.tsx — mount ONCE at root, above tab bar; subscribes to the queue
export function ToastHost() {
  const insets = useSafeAreaInsets();
  const [q, setQ] = useState<T[]>([]);
  useEffect(() => toast.subscribe(setQ), []);
  return (
    <View pointerEvents="box-none" style={[StyleSheet.absoluteFill, { justifyContent: 'flex-end', paddingBottom: insets.bottom + 12 }]}>
      {q.slice(-3).map(item => (
        <Animated.View key={item.id} entering={SlideInDown.springify()} exiting={FadeOut}
          style={[styles.toast, item.variant === 'error' && { backgroundColor: t.errorContainer }, item.variant === 'success' && { backgroundColor: t.successContainer }]}>
          <Text style={styles.toastText}>{item.msg}</Text>
          {item.action && <Pressable onPress={item.action.onPress} hitSlop={8}><Text style={styles.toastAction}>{item.action.label}</Text></Pressable>}
        </Animated.View>
      ))}
    </View>
  );
}
// Call anywhere: toast.error('Network failed'); toast.success('Saved');
```

**Compose** — one `SnackbarHost(hostState)` in `Scaffold`; call `scope.launch { hostState.showSnackbar(message, actionLabel, duration) }`. **SwiftUI** — an `@Observable` `ToastCenter` on the root, presented via an `.overlay(alignment: .bottom)` on the app's root view. **Flutter** — `ScaffoldMessenger.of(context).showSnackBar(SnackBar(...))` (the framework already queues).

---

## Filter chips / chip group

Selectable chips for filtering a list — single- or multi-select, usually in a horizontal scroll row. Spec: pill (`full`), ~32–36dp tall, ≥44dp tap target (use `hitSlop`/padding); unselected = `surfaceContainer`/outline, selected = `secondaryContainer` + check icon (or `primary` for emphasis); horizontal scroll with edge padding so the row bleeds to the screen edge. Selection toggles fire `selectionAsync()`. See `components.md` Chip for the basic single tag.

**RN/Expo**
```tsx
<ScrollView horizontal showsHorizontalScrollIndicator={false} contentContainerStyle={{ gap: t.space.sm, paddingHorizontal: t.space.lg }}>
  {filters.map(f => {
    const on = selected.includes(f.id);
    return (
      <Pressable key={f.id} hitSlop={8}
        onPress={() => { setSelected(on ? selected.filter(x => x !== f.id) : [...selected, f.id]); Haptics.selectionAsync(); }}
        style={[styles.chip, on && { backgroundColor: t.secondaryContainer, borderColor: 'transparent' }]}>
        {on && <Ionicons name="checkmark" size={16} color={t.onSecondaryContainer} style={{ marginRight: 4 }} />}
        <Text style={[styles.chipLabel, on && { color: t.onSecondaryContainer }]}>{f.label}</Text>
      </Pressable>
    );
  })}
</ScrollView>
// chip: row centered, height 34, paddingH 14, radius full, borderWidth 1 outline, bg surfaceContainer
```

**Compose** — `LazyRow { items(filters) { FilterChip(selected = it.id in sel, onClick = { toggle(it.id) }, label = { Text(it.label) }, leadingIcon = { if (it.id in sel) Icon(Icons.Default.Check, null) }) } }`. **Flutter** — `Wrap`/`ListView(scrollDirection: horizontal)` of `FilterChip(selected:, onSelected:)`.

---

## Slider + range slider

Continuous (or stepped) value selection — volume, price range, distance. Spec: track height ~4dp (`surfaceContainerHighest` inactive, `primary` active); thumb ≥20dp visual but ≥44dp touch target; show the live value (label above thumb or a readout). Stepped sliders snap with a `selectionAsync()` haptic per tick. Range = two thumbs over one track, filled segment between; thumbs must not cross.

**SwiftUI** — `Slider(value: $v, in: 0...100, step: 1) { Text("Vol") }`. SwiftUI has no native range slider; compose two clamped `Slider`s or use a custom gesture track.

**Compose** — `Slider(value, onValueChange, valueRange = 0f..100f, steps = 9)`; range: `RangeSlider(value = range, onValueChange = { range = it }, valueRange = 0f..100f)`.

**Flutter** — `Slider(value:, min:, max:, divisions:, onChanged:)`; range: `RangeSlider(values: RangeValues(a, b), min:, max:, onChanged:)`.

**RN/Expo** — `@react-native-community/slider` for single; range needs `react-native-awesome-slider` or `@ptomasroos/react-native-multi-slider` (no community range slider). Fire `Haptics.selectionAsync()` on `onSlidingComplete` (or per step), not on every pixel.

---

## Switch row / settings toggle row

The settings primitive: **label (+ description) + trailing switch**, whole row tappable. Spec: ≥48dp tall, label = Body/Title, description = Body Small `onSurfaceVariant` wrapping below; switch right-aligned; tapping anywhere in the row toggles. A11y: the row is **one** control — group label+description+state, announce role=switch with the description as hint (don't ship a stray focusable switch with no name). Toggle = `selectionAsync()`.

**RN/Expo**
```tsx
function SwitchRow({ label, desc, value, onValueChange }: { label: string; desc?: string; value: boolean; onValueChange: (v: boolean) => void }) {
  const toggle = () => { onValueChange(!value); Haptics.selectionAsync(); };
  return (
    <Pressable onPress={toggle} style={styles.row}
      accessibilityRole="switch" accessibilityState={{ checked: value }}
      accessibilityLabel={label} accessibilityHint={desc}>
      <View style={{ flex: 1, paddingRight: t.space.md }}>
        <Text style={styles.rowLabel}>{label}</Text>
        {desc && <Text style={styles.rowDesc}>{desc}</Text>}
      </View>
      <Switch value={value} onValueChange={toggle} />
    </Pressable>
  );
}
// row: row centered, minHeight 48, paddingV 10 · rowDesc: 13, onSurfaceVariant, marginTop 2
```

**SwiftUI** — inside a `Form`: `Toggle(isOn: $on) { VStack(alignment:.leading) { Text("Wi-Fi"); Text("Connect automatically").font(.caption).foregroundStyle(.secondary) } }` — `Toggle` already exposes the switch role and labels it. **Compose** — `Row(Modifier.toggleable(value, role = Role.Switch) { onChange(it) }) { Column { Text(label); Text(desc) }; Spacer(Modifier.weight(1f)); Switch(value, null) }` (pass `null` so the row, not the switch, handles the click). **Flutter** — `SwitchListTile(title:, subtitle:, value:, onChanged:)`.

---

## Search bar

For querying a list. Spec: rounded `full`/`lg` field, leading search icon, clear (×) button when non-empty, trailing **Cancel** (iOS) that dismisses + clears focus; placeholder dims; ≥44dp tall. **Debounce input** (~300ms) before querying — wire the query through `useDebouncedValue` and TanStack Query from `data-and-forms.md`; show recent searches (chips/rows) when empty and focused, results while typing, empty-state when none.

**RN/Expo**
```tsx
const [q, setQ] = useState('');
const [focused, setFocused] = useState(false);
const debounced = useDebouncedValue(q, 300);              // see data-and-forms.md
const { data, isFetching } = useQuery({ queryKey: ['search', debounced], queryFn: () => api.search(debounced), enabled: debounced.length > 1 });

<View style={styles.searchRow}>
  <View style={styles.searchField}>
    <Ionicons name="search" size={18} color={t.onSurfaceVariant} />
    <TextInput style={styles.searchInput} value={q} onChangeText={setQ}
      placeholder="Search" placeholderTextColor={t.onSurfaceVariant}
      returnKeyType="search" autoCorrect={false} clearButtonMode="while-editing"
      onFocus={() => setFocused(true)} onBlur={() => setFocused(false)} />
    {q.length > 0 && Platform.OS === 'android' && (
      <Pressable hitSlop={8} onPress={() => setQ('')}><Ionicons name="close-circle" size={18} color={t.onSurfaceVariant} /></Pressable>
    )}
  </View>
  {focused && <Pressable hitSlop={8} onPress={() => { setQ(''); Keyboard.dismiss(); }}><Text style={styles.cancel}>Cancel</Text></Pressable>}
</View>
{q.length === 0 && focused && <RecentSearches onPick={setQ} />}
```

**SwiftUI** — `.searchable(text: $query)` on a `NavigationStack` list gives the native bar, cancel and scopes for free; debounce with `.task(id: query)`. **Compose** — `SearchBar`/`DockedSearchBar` (M3) with `query`, `onQueryChange`, `active`, `onActiveChange`; render recents in its content slot.

---

## Bottom action bar / sticky CTA over content

A primary CTA pinned to the bottom over scrollable content (checkout total + Pay, form Submit, detail Add to cart). Spec: container hugs the **bottom safe area** (`insets.bottom`), `surface` bg with a top hairline or shadow/blur to separate it from scrolling content; the scroll view gets bottom padding equal to the bar height so the last item isn't hidden. May hold a summary (price) + button. The button itself follows `components.md` Button spec; commit fires a haptic.

**RN/Expo**
```tsx
const insets = useSafeAreaInsets();
<View style={styles.screen}>
  <ScrollView contentContainerStyle={{ paddingBottom: BAR_H + insets.bottom + 16 }}>{/* content */}</ScrollView>
  <View style={[styles.bottomBar, { paddingBottom: insets.bottom + 12 }]}>
    <View style={{ flex: 1 }}>
      <Text style={styles.barCaption}>Total</Text>
      <Text style={styles.barTotal}>{fmtUzs(total) /* locale-uz.md */}</Text>
    </View>
    <PrimaryButton title="Pay" onPress={() => { Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Medium); pay(); }} />
  </View>
</View>
// bottomBar: absolute bottom, row centered gap 12, paddingH lg / paddingTop 12, bg surface, hairline top border outlineVariant
```

**Compose** — `Scaffold(bottomBar = { Surface(tonalElevation = 3.dp) { Row(Modifier.navigationBarsPadding().padding(16.dp)) { /* total */; Button(...) } } })`. **SwiftUI** — `.safeAreaInset(edge: .bottom) { HStack { … }.padding().background(.bar) }` — keeps content from sliding under it automatically. **Flutter** — `Scaffold(bottomNavigationBar: SafeArea(child: Padding(... FilledButton)))`.

---

## Avatar group / stacked avatars + status dot

Overlapping avatars showing participants (group chat, shared trip) with a `+N` overflow. Builds on `components.md` Avatar. Spec: each avatar has a `surface`-colored ring (2dp) so overlaps read cleanly; overlap by ~40% of width; cap at 3–4 + a `+N` count chip; size from the token scale (24/32/40). Status dot = small circle bottom-right with a `surface` ring (green=online, amber=away, grey=offline) — a presence indicator, not a notification badge.

**RN/Expo**
```tsx
function AvatarGroup({ users, size = 32, max = 3 }: { users: U[]; size?: number; max?: number }) {
  const shown = users.slice(0, max);
  const extra = users.length - shown.length;
  return (
    <View style={{ flexDirection: 'row' }}>
      {shown.map((u, i) => (
        <View key={u.id} style={{ marginLeft: i === 0 ? 0 : -size * 0.4, borderRadius: size, borderWidth: 2, borderColor: t.surface }}>
          <Avatar uri={u.avatar} size={size} />
        </View>
      ))}
      {extra > 0 && (
        <View style={[styles.more, { width: size, height: size, borderRadius: size, marginLeft: -size * 0.4 }]}>
          <Text style={{ fontSize: size * 0.34, fontWeight: '600', color: t.onSurfaceVariant }}>+{extra}</Text>
        </View>
      )}
    </View>
  );
}
// status dot, overlaid on a single avatar:
// <View style={{ position:'absolute', right:0, bottom:0, width:size*0.3, height:size*0.3, borderRadius:size,
//   backgroundColor: online ? t.success : t.outline, borderWidth:2, borderColor:t.surface }} />
```

**Compose** — `Box` per avatar with negative `offset((-size*0.4))` and a `border(2.dp, surface, CircleShape)`; the `+N` is a `Surface(CircleShape)` with centered text. **SwiftUI** — `ZStack` of `Image`s with `.overlay(Circle().stroke(Color(.systemBackground), lineWidth: 2))` and `.offset(x:)` per index.

---

## Pull-to-refresh + skeleton list combo

Two states that ship together on any data list (`data-and-forms.md`, `components.md` States). **First load → skeleton** (shimmer rows matching the real layout, not a spinner). **Refresh (already have data) → pull-to-refresh** spinner at the top; don't blank the list. Empty/error handled separately.

Spec: skeleton rows mirror real row geometry (avatar circle + 2 text bars) so layout doesn't jump on load; shimmer is a subtle moving gradient (`motion-recipes.md`), not a flashing opacity. Pull-to-refresh uses the **native** refresh control; success fires a light haptic.

| Framework | Pull-to-refresh | Skeleton |
| --- | --- | --- |
| RN/Expo | `RefreshControl` on `FlatList`/`FlashList` | shimmer rows (`react-content-loader` / Reanimated gradient) |
| Compose | `PullToRefreshBox` (M3) | placeholder rows + `Brush` shimmer modifier |
| SwiftUI | `.refreshable { await reload() }` on `List` | `.redacted(reason: .placeholder)` over mock rows |
| Flutter | `RefreshIndicator` | `shimmer` package over greybox rows |

```tsx
// RN: skeleton until first data, then live list with pull-to-refresh
if (isLoading) return <SkeletonList count={8} />;     // shimmer rows matching ListRow
return (
  <FlashList
    data={items} renderItem={({ item }) => <ListRow item={item} />}
    refreshControl={<RefreshControl refreshing={isRefetching} onRefresh={() => { refetch(); Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light); }} tintColor={t.primary} />}
    ListEmptyComponent={<EmptyState />}
  />
);
```

---

## Quality checklist (advanced components)

- [ ] **Native first:** date/time uses the OS picker; search uses `.searchable`/`SearchBar`/native field; toasts use one root host. Hand-rolled only where the platform can't do it.
- [ ] **Tokens, not magic numbers:** every size/space/radius/color/elevation comes from `design-tokens-starter.md` — no raw hex or stray `borderRadius: 14`.
- [ ] **Hit targets ≥44/48dp** on every interactive part — stepper buttons, chips, OTP boxes, clear/cancel, status-row switch — via padding or `hitSlop`.
- [ ] **All interaction states** present: default / pressed / focused / selected / disabled (dimmed, not removed) / error.
- [ ] **Haptics on commit:** selection on chips/segments/switch/slider-step, light per step / medium at boundary on stepper, success/error on OTP + toast, light on refresh. No haptic on scroll or per keystroke.
- [ ] **Toast is a system:** single host at root, imperative API, queue, auto-dismiss (pausable), action, swipe-dismiss, success/error variants, above safe area + sticky bars.
- [ ] **OTP** auto-advances, accepts paste, wires SMS autofill (`oneTimeCode`/`sms-otp`), numeric keypad, auto-submits + haptic, shakes on error. (`locale-uz.md`)
- [ ] **Sticky CTA** sits above `insets.bottom`; scroll content padded so nothing hides behind it.
- [ ] **Sliders/segments** animate the thumb (spring) and snap to steps; range thumbs can't cross.
- [ ] **Search** debounced (~300ms) into the data layer (`data-and-forms.md`); recents when empty, results while typing, empty-state when none.
- [ ] **Lists** show skeletons on first load (matching geometry) and pull-to-refresh on reload — never a blank screen or full-page spinner over existing data.
- [ ] **A11y:** switch rows announce one switch control with name+state; pickers/sliders expose role + value; group avatar stacks behind a single meaningful label. (`accessibility-deep.md`)
- [ ] **Locale-correct:** dates `DD.MM.YYYY` 24h, money via `fmtUzs`, OTP for `+998` SMS login. (`locale-uz.md`)
