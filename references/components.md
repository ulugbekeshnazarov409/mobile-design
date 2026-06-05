# Component Catalog — cross-framework recipes

For each component: what it's for, the design spec (from `design-foundations.md`), and a compact, idiomatic snippet in all four frameworks. Adapt names/imports to the project. These are starting points — always wire to real theme tokens, not the literals shown.

Conventions in snippets:
- Compose: assumes `MaterialTheme` set up; uses M3 components.
- SwiftUI: uses system styles/semantic colors.
- Flutter: assumes `useMaterial3: true` ThemeData.
- RN/Expo: plain `StyleSheet`/inline for portability; swap for NativeWind/styled if the repo uses it.

---

## Button (primary / secondary / text)

Spec: height 48–56, radius `sm`(8) or `full`, label = Label Large/medium weight, primary = `primary`/`onPrimary`, full-width for screen CTAs. Press feedback required.

**Compose**
```kotlin
Button(
    onClick = onClick,
    modifier = Modifier.fillMaxWidth().height(52.dp),
    shape = RoundedCornerShape(12.dp),
) { Text("Continue", style = MaterialTheme.typography.labelLarge) }

OutlinedButton(onClick = onClick, modifier = Modifier.fillMaxWidth().height(52.dp)) { Text("Cancel") }
TextButton(onClick = onClick) { Text("Skip") }
```

**SwiftUI**
```swift
Button(action: onTap) {
    Text("Continue").font(.headline).frame(maxWidth: .infinity)
}
.buttonStyle(.borderedProminent)
.controlSize(.large)
.tint(.accentColor)

Button("Cancel", action: onTap).buttonStyle(.bordered).controlSize(.large)
Button("Skip", action: onTap).buttonStyle(.plain).foregroundStyle(.secondary)
```

**Flutter**
```dart
SizedBox(
  width: double.infinity,
  child: FilledButton(
    onPressed: onPressed,
    style: FilledButton.styleFrom(minimumSize: const Size.fromHeight(52)),
    child: const Text('Continue'),
  ),
);
// OutlinedButton(...) / TextButton(...) for secondary / text
```

**RN/Expo**
```tsx
function PrimaryButton({ title, onPress }: { title: string; onPress: () => void }) {
  return (
    <Pressable
      onPress={onPress}
      style={({ pressed }) => [styles.btn, pressed && { opacity: 0.85 }]}
      android_ripple={{ color: 'rgba(255,255,255,0.2)' }}>
      <Text style={styles.btnText}>{title}</Text>
    </Pressable>
  );
}
const styles = StyleSheet.create({
  btn: { height: 52, borderRadius: 12, backgroundColor: t.primary, alignItems: 'center', justifyContent: 'center' },
  btnText: { color: t.onPrimary, fontSize: 15, fontWeight: '600' },
});
```

---

## Text field

Spec: height ~56, radius `sm`, label + helper/error text, leading/trailing icons optional, clear focus state. Validate inline.

**Compose**
```kotlin
OutlinedTextField(
    value = email, onValueChange = { email = it },
    label = { Text("Email") },
    singleLine = true,
    isError = error != null,
    supportingText = { error?.let { Text(it) } },
    modifier = Modifier.fillMaxWidth(),
)
```

**SwiftUI**
```swift
TextField("Email", text: $email)
    .textContentType(.emailAddress)
    .keyboardType(.emailAddress)
    .autocorrectionDisabled()
    .textFieldStyle(.roundedBorder)
// grouped Form rows are often better for settings-style input
```

**Flutter**
```dart
TextField(
  controller: emailCtrl,
  decoration: InputDecoration(
    labelText: 'Email',
    border: const OutlineInputBorder(),
    errorText: error,
    prefixIcon: const Icon(Icons.mail_outline),
  ),
);
```

