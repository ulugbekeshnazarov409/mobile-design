# Project Awareness — design to fit the project, not a blank slate

The single most important rule of this skill: **inspect the project before generating UI, and conform to it.** A great component that ignores the repo's framework, styling system, tokens, and conventions is a bad component. Match the codebase; reuse what exists; extend it cleanly.

Run this pre-flight whenever the task touches an existing repo.

---

## 1. Detect the framework & platform

| Signal (files) | Stack | Reference |
| --- | --- | --- |
| `build.gradle(.kts)`, `*.kt`, `androidx.compose.*` | Jetpack Compose | `frameworks/jetpack-compose.md` |
| `*.xcodeproj` / `Package.swift`, `*.swift`, `import SwiftUI` | SwiftUI | `frameworks/swiftui.md` |
| `pubspec.yaml`, `lib/*.dart` | Flutter | `frameworks/flutter.md` |
| `package.json` with `react-native` / `expo` | React Native / Expo | `frameworks/react-native-expo.md` |

If multiple (e.g. a monorepo), ask which app, or match the directory you were pointed at.

---

## 2. Detect the styling / theming system (then obey it)

Don't impose a styling approach — find the one in use and write in it.

- **React Native:** `nativewind` (+`tailwind.config.js`/`global.css`) → className-only Tailwind. `tamagui` → styled+tokens. `@shopify/restyle` → theme box/text. `dripsy` → `sx`. Otherwise `StyleSheet` + a token module. (Table in `frameworks/react-native-expo.md`.)
- **Compose:** is there a custom `Theme.kt` / `Color.kt` / `Type.kt`? Use those tokens. Material2 vs Material3 — match the imports already present.
- **SwiftUI:** is there a `Theme`/`Tokens`/asset `Color` set? Use it. Otherwise system semantic colors.
- **Flutter:** existing `ThemeData` / `theme.dart` / `AppColors`? Use it. M2 vs M3 — match `useMaterial3`.

**Grep the existing screens** to see the real convention (how they import colors, how they space, how they name files) and copy it.

---

## 3. Find and reuse existing design tokens & components

Before writing anything new:
1. Locate the token source — `tailwind.config`, `Theme.kt`, `theme.dart`, `tokens.ts`, an `Assets.xcassets` color set, a Figma-exported token file.
2. Locate existing components — a `components/` / `ui/` / `widgets/` folder, a design-system package. **Reuse or extend these**; don't fork a parallel button.
3. Match naming & structure — file layout, naming (`PrimaryButton` vs `AppButton`), export style, prop patterns.

If the repo has a design-system package or a skill for it (e.g. `nativewind-ui`, `cascadia-mobile-ui`), defer to that skill's rules.

> The auto-memory may name the active repo/convention (e.g. a NativeWind className-only rule). Honor it, but verify the file/flag still exists before relying on it.

---

## 4. Add-package policy

Prefer what's installed. When a real capability gap exists (e.g. no bottom-sheet lib for a sheet-heavy screen):
1. Check if an installed dep already covers it.
2. If not, propose the **standard** package for that ecosystem + the exact install command, and explain why.
3. Use the ecosystem's installer: `npx expo install …` (Expo), `flutter pub add …` (Flutter), Gradle/version-catalog entry (Android), SPM/`Package.swift` (iOS).
4. Note any config wiring (babel/metro plugin, `Info.plist`, Gradle, pod install) so the build doesn't break.
5. Don't pull in heavy deps for something the platform already does well.

---

## 5. Conform to project conventions

- File/folder structure, naming, and component granularity: match the repo.
- State/data patterns already in use (hooks, ViewModel, Riverpod/Bloc, Redux/Zustand) — follow them; don't introduce a new one for a UI task.
- Lint/format config (ESLint/Prettier/ktlint/SwiftLint/`dart format`) — write code that passes it.
- Accessibility/i18n hooks the app already uses (string tables, `Semantics`, `accessibilityLabel`).

---

## 6. Pre-flight checklist (run before generating UI)

- [ ] Framework identified from project files.
- [ ] Styling system identified — I will write in it, not replace it.
- [ ] Existing tokens located and used (colors, spacing, type, radii).
- [ ] Existing components located — reused/extended, not duplicated.
- [ ] Naming/structure/state patterns matched.
- [ ] Any new package justified, installed the ecosystem way, wiring noted.
- [ ] Dark mode + safe areas handled the way the app already does.

Only after this, build from `components.md` / `screen-patterns.md` with `design-foundations.md` tokens — mapped onto the project's actual token names.
