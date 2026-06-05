# Flutter (Dart) — Material 3 + Cupertino

Target: polished cross-platform UI. Default to Material 3 (`useMaterial3: true`); reach for Cupertino widgets when the app should feel iOS-native or when the user asks for it. Adapt per platform where it matters.

## Theming

One `ThemeData` (+ dark) seeded from a brand color. Read everything from `Theme.of(context)` — never hardcode colors/sizes in widgets.

```dart
final seed = const Color(0xFF6750A4);
MaterialApp(
  theme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(seedColor: seed, brightness: Brightness.light),
  ),
  darkTheme: ThemeData(
    useMaterial3: true,
    colorScheme: ColorScheme.fromSeed(seedColor: seed, brightness: Brightness.dark),
  ),
  themeMode: ThemeMode.system,
  home: const RootScreen(),
);
```

- Colors: `Theme.of(context).colorScheme.primary / surface / onSurfaceVariant …`.
- Type: `Theme.of(context).textTheme.titleMedium / bodyMedium / labelLarge`.
- Centralize spacing/radii in a constants file (`class Insets { static const md = 16.0; }`) or theme extensions.

## Layout idioms

- `Scaffold(appBar:, body:, bottomNavigationBar:, floatingActionButton:)` is the frame.
- Spacing with `Padding`, `SizedBox(height: 16)`, `Gap` (gap package), or `Wrap`/`spacing:`. Screen padding 16. Use the 4dp grid.
- Lists: `ListView.separated` / `ListView.builder` (never a giant `Column` for long lists). Grids: `GridView.builder` with `SliverGridDelegate`.
- Collapsing headers: `CustomScrollView` + `SliverAppBar(pinned: true, expandedHeight:, flexibleSpace: FlexibleSpaceBar(...))` + `SliverList`.
- Safe areas: wrap with `SafeArea` or use `Scaffold` defaults; `MediaQuery.viewInsets` for keyboard.
- Constraints: understand `Expanded`/`Flexible` inside `Row`/`Column`; `Stack` + `Positioned` for overlays/FAB-like placement.

## Components

Material 3: `FilledButton/OutlinedButton/TextButton`, `TextField`(`InputDecoration`), `Card`, `ListTile`, `AppBar/SliverAppBar`, `NavigationBar`, `FloatingActionButton`, `showModalBottomSheet`, `AlertDialog`, `FilterChip`, `Badge`. See `components.md`. Build small reusable widgets — extract a `widget` class, don't nest 200 lines in `build`.

Cupertino: `CupertinoPageScaffold`, `CupertinoNavigationBar`, `CupertinoButton`, `CupertinoListSection.insetGrouped`, `CupertinoSlidingSegmentedControl`, `showCupertinoModalPopup`. Use `Theme.of(context).platform == TargetPlatform.iOS` or `.adaptive` constructors (`Switch.adaptive`, `CircularProgressIndicator.adaptive`) to switch.

## Navigation

- Simple: `Navigator.push(context, MaterialPageRoute(builder: …))`.
- Recommended for real apps: **go_router** (declarative, deep links, nested shells for tab scaffolds):
```dart
final router = GoRouter(routes: [
  GoRoute(path: '/', builder: (c, s) => const HomeScreen()),
  GoRoute(path: '/item/:id', builder: (c, s) => ItemScreen(id: s.pathParameters['id']!)),
]);
```
- `Hero(tag: …)` for shared-element transitions between routes.

## Animation

- Implicit first: `AnimatedContainer`, `AnimatedOpacity`, `AnimatedPadding`, `AnimatedSwitcher`, `TweenAnimationBuilder` — cover most needs with `duration: 250ms, curve: Curves.easeOutCubic`.
- Explicit when needed: `AnimationController` (+ `SingleTickerProviderStateMixin`) + `Curve` + `FadeTransition/SlideTransition/ScaleTransition`.
- `Hero` for navigation; `AnimatedList` for list insert/remove.
- Ripple: wrap tappables in `InkWell`/`InkResponse` inside a `Material` (Cards already give this).

## Cross-platform polish

- Use `.adaptive` widgets and platform checks so iOS gets Cupertino feel, Android gets Material.
- `MediaQuery`/`LayoutBuilder` for responsive (phone vs tablet — switch to two-pane at wide widths).
- Match icon set to platform feel (`Icons.*` Material, `CupertinoIcons.*` for iOS screens).
- Skeletons via `shimmer` package or animated containers; always handle empty/error.

## Anti-slop checklist (Flutter)

- `useMaterial3: true`, scheme from seed, light + dark.
- All colors/text from `Theme.of(context)`; no hardcoded hex/sizes in widgets.
- Long lists are `ListView.builder`; collapsing headers use slivers.
- Reusable extracted widgets, not mega-`build` methods.
- Platform-adaptive where it matters; one accent; tonal M3 surfaces over heavy shadows.
