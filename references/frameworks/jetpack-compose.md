# Jetpack Compose (Kotlin) + Material 3

Target: idiomatic, premium Android UI with Material 3 (`androidx.compose.material3`). Use Material You dynamic color where it fits, always provide light + dark.

## Setup & theming

Define one `MaterialTheme` with color, type, and shape. Drive everything off `MaterialTheme.colorScheme` / `.typography` / `.shapes` — never hardcode colors in composables.

```kotlin
@Composable
fun AppTheme(darkTheme: Boolean = isSystemInDarkTheme(), content: @Composable () -> Unit) {
    val context = LocalContext.current
    val colorScheme = when {
        Build.VERSION.SDK_INT >= Build.VERSION_CODES.S ->
            if (darkTheme) dynamicDarkColorScheme(context) else dynamicLightColorScheme(context)
        darkTheme -> darkColorScheme(primary = BrandPrimary, /* … */)
        else -> lightColorScheme(primary = BrandPrimary, /* … */)
    }
    MaterialTheme(
        colorScheme = colorScheme,
        typography = AppTypography,
        shapes = Shapes(
            small = RoundedCornerShape(8.dp),
            medium = RoundedCornerShape(12.dp),
            large = RoundedCornerShape(16.dp),
        ),
        content = content,
    )
}
```

- Seed your own scheme from a brand color with `dynamicColorScheme` (Material color utilities) when you want a fixed brand instead of wallpaper-based.
- Type: build `Typography` once with the M3 roles you use (titleMedium, bodyMedium, labelLarge…).

## Layout idioms

- `Scaffold(topBar, bottomBar, floatingActionButton)` is the screen frame; it hands you `innerPadding` — apply it to content.
- Spacing via `Arrangement.spacedBy(8.dp)` and `Modifier.padding(...)` from the 4dp grid. Screen padding 16.dp.
- Lists: `LazyColumn` / `LazyVerticalGrid` with `contentPadding`, `verticalArrangement = Arrangement.spacedBy(...)`, and stable `key`s. Never render long lists in a plain `Column`.
- Edge-to-edge: `enableEdgeToEdge()` in the Activity, then consume insets with `Modifier.windowInsetsPadding(...)` / `Scaffold` handling.
- Hoist state: pass data down, events up. Composables take values + lambdas, not ViewModels, for reusability.

## Components

Prefer M3 components: `Button/FilledTonalButton/OutlinedButton/TextButton`, `OutlinedTextField`, `Card`, `ListItem`, `TopAppBar/LargeTopAppBar`, `NavigationBar`, `FloatingActionButton`, `ModalBottomSheet`, `AlertDialog`, `FilterChip`, `Badge`. See `components.md` for snippets. Don't rebuild what M3 provides.

## Navigation

Navigation-Compose:
```kotlin
val nav = rememberNavController()
NavHost(nav, startDestination = "home") {
    composable("home") { HomeScreen(onItem = { nav.navigate("detail/$it") }) }
    composable("detail/{id}") { DetailScreen(id = it.arguments?.getString("id")) }
}
```
Type-safe routes (Compose Navigation 2.8+ with `@Serializable` route objects) are preferred for new code.

## Animation

- State-based: `animateColorAsState`, `animateDpAsState`, `animateFloatAsState` for simple property changes.
- `AnimatedVisibility` (fade/slide/expand) for enter/exit; `AnimatedContent` for swapping content; `Modifier.animateItem()` for list insert/remove.
- Use `spring()` or `tween(durationMillis = 250, easing = FastOutSlowInEasing)`. M3 emphasized easing for hero transitions.
- Press feedback comes free from `clickable`/buttons (ripple). Custom: `Modifier.clickable(interactionSource, indication = ripple())`.

## Android-specific polish

- Status/nav bar: edge-to-edge, transparent system bars, correct icon contrast for theme.
- Ripple on every tappable surface. FABs for the primary creative action.
- Respect `WindowSizeClass` for tablets/foldables (switch to list-detail at expanded width).
- Use Material Symbols / `Icons.*`. Keep one icon style (outlined or filled) consistent.

## Anti-slop checklist (Compose)

- No hardcoded `Color(0xFF...)` inside composables — use `colorScheme`.
- No fixed `.dp` text sizes — use `typography` roles.
- Screen frame is a `Scaffold`, insets handled, lists are `Lazy*` with keys.
- Light + dark both verified; one accent; tonal elevation over heavy shadows.
- Reusable stateless composables + a thin stateful wrapper.
