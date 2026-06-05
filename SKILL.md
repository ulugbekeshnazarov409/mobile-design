---
name: mobile-design
description: >-
  Design-grade mobile UI skill. Builds polished, production-quality, native-feeling
  mobile screens and components across Jetpack Compose (Kotlin), SwiftUI (Swift),
  Flutter (Dart), and React Native / Expo. Grounded in Google Material Design 3,
  Apple Human Interface Guidelines, and real-world app layout patterns (Mobbin-style
  flows). Use for ANY task that designs, styles, or builds a mobile screen, component,
  or design system — "build a chat UI like Telegram", "make this Compose screen look
  premium", "design an onboarding flow", "SwiftUI settings screen", "Flutter card",
  "RN bottom sheet", "mobile design tokens", "make it look native not AI". Triggers —
  mobile ui, mobile design, jetpack compose, kotlin ui, swiftui, swift ui, flutter
  ui, react native ui, expo ui, material 3, material you, apple hig, design tokens,
  bottom sheet, bottom nav, app bar, fab, onboarding, chat screen, feed, profile
  screen, settings screen, native mobile design.
---

# Mobile Design

Build mobile UI that looks **intentional, native, and shipped** — like screens you'd find on Mobbin from Linear, Revolut, Telegram, Airbnb — not generic AI/Bootstrap mush.

This skill spans four target stacks. Pick the user's framework, then apply the same design judgment to all of them:

| Platform | Framework | Design system |
| --- | --- | --- |
| Android | Jetpack Compose (Kotlin) | Material Design 3 |
| iOS | SwiftUI (Swift) | Apple Human Interface Guidelines |
| Cross-platform | Flutter (Dart) | Material 3 + Cupertino |
| Cross-platform | React Native / Expo | Adapt M3/HIG per-OS |

## Two modes

- **Build mode** — create/style a screen or component (default).
- **Review / lint mode** — audit existing UI code for design problems and optionally refactor. Trigger on "review/lint my UI", "is this design correct", "fix the design issues", "make this match guidelines". Use `references/design-review.md`.

## Decision flow — DETECT → DECIDE → PLAN (do this before writing any UI)

Never start coding UI blind. First work out *what language/framework the code is in*, then *which UI toolkit + styling system that implies*, then *state the plan*. This is a hard gate, not optional.

### Step A — DETECT the language & framework (from the repo, not assumptions)

Look at the actual project files and infer the stack. Signals → conclusion:

| Files / signals found | Language | Framework | UI toolkit |
| --- | --- | --- | --- |
| `*.kt`, `build.gradle.kts`, `androidx.compose.*` | Kotlin | Jetpack Compose | Material 3 |
| `*.kt`/`*.java`, `res/layout/*.xml` (no Compose) | Kotlin/Java | Android Views (XML) | Material Components |
| `*.swift`, `import SwiftUI` | Swift | SwiftUI | Apple HIG |
| `*.swift`, `*.storyboard`/`*.xib` | Swift | UIKit | Apple HIG |
| `*.dart`, `pubspec.yaml` | Dart | Flutter | Material 3 / Cupertino |
| `package.json` + `react-native`/`expo`, `*.tsx` | TS/JS | React Native / Expo | (see Step B) |

If no repo (greenfield) or genuinely ambiguous after looking → **ask one question**: which platform/framework. Don't guess between Compose and SwiftUI.

### Step B — DECIDE the UI & styling approach the detected stack dictates

The framework alone doesn't fully decide *how* to style — especially React Native. Inspect deeper and pick the approach the project already uses:

