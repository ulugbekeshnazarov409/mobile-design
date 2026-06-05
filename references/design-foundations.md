# Design Foundations — Tokens

Every screen is built from tokens, never ad-hoc values. This file is the source of truth for spacing, type, color, shape, elevation, and motion across all four frameworks. Map these to whatever the framework calls them (Theme, ThemeData, Color roles, design-token objects).

---

## 1. Spacing — 4dp/4pt base grid

Use multiples of 4. These are the only spacing values you should reach for:

| Token | Value | Typical use |
| --- | --- | --- |
| `space.0` | 0 | reset |
| `space.1` | 4 | icon-to-label, tight inline gaps |
| `space.2` | 8 | inside chips/badges, compact list gaps |
| `space.3` | 12 | between related items |
| `space.4` | 16 | **default screen/card padding**, list row padding |
| `space.5` | 20 | comfortable section inner padding |
| `space.6` | 24 | between groups, section spacing |
| `space.8` | 32 | major section breaks |
| `space.10` | 40 | hero spacing |
| `space.12` | 48 | top/bottom screen breathing room |

Rules:
- Screen horizontal margin: **16dp** (Android), **16–20pt** (iOS). Be consistent within a screen.
- Related elements: 8–12. Unrelated groups: 24–32.
- Never eyeball; if a value isn't on the grid, it's probably wrong.

---

## 2. Type scale

Use 2–3 sizes per screen. Differentiate hierarchy with **weight and color**, not by inventing sizes.

### Material 3 type roles (Android / Flutter Material)
| Role | Size / Line / Weight | Use |
| --- | --- | --- |
| Display L/M/S | 57/45/36 | marketing, hero numbers |
| Headline L/M/S | 32/28/24 | screen titles (large), key headings |
| Title L/M/S | 22/16/14, medium | app bar title, card titles, dialog titles |
| Body L/M/S | 16/14/12 | paragraphs, list content |
| Label L/M/S | 14/12/11, medium | buttons, chips, captions |

### iOS text styles (SwiftUI / HIG) — these scale with Dynamic Type
| Style | ~pt (default) | Use |
| --- | --- | --- |
| Large Title | 34 | top of scrollable root screens |
| Title1/2/3 | 28/22/20 | section headers |
| Headline | 17 semibold | emphasized row text |
| Body | 17 | primary content |
| Callout | 16 | secondary content |
| Subheadline | 15 | supporting |
| Footnote | 13 | metadata |
| Caption1/2 | 12/11 | labels, timestamps |

**Always use the named style/role** (`.font(.body)`, `MaterialTheme.typography.titleMedium`, `Theme.of(context).textTheme.titleMedium`). On iOS prefer Dynamic Type styles so text respects accessibility sizing.

---

## 3. Color — semantic roles, not raw hex

Pick **one** accent/brand color. Everything else derives from roles. Define the role set once in the theme; reference roles everywhere.

### Material 3 color roles (key ones)
- `primary` / `onPrimary` — main brand actions (FAB, primary button).
- `primaryContainer` / `onPrimaryContainer` — lower-emphasis primary surfaces.
- `secondary` / `tertiary` (+ containers) — accents, supporting chips.
- `surface`, `surfaceContainerLowest…Highest`, `onSurface`, `onSurfaceVariant` — backgrounds and content. M3 uses surface-container tones for elevation instead of shadows.
- `surfaceVariant`, `outline`, `outlineVariant` — dividers, borders.
- `error` / `onError` / `errorContainer`.
- `inverseSurface`, `scrim`.

Generate a full tonal scheme from one seed color (`dynamicColorScheme` / `ColorScheme.fromSeed(seedColor:)`). Provide light **and** dark schemes.

### Apple HIG semantic colors (SwiftUI)
Use system semantic colors so light/dark + contrast come free:
- Content: `.primary`, `.secondary`, `Color(.label)`, `.secondaryLabel`, `.tertiaryLabel`.
- Backgrounds: `Color(.systemBackground)`, `.secondarySystemBackground`, `.tertiarySystemBackground`; grouped: `.systemGroupedBackground`, `.secondarySystemGroupedBackground`.
- Fills: `.systemFill` … `.quaternarySystemFill` (control backgrounds).
- Separators: `Color(.separator)`, `.opaqueSeparator`.
- Accent: `.tint(.accentColor)` (one app accent), plus system semantic `.red/.green/.orange` for status.