**RN/Expo**
```tsx
<View>
  <TextInput
    placeholder="Email" placeholderTextColor={t.onSurfaceVariant}
    value={email} onChangeText={setEmail}
    keyboardType="email-address" autoCapitalize="none"
    style={[styles.input, focused && { borderColor: t.primary }]}
    onFocus={() => setFocused(true)} onBlur={() => setFocused(false)} />
  {error && <Text style={styles.error}>{error}</Text>}
</View>
```

---

## Card

Spec: radius `md`(12), padding 16, `surfaceContainer` bg, tonal elevation (Android) / subtle shadow + bg tier (iOS). Clear internal hierarchy.

**Compose**
```kotlin
Card(shape = RoundedCornerShape(12.dp), modifier = Modifier.fillMaxWidth()) {
    Column(Modifier.padding(16.dp), verticalArrangement = Arrangement.spacedBy(8.dp)) {
        Text("Title", style = MaterialTheme.typography.titleMedium)
        Text("Body", style = MaterialTheme.typography.bodyMedium,
            color = MaterialTheme.colorScheme.onSurfaceVariant)
    }
}
```

**SwiftUI**
```swift
VStack(alignment: .leading, spacing: 8) {
    Text("Title").font(.headline)
    Text("Body").font(.subheadline).foregroundStyle(.secondary)
}
.padding(16)
.frame(maxWidth: .infinity, alignment: .leading)
.background(Color(.secondarySystemBackground), in: .rect(cornerRadius: 12))
```

**Flutter**
```dart
Card(
  shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(12)),
  child: Padding(
    padding: const EdgeInsets.all(16),
    child: Column(crossAxisAlignment: CrossAxisAlignment.start, children: [
      Text('Title', style: Theme.of(context).textTheme.titleMedium),
      const SizedBox(height: 8),
      Text('Body', style: Theme.of(context).textTheme.bodyMedium),
    ]),
  ),
);
```

**RN/Expo**
```tsx
<View style={{ backgroundColor: t.surfaceContainer, borderRadius: 12, padding: 16, gap: 8 }}>
  <Text style={{ fontSize: 16, fontWeight: '600', color: t.onSurface }}>Title</Text>
  <Text style={{ fontSize: 14, color: t.onSurfaceVariant }}>Body</Text>
</View>
```

---

## List item / row

Spec: 56dp min height, leading icon/avatar · title (+ optional subtitle) · trailing (chevron/value/switch). Ripple/highlight on tap.

**Compose** — `ListItem(headlineContent = …, leadingContent = …, trailingContent = …)` inside a clickable `Modifier`.
**SwiftUI** — a `Label` + `Spacer` + trailing inside `NavigationLink`/`Button` rows in a `List`.
**Flutter** — `ListTile(leading:, title:, subtitle:, trailing: const Icon(Icons.chevron_right), onTap:)`.
**RN/Expo** — `Pressable` row: `flexDirection: 'row', alignItems: 'center', paddingVertical: 12, gap: 12`, trailing chevron pushed with `flex:1` spacer.

---

## Chip / tag

Spec: pill (`full` radius), 32dp tall, Label, `secondaryContainer` (selected = `primary`). Use for filters/tags.

- Compose: `FilterChip(selected, onClick, label)` / `AssistChip`.
- SwiftUI: capsule `Text` with `.padding(.horizontal,12).padding(.vertical,6).background(Capsule().fill(...))`, or a segmented `Picker` for exclusive filters.
- Flutter: `FilterChip` / `ChoiceChip` / `Chip`.
- RN: `Pressable` pill, bg = selected ? primaryContainer : surfaceVariant.

---

## Top app bar

Spec: 56dp (small) / large collapsing. Leading nav icon, centered or leading title, trailing actions (max ~3, overflow to menu). Transparent over hero, solid on scroll.