- **Compose** → M3 components + the repo's `Theme.kt`/`Color.kt`/`Type.kt` tokens. (M2 vs M3: match existing imports.)
- **SwiftUI** → system components + semantic colors / any `Theme`/asset color set in the repo.
- **Flutter** → `ThemeData` (`useMaterial3`?) tokens; Material vs Cupertino per platform intent.
- **React Native / Expo** → **detect the UI/styling library and lock into it for the whole task** (this is the big one — see `references/frameworks/react-native-ui-libraries.md` for the full catalog + per-library cheat-sheets):
  - `nativewind` + `tailwind.config.js` → **className-only Tailwind**, no `StyleSheet`.
  - `react-native-paper` → Paper components + Paper theme. `tamagui` → `styled()` + tokens. `@rneui/themed` → RN Elements. `@ui-kitten` → Eva. `@gluestack-ui` → gluestack. `@shopify/restyle` → typed theme box/text. `dripsy` → `sx`. `react-native-unistyles` → themed StyleSheet.
  - none → `StyleSheet.create` + a token module (or suggest a fitting lib if warranted).
  - **One system per app, used to the end** — never mix two (no raw StyleSheet in a Tamagui app, no Paper button in a NativeWind screen).
  - Also detect companions (`@gorhom/bottom-sheet`, FlashList, reanimated/moti), nav (Expo Router vs React Navigation), existing `components/ui` folder, and the design-token source — reuse them.

Full detection tables: `references/project-awareness.md`, `references/frameworks/react-native-expo.md`, `references/frameworks/react-native-ui-libraries.md`.

### Step C — PLAN out loud before coding (one short block)

Once detected and decided, state a brief plan so the user sees the chosen path before any code. Template:

```
Stack:    <language> / <framework>   (detected from <signal>)
Styling:  <system in use>            (e.g. NativeWind className-only)
Tokens:   <reuse existing X / define from foundations>
Screen:   <pattern from screen-patterns.md, e.g. chat scaffold>
Components: <list, reused-or-new>
Nav/deps: <existing | add <pkg> via <installer> because <gap>>
States:   <empty / loading / error to include>
```

Keep it 5–7 lines, then build. For a tiny one-component ask, a one-line version is fine — but still name the stack + styling system explicitly. **The point: the user always knows which language, which UI toolkit, which styling system, and what will be built, before code appears.**

## How to use this skill (build mode)