### Cross-platform (RN/Expo)
Define a token object with both light and dark maps; switch on `useColorScheme()`. Mirror M3 role names so the mental model stays consistent. Never inline hex in components — reference the token.

**Contrast:** body text ≥ 4.5:1, large text/icons ≥ 3:1. On colored surfaces always use the matching on-color.

---

## 4. Shape / corner radius

| Token | Radius | Use |
| --- | --- | --- |
| `shape.xs` | 4 | chips, small tags |
| `shape.sm` | 8 | buttons, text fields, small cards |
| `shape.md` | 12 | cards, list containers |
| `shape.lg` | 16–20 | sheets, large cards, modals |
| `shape.xl` | 28 | bottom sheet top corners (M3), prominent containers |
| `shape.full` | 999 | pills, FABs, avatars, segmented controls |

Be consistent: a screen should use 1–2 radii, not five. iOS tends to larger continuous corners (12–16) on cards and 10 on rows; M3 uses 12 on cards, 28 on sheets.

---

## 5. Elevation & depth

- **Material 3:** prefer **tonal elevation** (surface-container color tiers) over heavy shadows. Reserve real shadows for FAB, menus, and elements that float. Levels 0–5 map to surface tones + small shadow.
- **iOS:** depth comes from **layering and background tiers** (grouped backgrounds, materials/blur `.ultraThinMaterial`), thin separators, and subtle shadows — not big drop shadows. Cards sit on a slightly different background tier.
- **General:** a soft shadow is `y: 2–8, blur: 8–24, low opacity (0.06–0.16), neutral color`. Never a hard black 0.5 shadow.

---

## 6. Motion

Default to subtle, fast, purposeful.

| Intent | Duration | Easing |
| --- | --- | --- |
| Press/state toggle | 100–150ms | ease-out / standard |
| Enter (appear) | 200–300ms | decelerate (ease-out) |
| Exit (dismiss) | 150–200ms | accelerate (ease-in) |
| Shared element / expand | 300–400ms | standard / spring |
| Sheet present | 300–350ms | spring or standard-decelerate |

- **M3 easing:** Emphasized (`emphasized`, `emphasizedDecelerate/Accelerate`) for hero transitions; Standard for most. Compose: `MotionScheme` / `tween` + `FastOutSlowInEasing`, or `spring()`.
- **iOS:** `.spring(response:dampingFraction:)` is the default feel; `.easeInOut` for simple. Respect `Reduce Motion`.
- **Flutter:** `Curves.easeOutCubic` / `Curves.fastOutSlowIn`; implicit `Animated*` widgets first, explicit controllers when needed.
- **RN:** Reanimated springs (`withSpring`) and `withTiming(…, { easing: Easing.out(Easing.cubic) })`; `LayoutAnimation` for list changes.

Animate: button press, selection, list insert/remove, navigation, expand/collapse. Don't animate: idle decoration, anything that fights the user.

---

## 7. Iconography & imagery

- One icon set per platform: **Material Symbols** (Android/Flutter), **SF Symbols** (iOS). RN: a single consistent set (e.g. SF-style on iOS, Material on Android) — don't mix.
- Icon sizes: 20/24 in rows and bars, 28–32 for emphasis. Optical-align with text baselines.
- Images: enforce aspect ratios, round with `shape.md`+, always provide a placeholder/skeleton and a failure state.

---

## 8. Token checklist when starting a screen

1. Seed/accent color chosen → generate light + dark scheme.
2. Type roles mapped (title / body / label you'll actually use).
3. Spacing scale available (screen margin = 16).
4. Shape radii picked (1–2).
5. Elevation strategy (tonal vs shadow) decided per platform.
6. Motion defaults set.

Then build components from `components.md` and lay them out per `screen-patterns.md`.
