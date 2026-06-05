# SwiftUI (Swift) + Apple HIG

Target: UI that feels unmistakably iOS — SF font, Dynamic Type, large titles, grouped lists, system materials, swipe-back, bottom sheets with detents. Follow Apple Human Interface Guidelines.

## Theming

iOS theming is mostly "use the system, don't fight it":
- **Color:** use semantic colors (`.primary`, `.secondary`, `Color(.systemBackground)`, `.secondarySystemGroupedBackground`, `Color(.separator)`, `.systemFill`). Light/dark/contrast come free. Define **one** app accent via `.tint(.accentColor)` (or an Asset accent color); use `.red/.green/.orange` for status only.
- **Type:** always use text styles (`.font(.body)`, `.headline`, `.title2`, `.caption`) so Dynamic Type works. Avoid `.system(size:)` except for true display numerics.
- **Materials:** `.ultraThinMaterial` / `.regularMaterial` for blur layers (bars, sheets, overlays).
- A small `Theme`/`Tokens` enum for spacing/radii is fine: `enum Space { static let md: CGFloat = 16 }`.

## Layout idioms

- `NavigationStack { … }` per tab; `ScrollView` or `List` for content. `List` with `.listStyle(.insetGrouped)` is the canonical settings/profile layout.
- `VStack/HStack/ZStack` + `Spacer()`; spacing from the grid (`spacing: 16`). `Grid` / `LazyVGrid` for grids.
- Safe areas are automatic; use `.ignoresSafeArea()` only for hero/background, and `.safeAreaInset(edge: .bottom) { CTA }` to pin a button above the home indicator.
- `.frame(maxWidth: .infinity)` for full-width CTAs. Padding default 16.

## Components & conventions

- Buttons: `.buttonStyle(.borderedProminent)` (primary), `.bordered` (secondary), `.plain` (text); `.controlSize(.large)`; `.tint(...)`.
- Lists/forms: `Form { Section("Account") { … } }` for settings; `Label("Title", systemImage: "gear")` rows; `NavigationLink` for drill-in; trailing chevrons are automatic.
- Top bar: `.navigationTitle("…")` + `.navigationBarTitleDisplayMode(.large)` + `.toolbar { ToolbarItem(placement: .topBarTrailing) { … } }`. **No FABs on iOS** — put the primary action in the nav bar or a pinned bottom button.
- Sheets: `.sheet(isPresented:) { … .presentationDetents([.medium, .large]).presentationDragIndicator(.visible) }`. Confirmations: `.confirmationDialog`; alerts: `.alert`.
- Tabs: `TabView { … .tabItem { Label(...) } }`; `.badge(n)` for counts.
- Icons: **SF Symbols** only (`Image(systemName:)`), weight/scale matched to text.

## Navigation

```swift
NavigationStack(path: $path) {
    HomeView()
        .navigationDestination(for: Item.self) { DetailView(item: $0) }
}
// push: path.append(item)
```
Use value-based `navigationDestination`; `TabView` at the root holding a stack per tab.

## Animation

- `withAnimation(.spring(response: 0.35, dampingFraction: 0.85)) { state.toggle() }` — spring is the default iOS feel.
- Implicit `.animation(_, value:)`; `.transition(.move/.opacity)` with `if`-inserted views; `matchedGeometryEffect` for shared-element/hero.
- `.contentTransition(.numericText())` for animating numbers. Respect Reduce Motion (`@Environment(\.accessibilityReduceMotion)`).
- Haptics: `.sensoryFeedback(.impact, trigger:)` on key interactions.

## iOS polish

- Large title that collapses on scroll (free with `List`/`ScrollView` + `.large`).
- Swipe-back works automatically with `NavigationStack` — don't replace the system back button needlessly.
- Swipe actions on rows: `.swipeActions { Button(role: .destructive) {…} }`.
- Pull to refresh: `.refreshable { await load() }`. Search: `.searchable(text:)`.
- `.redacted(reason: .placeholder)` for skeleton loading states.

## Anti-slop checklist (SwiftUI)

- Semantic colors + one accent; no raw hex per view; light/dark verified.
- Text styles (Dynamic Type), not fixed sizes.
- Right iOS component (Form/List/sheet/toolbar) — no Android FAB, no Material look.
- Pinned CTA via `.safeAreaInset`; spacing from the grid.
- Springs for interaction; Reduce Motion respected; SF Symbols throughout.