- Compose: `TopAppBar` / `LargeTopAppBar` + `TopAppBarScrollBehavior` for collapse.
- SwiftUI: `.navigationTitle(...)` + `.navigationBarTitleDisplayMode(.large)` + `.toolbar { ToolbarItem(...) }`.
- Flutter: `AppBar` / `SliverAppBar(pinned: true, expandedHeight: …, flexibleSpace: FlexibleSpaceBar(...))`.
- RN/Expo: Expo Router `Stack.Screen options={{ headerLargeTitle: true, headerRight: … }}` or a custom header View honoring `useSafeAreaInsets()`.

---

## Bottom navigation / tab bar

Spec: 3–5 items, icon + label, selected = accent + filled icon, sits above bottom safe area.

- Compose: `NavigationBar { NavigationBarItem(selected, onClick, icon, label) }`.
- SwiftUI: `TabView { View().tabItem { Label("Home", systemImage: "house.fill") } }`.
- Flutter: `NavigationBar(destinations: [NavigationDestination(icon:, label:)])`.
- RN/Expo: Expo Router `Tabs` with `tabBarIcon`, `tabBarActiveTintColor`, or React Navigation bottom tabs.

---

## FAB (Android-centric)

Spec: 56dp, `full` radius, `primaryContainer`, real shadow, bottom-right above nav, optional extended (icon + label). iOS does **not** use FABs — use a toolbar/nav-bar action instead.

- Compose: `FloatingActionButton` / `ExtendedFloatingActionButton`.
- Flutter: `FloatingActionButton` / `.extended`.
- RN: absolute-positioned `Pressable` circle with elevation/shadow (Android screens only).

---

## Bottom sheet

Spec: top corners `xl`(28 M3 / 16 iOS), grabber handle, `surfaceContainerLow` bg, drag-to-dismiss, scrim behind. Use for contextual actions/forms.

- Compose: `ModalBottomSheet(onDismissRequest, sheetState)`.
- SwiftUI: `.sheet(isPresented:) { … .presentationDetents([.medium, .large]).presentationDragIndicator(.visible) }`.
- Flutter: `showModalBottomSheet(context:, isScrollControlled: true, shape: RoundedRectangleBorder(borderRadius: vertical top 28))`.
- RN/Expo: `@gorhom/bottom-sheet` (`BottomSheetModal` with `snapPoints`) — the standard choice.

---

## Dialog / alert

Spec: radius `lg`, title + body + 1–2 actions (confirm emphasized, cancel text). Destructive = red. Keep copy short.

- Compose: `AlertDialog(onDismissRequest, title, text, confirmButton, dismissButton)`.
- SwiftUI: `.alert(title, isPresented:) { Button("Delete", role: .destructive){} Button("Cancel", role: .cancel){} }`.
- Flutter: `showDialog` → `AlertDialog(title:, content:, actions:)`.
- RN: `@gorhom` sheet or a custom `Modal` with a centered card; on simple cases `Alert.alert(...)`.

---

## Avatar

Spec: `full` radius, sizes 24/32/40/56, image with initials fallback, optional presence dot (bottom-right, `surface` ring).

All frameworks: a circular clipped image + a fallback colored circle with centered initials. Presence = small circle overlaid bottom-right with a 2px background-colored border.

---

## Badge

Spec: `full` pill, `error`/accent bg, on-color label, min size for single digit, "99+" cap. Sits top-right of an icon.

- Compose: `BadgedBox(badge = { Badge { Text("3") } }) { Icon(...) }`.
- SwiftUI: `.badge(3)` on tabs/lists, or overlay a `Capsule`.
- Flutter: `Badge(label: Text('3'), child: Icon(...))`.
- RN: absolute-positioned pill over the icon container.

---

## States: loading / empty / error

Always design these for data screens.
- **Loading:** skeleton placeholders (shimmer) matching the real layout; or a centered spinner only for full-page initial load. Compose: `Modifier.placeholder`/custom shimmer; SwiftUI: `.redacted(reason: .placeholder)`; Flutter: `shimmer` package or animated containers; RN: `react-content-loader` / animated opacity blocks.
- **Empty:** centered illustration/icon + one short line + a primary action.
- **Error:** centered icon + message + "Retry" button.
