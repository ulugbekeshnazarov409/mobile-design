# Design Tokens — copy-paste starter system

A complete, opinionated, premium default token set so you never hardcode magic numbers — accent ladder, tinted neutrals, type ramp, spacing, radius, elevation, motion — expressed in every framework. These are **sane premium defaults to adapt** (swap the indigo seed for the project's brand hue), not laws. If the project already has a token system, map onto its names; do not bolt a second one on.

> Cross-ref: `design-foundations.md` (the theory), `color-systems.md` (how to derive the ladder), `typography-systems.md` (voice + tabular figures), `frameworks/*.md` (wiring details).

---

## The token model — 3 layers

1. **Primitive** — raw values: `indigo500 = #6E56CF`, `space4 = 16`. No meaning, never referenced by UI.
2. **Semantic** — roles that map onto primitives, separately per mode: `background`, `accent`, `text.secondary`, `space.lg`. **Components reference only this layer.**
3. **Component** — optional, for repeated widgets: `button.height = 52`, `card.radius = radius.lg`. Built from semantic tokens.

UI code touches semantic/component tokens only. Re-theming = remap the semantic layer; components don't change.

---

## Semantic color palette (seed = indigo `#6E56CF`)

Off-black base (`#0B0B0F`, never `#000`), tinted neutrals (a whisper of indigo, not dead gray), one accent expanded into the ladder. Light step ≈ tonal 50–700, dark step ≈ 400–900.

| Role | Light | Dark | Use |
| --- | --- | --- | --- |
| `background` | `#FBFBFD` | `#0B0B0F` | app base |
| `surface` | `#FFFFFF` | `#15151B` | cards, sheets |
| `surfaceElevated` | `#FFFFFF` | `#1D1D26` | popovers, raised cards |
| `border` / hairline | `#E7E7EC` | `#2A2A35` | dividers, outlines |
| `text.primary` | `#16161D` | `#F4F4F7` | titles, body (~87%) |
| `text.secondary` | `#5B5B66` | `#A0A0AD` | supporting (~60%) |
| `text.tertiary` | `#8E8E99` | `#6B6B78` | hints, disabled (~38%) |
| `accent` | `#6E56CF` | `#8674E0` | primary fill, FAB |
| `accentSubtle` | `#EEEBFA` | `#211C38` | wash, selected row |
| `onAccent` | `#FFFFFF` | `#0B0B0F` | label on accent fill |
| `success` | `#1F9D55` | `#3FBE74` | confirm, gain |
| `successContainer` | `#E3F6EC` | `#13301F` | success banner bg |
| `warning` | `#C77700` | `#E8A13A` | caution |
| `warningContainer` | `#FCF1DE` | `#332205` | warning banner bg |
| `danger` | `#D6334B` | `#F0617A` | destructive, loss |
| `dangerContainer` | `#FBE4E8` | `#3A141C` | danger banner bg |

Dark accent is lifted + slightly desaturated so it doesn't vibrate; containers become dark tinted tones, not pale ones.

---

## Spacing scale (4-base)

`2, 4, 8, 12, 16, 20, 24, 32, 40, 48, 64`. Names: `xs=4 sm=8 md=12 base=16 lg=24 xl=32 2xl=48 3xl=64` (`2`=hairline gap, `20`=between-base-and-lg). Layout uses these only — no `13`, no `padding: 17`.

## Radius scale

`sm=8, md=12, lg=16, xl=24, full=999`. Cards `lg`, sheets/modals `xl`, chips/pills `full`, inputs/buttons `md`. Keep one family of curvature per screen.

## Elevation tiers (soft, layered, low-opacity)

Premium shadows are large-blur + low-alpha, often two stacked layers, tinted toward the brand — never a hard `0 2 4 #000`.

| Tier | Light shadow | Dark |
| --- | --- | --- |
| `e0` | none | none (use border) |
| `e1` | `0 1 2 rgba(16,16,29,.06)` | rely on `surfaceElevated` + border |
| `e2` | `0 4 12 rgba(16,16,29,.08)` | subtle `0 4 12 rgba(0,0,0,.4)` |
| `e3` | `0 12 32 rgba(16,16,29,.12)` | `0 12 32 rgba(0,0,0,.5)` |

In dark mode prefer tinted elevated surfaces + hairlines over visible shadows.

## Type ramp

Variable sans (Inter/Geist/SF/Roboto Flex). Tighten large, loosen small caps. Use tabular figures wherever numbers change (see `typography-systems.md §4`).

| Role | Size / LH | Weight | Tracking |
| --- | --- | --- | --- |
| `display` | 34 / 40 | 700 | −0.5 |
| `title` | 28 / 34 | 700 | −0.4 |
| `headline` | 20 / 26 | 600 | −0.2 |
| `body` | 16 / 24 | 400 | 0 |
| `bodyStrong` | 16 / 24 | 600 | 0 |
| `label` | 14 / 18 | 500 | +0.1 |
| `caption` | 12 / 16 | 500 | +0.4 (often uppercase) |

## Motion tokens

Durations `fast=120, base=200, slow=300` ms. Easing: standard `cubic-bezier(0.2, 0, 0, 1)` (emphasized-decelerate), exit `cubic-bezier(0.4, 0, 1, 1)`. Springs for gesture-driven UI: ~`stiffness 300, damping 30` (snappy) / `stiffness 180, damping 26` (gentle). Respect reduce-motion.

---

## React Native / Expo — `theme.ts`

```ts
export const palette = {
  light: {
    background: '#FBFBFD', surface: '#FFFFFF', surfaceElevated: '#FFFFFF',
    border: '#E7E7EC',
    textPrimary: '#16161D', textSecondary: '#5B5B66', textTertiary: '#8E8E99',
    accent: '#6E56CF', accentSubtle: '#EEEBFA', onAccent: '#FFFFFF',
    success: '#1F9D55', successContainer: '#E3F6EC',
    warning: '#C77700', warningContainer: '#FCF1DE',
    danger: '#D6334B', dangerContainer: '#FBE4E8',
  },
  dark: {
    background: '#0B0B0F', surface: '#15151B', surfaceElevated: '#1D1D26',
    border: '#2A2A35',
    textPrimary: '#F4F4F7', textSecondary: '#A0A0AD', textTertiary: '#6B6B78',
    accent: '#8674E0', accentSubtle: '#211C38', onAccent: '#0B0B0F',
    success: '#3FBE74', successContainer: '#13301F',
    warning: '#E8A13A', warningContainer: '#332205',
    danger: '#F0617A', dangerContainer: '#3A141C',
  },
} as const;

export const space = { xs: 4, sm: 8, md: 12, base: 16, lg: 24, xl: 32, '2xl': 48, '3xl': 64 } as const;
export const radius = { sm: 8, md: 12, lg: 16, xl: 24, full: 999 } as const;
export const duration = { fast: 120, base: 200, slow: 300 } as const;

export const type = {
  display:    { fontSize: 34, lineHeight: 40, fontWeight: '700', letterSpacing: -0.5 },
  title:      { fontSize: 28, lineHeight: 34, fontWeight: '700', letterSpacing: -0.4 },
  headline:   { fontSize: 20, lineHeight: 26, fontWeight: '600', letterSpacing: -0.2 },
  body:       { fontSize: 16, lineHeight: 24, fontWeight: '400' },
  bodyStrong: { fontSize: 16, lineHeight: 24, fontWeight: '600' },
  label:      { fontSize: 14, lineHeight: 18, fontWeight: '500', letterSpacing: 0.1 },
  caption:    { fontSize: 12, lineHeight: 16, fontWeight: '500', letterSpacing: 0.4 },
} as const;

export const elevation = {
  e1: { shadowColor: '#10101D', shadowOpacity: 0.06, shadowRadius: 2,  shadowOffset: { width: 0, height: 1 },  elevation: 1 },
  e2: { shadowColor: '#10101D', shadowOpacity: 0.08, shadowRadius: 12, shadowOffset: { width: 0, height: 4 },  elevation: 4 },
  e3: { shadowColor: '#10101D', shadowOpacity: 0.12, shadowRadius: 32, shadowOffset: { width: 0, height: 12 }, elevation: 12 },
} as const;
```

```ts
// useTheme hook — switch the semantic map on colorScheme
import { useColorScheme } from 'react-native';
import { palette, space, radius, type, elevation, duration } from './theme';

export function useTheme() {
  const scheme = useColorScheme() ?? 'light';
  return { colors: palette[scheme], scheme, space, radius, type, elevation, duration };
}
// const t = useTheme(); <View style={{ padding: t.space.base, backgroundColor: t.colors.surface }} />
```

### NativeWind — `tailwind.config.js`

```js
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: ['./app/**/*.{ts,tsx}', './components/**/*.{ts,tsx}'],
  darkMode: 'class', // toggle a top-level `dark` class from useColorScheme
  theme: {
    extend: {
      colors: {
        background: { DEFAULT: '#FBFBFD', dark: '#0B0B0F' },
        surface:    { DEFAULT: '#FFFFFF', dark: '#15151B', elevated: '#1D1D26' },
        border:     { DEFAULT: '#E7E7EC', dark: '#2A2A35' },
        accent:     { DEFAULT: '#6E56CF', dark: '#8674E0', subtle: '#EEEBFA' },
        success: '#1F9D55', warning: '#C77700', danger: '#D6334B',
      },
      spacing: { xs: '4px', sm: '8px', md: '12px', base: '16px', lg: '24px', xl: '32px', '2xl': '48px', '3xl': '64px' },
      borderRadius: { sm: '8px', md: '12px', lg: '16px', xl: '24px', full: '999px' },
    },
  },
};
```

---

## Jetpack Compose (M3)

```kotlin
// Color.kt
import androidx.compose.ui.graphics.Color

val Indigo        = Color(0xFF6E56CF)
val IndigoDark    = Color(0xFF8674E0)

// Color.kt — light
val md_background      = Color(0xFFFBFBFD)
val md_surface         = Color(0xFFFFFFFF)
val md_surfaceVariant  = Color(0xFFEEEBFA) // accentSubtle / container role
val md_outline         = Color(0xFFE7E7EC)
val md_onSurface       = Color(0xFF16161D)
val md_onSurfaceVar    = Color(0xFF5B5B66)
val md_error           = Color(0xFFD6334B)
// dark
val mdd_background     = Color(0xFF0B0B0F)
val mdd_surface        = Color(0xFF15151B)
val mdd_surfaceVariant = Color(0xFF211C38)
val mdd_outline        = Color(0xFF2A2A35)
val mdd_onSurface      = Color(0xFFF4F4F7)
val mdd_onSurfaceVar   = Color(0xFFA0A0AD)
val mdd_error          = Color(0xFFF0617A)
```

```kotlin
// Theme.kt
import androidx.compose.material3.*
import androidx.compose.runtime.Composable
import androidx.compose.foundation.shape.RoundedCornerShape
import androidx.compose.ui.unit.dp

private val Light = lightColorScheme(
    primary = Indigo, onPrimary = Color.White, primaryContainer = md_surfaceVariant,
    background = md_background, surface = md_surface, surfaceVariant = md_surfaceVariant,
    onBackground = md_onSurface, onSurface = md_onSurface, onSurfaceVariant = md_onSurfaceVar,
    outline = md_outline, error = md_error,
)
private val Dark = darkColorScheme(
    primary = IndigoDark, onPrimary = Color(0xFF0B0B0F), primaryContainer = mdd_surfaceVariant,
    background = mdd_background, surface = mdd_surface, surfaceVariant = mdd_surfaceVariant,
    onBackground = mdd_onSurface, onSurface = mdd_onSurface, onSurfaceVariant = mdd_onSurfaceVar,
    outline = mdd_outline, error = mdd_error,
)

val AppShapes = Shapes(
    small  = RoundedCornerShape(8.dp),
    medium = RoundedCornerShape(12.dp),
    large  = RoundedCornerShape(16.dp),
)

@Composable
fun AppTheme(dark: Boolean = androidx.compose.foundation.isSystemInDarkTheme(), content: @Composable () -> Unit) =
    MaterialTheme(colorScheme = if (dark) Dark else Light, typography = AppTypography, shapes = AppShapes, content = content)
```

```kotlin
// Type.kt — tighten large, tabular figures on numerics
import androidx.compose.material3.Typography
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.unit.sp

val AppTypography = Typography(
    displaySmall = TextStyle(fontSize = 34.sp, lineHeight = 40.sp, fontWeight = FontWeight.Bold,     letterSpacing = (-0.5).sp),
    headlineMedium = TextStyle(fontSize = 28.sp, lineHeight = 34.sp, fontWeight = FontWeight.Bold,   letterSpacing = (-0.4).sp),
    titleLarge = TextStyle(fontSize = 20.sp, lineHeight = 26.sp, fontWeight = FontWeight.SemiBold,   letterSpacing = (-0.2).sp),
    bodyLarge  = TextStyle(fontSize = 16.sp, lineHeight = 24.sp, fontWeight = FontWeight.Normal),
    labelLarge = TextStyle(fontSize = 14.sp, lineHeight = 18.sp, fontWeight = FontWeight.Medium,     letterSpacing = 0.1.sp),
    labelSmall = TextStyle(fontSize = 12.sp, lineHeight = 16.sp, fontWeight = FontWeight.Medium,     letterSpacing = 0.4.sp),
)
// numerics: TextStyle(fontFeatureSettings = "tnum")
```

```kotlin
// Spacing.kt — expose via a small object (or a CompositionLocal for theming)
object Space { val xs=4.dp; val sm=8.dp; val md=12.dp; val base=16.dp; val lg=24.dp; val xl=32.dp; val xxl=48.dp }
```

---

## SwiftUI

Prefer **asset-catalog color sets** (Any/Dark appearance per role) so the system handles dark mode; this extension names them.

```swift
import SwiftUI

extension Color {
    // Map each to an asset color set with Light + Dark variants.
    static let appBackground      = Color("background")       // #FBFBFD / #0B0B0F
    static let appSurface         = Color("surface")          // #FFFFFF / #15151B
    static let appSurfaceElevated = Color("surfaceElevated")  // #FFFFFF / #1D1D26
    static let appBorder          = Color("border")           // #E7E7EC / #2A2A35
    static let textPrimary        = Color("textPrimary")      // #16161D / #F4F4F7
    static let textSecondary      = Color("textSecondary")    // #5B5B66 / #A0A0AD
    static let textTertiary       = Color("textTertiary")     // #8E8E99 / #6B6B78
    static let accent             = Color("AccentColor")      // #6E56CF / #8674E0
    static let accentSubtle       = Color("accentSubtle")     // #EEEBFA / #211C38
    static let success            = Color("success")
    static let warning            = Color("warning")
    static let danger             = Color("danger")
}

enum Theme {
    enum Space  { static let xs:CGFloat=4, sm=8, md=12, base=16, lg=24, xl=32, xxl=48 }
    enum Radius { static let sm:CGFloat=8, md=12, lg=16, xl=24, full=999 }
    enum Dur    { static let fast=0.12, base=0.20, slow=0.30 }

    struct Font {
        static let display  = SwiftUI.Font.system(size: 34, weight: .bold).leading(.tight)
        static let title    = SwiftUI.Font.system(size: 28, weight: .bold)
        static let headline = SwiftUI.Font.system(size: 20, weight: .semibold)
        static let body     = SwiftUI.Font.system(size: 16, weight: .regular)
        static let label    = SwiftUI.Font.system(size: 14, weight: .medium)
        static let caption  = SwiftUI.Font.system(size: 12, weight: .medium)
    }
}
// shadow e2: .shadow(color: .black.opacity(0.08), radius: 12, x: 0, y: 4)
// numerics: Text(amount).monospacedDigit()
// .tint(.accent); animate with .animation(.easeOut(duration: Theme.Dur.base), value: x)
```

---

## Flutter (Material 3)

```dart
import 'package:flutter/material.dart';

const _seed = Color(0xFF6E56CF);

ThemeData appTheme(Brightness b) {
  final scheme = ColorScheme.fromSeed(seedColor: _seed, brightness: b).copyWith(
    surface: b == Brightness.light ? const Color(0xFFFFFFFF) : const Color(0xFF15151B),
    error:   b == Brightness.light ? const Color(0xFFD6334B) : const Color(0xFFF0617A),
  );
  return ThemeData(
    useMaterial3: true,
    colorScheme: scheme,
    scaffoldBackgroundColor: b == Brightness.light ? const Color(0xFFFBFBFD) : const Color(0xFF0B0B0F),
    extensions: const [AppTokens()],
    textTheme: const TextTheme(
      displaySmall:   TextStyle(fontSize: 34, height: 40/34, fontWeight: FontWeight.w700, letterSpacing: -0.5),
      headlineMedium: TextStyle(fontSize: 28, height: 34/28, fontWeight: FontWeight.w700, letterSpacing: -0.4),
      titleLarge:     TextStyle(fontSize: 20, height: 26/20, fontWeight: FontWeight.w600, letterSpacing: -0.2),
      bodyLarge:      TextStyle(fontSize: 16, height: 24/16, fontWeight: FontWeight.w400),
      labelLarge:     TextStyle(fontSize: 14, height: 18/14, fontWeight: FontWeight.w500, letterSpacing: 0.1),
      labelSmall:     TextStyle(fontSize: 12, height: 16/12, fontWeight: FontWeight.w500, letterSpacing: 0.4),
    ),
  );
}

@immutable
class AppTokens extends ThemeExtension<AppTokens> {
  const AppTokens();
  // spacing
  double get xs => 4; double get sm => 8;  double get md => 12; double get base => 16;
  double get lg => 24; double get xl => 32; double get xxl => 48;
  // radius
  Radius get rSm => const Radius.circular(8);  Radius get rMd => const Radius.circular(12);
  Radius get rLg => const Radius.circular(16); Radius get rXl => const Radius.circular(24);
  // motion
  Duration get fast => const Duration(milliseconds: 120);
  Duration get base_ => const Duration(milliseconds: 200);
  @override AppTokens copyWith() => const AppTokens();
  @override AppTokens lerp(ThemeExtension<AppTokens>? o, double t) => this;
}
// MaterialApp(theme: appTheme(Brightness.light), darkTheme: appTheme(Brightness.dark));
// read: final t = Theme.of(context).extension<AppTokens>()!;  Padding(padding: EdgeInsets.all(t.base))
// tabular figures: TextStyle(fontFeatures: [FontFeature.tabularFigures()])
```

---

## Quality checklist (tokens)

- [ ] No raw hex / magic number in UI code — components reference semantic (or component) tokens only.
- [ ] Seed swapped to the real brand hue; whole ladder + neutrals re-derived (`color-systems.md`), not hand-edited.
- [ ] Neutrals are tinted toward the brand; off-white / off-black bases, never `#FFF` / `#000`.
- [ ] Light **and** dark both defined; dark accent lifted+desaturated; dark containers are dark tinted tones.
- [ ] Spacing from the 4-base scale; radius from one family; shadows soft/layered/low-alpha (or hairlines in dark).
- [ ] Type ramp wired with weight+tracking+line-height per role; tabular figures on changing numbers.
- [ ] Motion uses the duration/easing tokens; reduce-motion respected.
- [ ] If the project already had tokens, these were **mapped onto its names**, not duplicated.
```