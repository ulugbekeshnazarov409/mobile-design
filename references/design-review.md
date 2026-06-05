# Design Review / Lint mode

A mode where the skill **audits existing UI code** for design-quality problems and (optionally) refactors it — like a mobile equivalent of the Vercel Web Interface Guidelines review. Trigger when the user says "review this screen", "lint my UI", "is this design correct", "make this match guidelines", "fix the design issues", or asks for a refactor of existing UI code.

## How to run a review

1. **Identify stack & system** via `project-awareness.md` (framework, styling system, tokens). Rules are checked against the project's own tokens where they exist.
2. **Read the target file(s).** Map each UI element to the checklist below.
3. **Report findings** in this exact format, one per line, most severe first:

   ```
   path:line — <severity> — <problem>. <fix>.
   ```
   Severity: 🔴 critical (broken/inaccessible), 🟠 major (clear guideline violation), 🟡 minor (polish). No praise, no scope creep, skip pure formatting the linter already owns.

4. **Refactor only if asked** (`--fix`, "fix it", "refactor"). Apply the fixes, keep changes minimal and within the project's conventions, then summarize what changed.

## Audit checklist

### Touch & ergonomics
- Interactive target hit area ≥ **48dp (Android) / 44pt (iOS)**. Flag small icon buttons, tiny text links, crowded rows.
- ≥ 8dp spacing between adjacent targets.
- Primary action / primary nav in the **thumb zone** (bottom), not stranded top.

### Tokens & values
- No raw hex / `0xFF…` / magic spacing inside components — must use tokens/theme. Flag each literal.
- Spacing on the 4dp grid; flag stray values (e.g. `13`, `17`, `painfully specific 22`).
- Type uses the platform scale/role, not arbitrary `fontSize`.
- Radii consistent (1–2 per screen), from the shape scale.

### Color & contrast
- Semantic roles used, not per-element hex.
- Body text contrast ≥ 4.5:1, large text/icons ≥ 3:1. Flag low-contrast pairs.
- On colored surfaces, the matching on-color is used.

### Dark mode & theming
- Both light and dark handled. Flag hardcoded colors that won't adapt.
- Reads from theme/`colorScheme`/`useColorScheme`, not constants.

### Layout & safe areas
- Safe-area insets honored (no content under notch/home indicator/status bar).
- Long lists use the virtualized list (`Lazy*` / `FlatList` / `ListView.builder`), not a mapped scroll for big data.
- Full-width CTAs actually full-width; padding consistent (screen margin 16).
- Keyboard handling for inputs (avoiding/scroll).

### States
- Empty, loading (skeleton/spinner), and error states exist for data screens. Flag happy-path-only.
- Buttons show disabled + loading states; destructive actions visually distinct (error/red).

### Platform fit
- Right component for the OS (no Android FAB on an iOS-only screen; no Material look forced on iOS, etc.).
- Back/dismiss works per platform; navigation depth matches intent.
- One consistent icon set.

### Motion & feedback
- Tappables give press feedback (ripple/opacity/scale).
- State changes/transitions animated with platform easing, 150–300ms; Reduce Motion respected.

### Component hygiene
- No giant mega-component; reusable, variant-driven, stateless-where-possible.
- Accessibility labels/roles present; dynamic type not broken by fixed heights.

## Example output

```
HomeScreen.kt:42 — 🔴 — IconButton hit area 32dp, below 48dp minimum. Wrap with Modifier.minimumInteractiveComponentSize() or size(48.dp).
HomeScreen.kt:58 — 🟠 — Hardcoded Color(0xFF1E88E5); won't adapt to dark theme. Use MaterialTheme.colorScheme.primary.
HomeScreen.kt:71 — 🟠 — Long list rendered in Column; will jank. Use LazyColumn with keys.
HomeScreen.kt:90 — 🟡 — padding(13.dp) off the 4dp grid. Use 12.dp.
HomeScreen.kt:— — 🟠 — No empty/loading state for the list. Add skeleton + empty view.
```

After reporting (or fixing), restate the top 1–3 things that most improve the design so the user knows what mattered most.