0. **Project pre-flight (do this first on any existing repo).** Load `references/project-awareness.md`. Detect the framework, the **styling system in use** (e.g. RN with NativeWind → className-only; Tamagui; restyle; or StyleSheet), the existing tokens, and existing components — then **conform to them**. Reuse tokens/components; don't impose a different system. Add a package only when there's a real gap, the ecosystem way (`npx expo install`, `flutter pub add`, etc.).
1. **Identify the framework.** If the user named one (Compose, SwiftUI, Flutter, RN/Expo) use it. If the repo makes it obvious (build.gradle.kts → Compose; *.xcodeproj/Package.swift → SwiftUI; pubspec.yaml → Flutter; package.json + react-native → RN), match it. If genuinely ambiguous, ask once.
2. **Read the foundations.** Load `references/design-foundations.md` for the token system (spacing, type, color, shape, elevation, motion), and `references/design-to-code.md` for token architecture, pixel-perfect handoff, and mobile UX laws (thumb zone, touch targets). Every screen starts from tokens — mapped onto the project's actual token names — never hardcoded magic numbers.
3. **Scope: screen or flow?** If the request implies a multi-step task ("checkout", "sign up", "send money", "onboarding", "booking"), you're building a **flow**, not one screen. Load `references/flow-architecture.md` — map the screen sequence, branches, carried state, and navigation; confirm scope; then build the whole journey. For any single screen, you'll still cover all its states in step 6.
4. **Decide hierarchy & density (do this before laying out).** Load `references/visual-hierarchy.md` — rank every element Primary/Secondary/Tertiary, pick the one primary action, apply the text opacity ramp. Then `references/screen-density.md` — choose the density that fits the screen's job (dense chat vs airy paywall) and use *hierarchical* spacing, not even gaps. These two fix the #1 and #2 AI failures. Use `references/psychology.md` to place the primary element on the eye's scan path (F/Z), use the strongest attention anchor, and group by proximity/alignment. For a premium reference point, pull the closest app's DNA from `references/case-studies.md` ("make it like Linear/Revolut/Telegram…").
5. **Identify the screen pattern.** Load `references/screen-patterns.md` to map the request ("chat", "feed", "onboarding", "profile", "settings", "paywall") to a proven layout skeleton.
6. **Build — every state, not just the happy path.** Load `references/components.md` for cross-framework component recipes, the matching `references/frameworks/<framework>.md` for idioms/navigation/theming, and `references/interaction-patterns.md` to enumerate and build **all** of the screen's states (initial/loading/empty/error/offline/submitting/success/disabled) and its exits — the happy path is ~30% of the work.
7. **Restraint + accessibility pass (mandatory).** Run `references/ui-restraint-and-accessibility.md`: cut anything that didn't earn its place (the most common failure is *too much* — text/icons/badges/colors breaking the UI), verify 1:1 fidelity, confirm every element uses the right semantic component with required props + a11y (labels, roles, states, ≥48dp/44pt targets).
8. **Premium polish pass.** Run `references/premium-polish.md` to push from "fine" to top-studio tier: kill AI-slop tells (off-black/white, text ramp, single accent, soft layered shadows), apply optical adjustments, tune typography, true dark mode, and one or two signature moves. Add **motion + tactile feel** from `references/motion-recipes.md` (timing/spring tiers, press scale, choreography, **every enter gets an exit**, hero/numeric/skeleton recipes) and `references/gestures-and-haptics.md` (swipe/drag/pull-refresh + the haptic-by-action map; gesture+motion+haptic trinity). Scan `references/anti-patterns.md` and remove any hits.
9. **Diagnostics pass (zero-tolerance).** Run `references/error-handling-and-diagnostics.md`: type-check + lint/`analyze` + run, and **fix every error AND warning in place** — read the exact message, fix the root cause, re-run until compile is clean, lint/analyze is zero, and the console/logcat is clean (no red, no yellow, no stray logs). Warnings are treated as bugs.
10. **Critique + score (final gate).** Run `references/design-critique-engine.md`: critique your own screen like a senior designer (the 8 ship-test questions + dimension-by-dimension), then **REVISE** the defects — at least one full critique→revise cycle. Then grade it with `references/premium-scorecard.md`; any dimension below threshold → revise that dimension and re-score (accessibility + diagnostics must be 10). Report the scorecard. Only ship at "premium". Self-check against the Quality bar below.

> Progressive disclosure: keep this file in context, open reference files only when the task needs them. Don't dump all references for a one-component ask.

## Design philosophy — read before writing any UI

Most AI-generated mobile UI fails the same way: flat, evenly-spaced, default-colored, no hierarchy, no motion, ignores platform conventions. Avoid every one of these.

**1. Hierarchy over uniformity.** A screen has one primary action, a few secondary ones, and supporting content. Make that visible through size, weight, color, and spacing — not by making everything the same.

**2. Spacing is a system, not a guess.** Use a 4dp/4pt base grid (4, 8, 12, 16, 24, 32, 48). Group related elements tight, separate unrelated ones wide. White space is a design tool, not waste.

**3. Type scale, not random sizes.** Use the platform type ramp (M3 type roles / iOS text styles). Two or three sizes per screen, differentiated by weight and color, beats five arbitrary sizes.

**4. Color with restraint.** One brand/accent color used deliberately. Neutral surfaces carry most of the UI. Derive on-colors, containers, and states from the role system (M3 color roles / HIG semantic colors) — never pick raw hex per element. Always ship light **and** dark.

**5. Respect the platform.** iOS feels like iOS (SF font, large titles, swipe-back, grouped lists, bottom sheets with grabbers). Android feels like Android (Material components, FABs, ripple, top app bar). Cross-platform code should adapt, not pick one and force it everywhere.

