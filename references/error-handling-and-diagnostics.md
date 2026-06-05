# Errors, Warnings & Diagnostics — zero-tolerance pass

A premium screen that throws a red box, spams the console, or ships lint warnings is not done. Treat **every error, warning, and log as a defect to fix in place — immediately**, not "later". Warnings are errors-in-waiting. This pass runs after building and before declaring done, and continuously while iterating.

> Default stance: **warning = bug.** Don't leave a single yellow line in the console or a single lint warning unaddressed. If a warning is genuinely intentional, suppress it *narrowly* with a comment explaining why — never blanket-disable.

---

## The diagnostics loop

For every change:
1. **Type-check / compile.** Resolve all type and build errors first — nothing ships red.
2. **Lint.** Run the project linter/formatter; fix every warning (don't just auto-format).
3. **Run & watch the console/logcat.** Read the actual output; hunt warnings, not just crashes.
4. **Fix at the source**, not by suppression. Re-run until the console is clean.
5. **Check all states** (empty/loading/error/long-text/dark) — many warnings only fire on a specific state.

Tooling by stack:
- **Compose:** `./gradlew assembleDebug` / `lint` / `ktlintCheck`; Android Studio "Problems"; Logcat (filter by tag/`E`/`W`).
- **SwiftUI:** Xcode build (treat warnings seriously), `swiftlint`, the Issue navigator; console for purple runtime/layout warnings.
- **Flutter:** `flutter analyze` (must be clean), `dart format`, `flutter run` console; the red/yellow error widgets.
- **RN/Expo:** `tsc --noEmit`, ESLint, Metro terminal, in-app LogBox (yellow/red), `npx expo-doctor`, Hermes/console logs.

---

## React Native / Expo — common errors & fixes

| Symptom | Cause | Fix |
| --- | --- | --- |
| `Each child in a list should have a unique "key"` | missing/duplicate `key` in mapped list | add a stable `keyExtractor`/`key`; never use index if items reorder |
| `Cannot update a component while rendering a different component` | setState during render | move to `useEffect`/event handler |
| `Maximum update depth exceeded` | setState loop / bad dep array | fix `useEffect` deps; guard the update |
| `Rendered more/fewer hooks than expected` | hook behind a condition/return | hooks at top level, unconditional |
| Red box `undefined is not an object (x.y)` | null/undefined access | optional chaining `?.`, defaults, guard before render |
| `Text strings must be rendered within a <Text>` | raw string in `<View>` | wrap in `<Text>`; check stray `{cond && 'str'}` |
| `VirtualizedList: missing keys` / nested-in-ScrollView warning | FlatList inside ScrollView | use the list's own scroll / `ListHeaderComponent` |
| `Warning: componentWillReceiveProps` / deprecation | old lifecycle/API | migrate to current API |
| `Possible unhandled promise rejection` | unawaited/uncaught async | `try/catch` or `.catch`; surface error state |
| `useInsertionEffect must not schedule updates` / Reanimated warnings | animating on JS thread / bad worklet | use `useAnimatedStyle`/worklets on UI thread |
| Metro `Unable to resolve module` | missing dep / cache | install dep; `npx expo start -c` to clear cache |
| Style/prop type warnings | wrong prop type (e.g. number vs string) | match the expected type; remove invalid props |
| `expo-doctor` flags version mismatch | dep not aligned to SDK | `npx expo install <pkg>` to align |

Discipline: no `console.log` left in shipped code; convert real diagnostics to proper logging or remove. Address every **LogBox** yellow.

---

## Jetpack Compose / Kotlin — common issues & fixes

| Symptom | Cause | Fix |
| --- | --- | --- |
| Recomposition loop / jank | reading state that writes in composition | hoist state; don't mutate during composition; use `derivedStateOf` |
| `@Composable invocations can only happen from a @Composable` | calling composable from non-composable | restructure; pass lambdas |
| Lint: hardcoded color/size, missing `contentDescription` | a11y/theming lint | use `colorScheme`/`dp` tokens; add `contentDescription` (or `null` for decorative) |
| `Modifier.fillMaxSize` inside scroll → crash | infinite constraints | use proper sizing inside `LazyColumn`/scroll |
| Skipping/unstable lambda recompositions | unstable params | stable types, `remember`, stable keys in `LazyColumn` |
| `NullPointerException` on nullable | unchecked null | safe calls `?.`, `?:`, require/checkNotNull with message |
| StrictMode / main-thread work | heavy work on main | move to coroutine/`Dispatchers.IO` |

Keep `lint`/`ktlint` clean. Don't ship `// TODO` warnings or unused imports.

---

## SwiftUI / Swift — common issues & fixes

| Symptom | Cause | Fix |
| --- | --- | --- |
| Purple runtime warning: "Publishing changes from within view updates" | state mutated during view update | defer to `.task`/`DispatchQueue.main.async`/event |
| "Modifying state during view update" | same | move mutation out of `body` |
| `Index out of range` | unsafe array access | guard / `indices.contains` / `safe:` subscript |
| Unexpectedly found nil | force-unwrap `!` | `if let`/`guard let`/`??`; avoid `!` |
| ForEach without stable id warning | non-`Identifiable`/unstable id | `Identifiable` or `id: \.self` only for stable values |
| Layout "ambiguous"/conflicting | missing frame/priority | set frames, `layoutPriority`, fix constraints |
| Deprecation warnings (`NavigationView`, etc.) | old API | migrate (`NavigationStack`, new APIs) |
| Main-thread / async warnings | blocking main | `async`/`await`, `Task`, `@MainActor` correctly |

Treat Xcode yellow warnings as must-fix. Run `swiftlint` if present and clear it.

---

## Flutter / Dart — common issues & fixes

| Symptom | Cause | Fix |
| --- | --- | --- |
| Yellow/black "overflowed by N pixels" | unbounded/oversized child | `Expanded`/`Flexible`, `SingleChildScrollView`, constrain size |
| "RenderBox was not laid out" / unbounded constraints | infinite size in flex/scroll | give bounded constraints; `shrinkWrap`/`Expanded` |
| `setState() called after dispose()` | async setState after unmount | check `mounted` before `setState` |
| `Null check operator used on a null value` | `!` on null | null-aware `?.`/`??`; guard |
| `flutter analyze` warnings/infos | lints (`use_key_in_widget_constructors`, unused, etc.) | fix each; keep analyze clean |
| `setState() during build` | state change in build | move to callback/`addPostFrameCallback` |
| Missing `Key` warnings | list items without keys | add `ValueKey`/`key:` |
| Deprecated API info | old widget/param | migrate to current |

`flutter analyze` must report **No issues found** before done. `dart format` applied.

---

## Sensitivity rules (how hard to push)

- **Errors:** block everything. Fix before any further work.
- **Warnings:** fix in place this pass; don't accumulate. A warning you ignore today is tomorrow's crash.
- **Console/log noise:** a clean console is a feature. Remove stray logs; fix the source of every warning line.
- **Lint/analyze:** zero issues. Match the project's config; don't relax rules to "pass".
- **Suppression:** only when truly justified, scoped to the smallest unit, with a `// reason:` comment. Never project-wide disables to hide problems.
- **Deprecations:** treat as work items — migrate, don't suppress.

## Fix-in-place workflow

1. Read the **exact** error/warning text (file:line) — don't guess.
2. Identify root cause (not the symptom).
3. Apply the minimal correct fix at the source.
4. Re-run type-check + lint + app; confirm the message is gone and nothing new appeared.
5. Verify across states (dark, long text, empty, error) where relevant.

## Done-gate

- [ ] Compiles / type-checks with **zero errors**.
- [ ] Linter / `analyze` reports **zero warnings** (or each suppression is justified + commented).
- [ ] App runs with a **clean console** — no red, no yellow, no stray logs.
- [ ] No deprecated APIs left unmigrated.
- [ ] All states exercised without new warnings.