**6. Touch targets & ergonomics.** Minimum 48×48dp (Android) / 44×44pt (iOS) hit area. Primary actions reachable by thumb (bottom of screen). Respect safe areas / notches / nav bars.

**7. Motion with purpose.** Use the platform's standard easing and durations. Animate state changes (press, expand, navigate), not decoration. Default to subtle: 150–300ms, ease-out for entrances.

**8. Real content, real states.** Design for long names, empty lists, loading, and errors — not just the happy path with perfect placeholder data. Include skeletons/empty states when building a full screen.

## Quality bar — self-check before declaring done

- [ ] Spacing comes from the grid scale; no stray magic numbers.
- [ ] Type uses the platform scale; clear primary/secondary/tertiary hierarchy.
- [ ] Color uses semantic roles; one deliberate accent; light + dark both work.
- [ ] Components match platform conventions (right component for the OS).
- [ ] Touch targets ≥ 48dp/44pt; safe-area insets respected.
- [ ] At least the obvious state changes are animated with platform easing.
- [ ] Empty/loading/error states considered for full screens.
- [ ] **Hierarchy:** at 0.5s one element wins the eye; elements ranked Primary/Secondary/Tertiary; text uses the opacity ramp. (The #1 AI failure.)
- [ ] **Density:** chosen to fit the screen's job; spacing is hierarchical (related tight, groups wide), not even gaps; item weight varies.
- [ ] **Restraint:** nothing on screen that didn't earn its place; one primary action, one accent, 2–3 type sizes; long text truncates; whitespace intentional. (Most AI UI fails by doing *too much*.)
- [ ] **No anti-patterns:** scanned `anti-patterns.md`, zero hits.
- [ ] **Zero diagnostics:** compiles with no errors; lint/`analyze` clean; app runs with a clean console (no red, no yellow, no stray logs); every warning fixed at the source, not suppressed.
- [ ] **Flow-aware:** if multi-step, the whole journey is built (screens + nav + carried state + exits), not one screen.
- [ ] **All states built:** loading/empty/error/offline/submitting/success/disabled — not just the happy path; no dead-ends or double-submit.
- [ ] **Motion + tactile:** press feedback + haptics on commits; transitions choreographed; every enter has an exit; Reduce Motion handled.
- [ ] **Critiqued + scored:** ran the anti-gaming sequence (forced-failure search + adversarial designer/engineer review) → ≥1 critique→revise cycle; premium scorecard passes with **evidence per dimension** (all dims ≥ threshold; a11y + diagnostics = 10); remaining items are stated taste trade-offs.
- [ ] Every element uses the right semantic component with required props + accessibility (label/role/state); 1:1 fidelity verified at small width, largest text scale, and dark mode.
- [ ] Code is idiomatic for the framework (composables/views/widgets/components named and structured the way that ecosystem expects) and reusable, not one giant function.
- [ ] (RN) One UI/styling library, used to the end — no mixing systems.

## Reference map

- `references/project-awareness.md` — **Pre-flight.** Detect the repo's framework, styling system, tokens, and components; conform to them; add-package policy. Run first on any existing repo.
- `references/design-foundations.md` — Tokens: spacing, type ramps, color roles (M3 + HIG), shape, elevation, motion.
- `references/design-to-code.md` — Token architecture (primitive→semantic→component), pixel-perfect handoff, mobile UX laws (thumb zone, touch targets, navigation), clean component code.
- `references/visual-hierarchy.md` — **The #1 AI fix.** Rank Primary/Secondary/Tertiary, the 3-second rule, text opacity ramp, one-primary-action, per-pattern hierarchy.
- `references/screen-density.md` — Density spectrum (dense↔airy) per screen job, spacing as hierarchical rhythm, anti-uniformity moves.
- `references/case-studies.md` — *Why* top apps feel premium: Linear, Revolut, Telegram, Airbnb, Notion, Stripe, Arc, Coinbase, Duolingo — hierarchy/density/radius/shadow/motion/color signatures + per-app AI pitfalls. Use for "make it like X".
- `references/anti-patterns.md` — The AI-UI blocklist: what NOT to do (slop tells), scanned in the polish pass.
- `references/error-handling-and-diagnostics.md` — **Zero-tolerance diagnostics.** Per-framework common error/warning/log → fix tables (Compose, SwiftUI, Flutter, RN/Expo), the diagnostics loop, and the fix-in-place workflow. Warnings treated as bugs.
- `references/design-critique-engine.md` — **Self-review gate.** The 8 ship-test questions ("would Linear/Apple ship this?"), dimension-by-dimension critique, and the BUILD→CRITIQUE→REVISE loop.
- `references/premium-scorecard.md` — **Measurable quality.** 0–10 rubric per dimension with weights + thresholds; revise any sub-threshold dimension; report format. A11y + diagnostics must be 10.
- `references/ui-restraint-and-accessibility.md` — **Mandatory final pass.** Restraint/anti-overcrowd, density & whitespace, 1:1 pixel fidelity, "what's underneath" component analysis, accessibility props per framework (label/role/state, hit area, dynamic type, contrast).
- `references/premium-polish.md` — **Premium tier.** Anti-AI-slop tells + fixes, optical adjustments, micro-interactions & haptics, motion choreography, layered depth/materials, typography craft, true dark mode, signature moves, performance-as-polish.
- `references/psychology.md` — Perception laws: F/Z scan patterns, attention anchors, Gestalt grouping, cognitive load (Hick/Miller/Jakob), Fitts/thumb zone, perceived performance. Diagnose "feels off".
- `references/screen-patterns.md` — Mobbin-style screen skeletons: onboarding, auth, feed/list, detail, chat, profile, settings, search, checkout/paywall, tab scaffold. Layout anatomy per pattern.
- `references/interaction-patterns.md` — **Screens are state machines.** All states per screen (loading/empty/error/offline/submitting/success/disabled), per-flow state recipes (auth/forms/checkout/chat/search), classic flow-bug prevention.
- `references/flow-architecture.md` — **Build the journey, not one screen.** Canonical flows (booking/messaging/transfer/onboarding/checkout) with sequence, branches, carried state, navigation shape.
- `references/motion-recipes.md` — Premium *feel*: timing tiers, spring personalities, press/choreography/sheet/hero/numeric/skeleton recipes per framework, **enter↔exit pairing**, Reduce Motion.
- `references/gestures-and-haptics.md` — Haptic-by-action map + swipe/drag/pull-refresh/long-press/reorder recipes per framework; the gesture+motion+haptic trinity.
- `references/components.md` — Cross-framework component catalog: buttons, text fields, cards, chips, list items, app bars, bottom nav/tab bar, FAB, bottom sheet, dialogs, avatars, badges.
- `references/design-review.md` — **Review/lint mode.** Audit checklist + report format for finding and refactoring design issues in existing UI code.
- `references/frameworks/jetpack-compose.md` — Compose + M3 idioms, theming, navigation, animation.
- `references/frameworks/swiftui.md` — SwiftUI + HIG idioms, theming, navigation, animation.
- `references/frameworks/flutter.md` — Flutter Material 3 + Cupertino, theming, navigation, animation.
- `references/frameworks/react-native-expo.md` — RN/Expo, **project styling detection** (NativeWind/Tamagui/restyle/StyleSheet), Expo internals, package policy, per-OS adaptation.
- `references/frameworks/react-native-ui-libraries.md` — Full RN UI-library catalog: detect → lock into one (NativeWind, Paper, Tamagui, RN Elements, UI Kitten, gluestack, Restyle, Dripsy, Unistyles, vanilla) + per-library theming/component cheat-sheets + companions.
