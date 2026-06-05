# Color Systems — premium color isn't a hex picker

AI's color tell: grab a brand hue, generate three tints (`#2563EB / #3B82F6 / #60A5FA`), slap them on. Premium apps don't think in shades — they think in **semantic layers**: the same accent appears as a filled button, a subtle background, a faint border, a tinted text — each a *role*, not a lightness. This file builds a real color system so color reads intentional, not auto-generated.

> Cross-ref: `design-foundations.md` (M3/HIG color roles), `premium-polish.md` (dark mode, tinted neutrals).

---

## 1. The semantic accent ladder

One accent hue expands into a ladder of roles. Components reference the **role**, never a raw shade:

| Role | What it is | Use |
| --- | --- | --- |
| `accent` | the full-strength brand color | primary button fill, FAB, active states |
| `accent-text` | accent tuned for legibility *on* neutral bg | links, selected labels, accent-colored text |
| `accent-subtle` | low-opacity accent (8–16%) | hover/pressed wash, selected-row background |
| `accent-surface` | a soft accent-tinted surface (a "container") | highlighted cards, badges, chips, banners |
| `accent-border` | accent at ~24–40% | borders of selected/active elements, focus rings |
| `on-accent` | the color *on top of* a filled accent | button label/icon on the accent fill |

This is exactly how M3 does primary/onPrimary/primaryContainer/onPrimaryContainer and how iOS does tint + tinted backgrounds. Mirror it whatever the stack. Result: the accent feels woven through the UI as a system, not dropped as 3 random blues.

---

## 2. Build the palette (seed → scale → roles)

1. **Pick one brand hue** (the seed). Resist a second brand color until you truly need it.
2. **Generate a tonal scale** — ~10 steps from near-white to near-black of that hue (50→950, like a tonal palette). Use a perceptual model: M3 `ColorScheme.fromSeed` / Material color utilities, or HCT/OKLCH so steps look evenly spaced to the eye (not naive HSL).
3. **Map roles to steps**, separately for light and dark (the *step* differs by mode):
   - light: `accent` ≈ 500–600, `accent-text` ≈ 600–700, `accent-surface` ≈ 50–100, `accent-border` ≈ 200, `on-accent` = near-white.
   - dark: `accent` ≈ 400–500 (slightly desaturated), `accent-surface` ≈ a dark tinted 800–900, `accent-text` ≈ 300–400, `on-accent` = near-black or white depending on contrast.
4. **Verify contrast** at every pairing (text 4.5:1, large/icon 3:1).

Never hand-pick disconnected hexes per element — derive everything from the scale.

---

## 3. Neutrals — tint them, don't use dead gray

Pure `#808080` grays look cheap and disconnected. Premium UIs use **tinted neutrals**: nudge the gray slightly toward the brand hue (or a cool/warm bias) so surfaces feel cohesive.
- Build a neutral tonal scale with a tiny chroma of the brand hue (M3 does this automatically; otherwise add ~2–6% saturation toward the seed).
- Surfaces, text ramp, borders all come from this tinted-neutral scale.
- Backgrounds: **off-white** (`#FAFAFA`-ish) / **off-black** (`#0B0B0F`–`#121212`), never `#FFF` / `#000` (slop tell).

---

## 4. Functional colors — status only

`success / warning / error / info` are **reserved for meaning**, never decoration:
- Each gets the same ladder treatment (color + container + on-color) at a smaller scale.
- Green/red in finance = gain/loss **only**; never as accents.
- Don't let functional colors compete with the brand accent for attention.

---

## 5. Dark mode is a re-mapping, not an inversion

Inverting light mode produces muddy, harsh dark themes. Tune it:
- **Surfaces:** raise lightness with elevation (don't sit on pure black). Higher elevation = lighter tinted surface (M3 surface-container tiers). Off-black base, not `#000`.
- **Accent:** slightly **desaturate / lighten** (a vivid light-mode accent vibrates on dark). Use the 400–500 step.
- **Text ramp:** ~87% / 60% / 38% white (primary/secondary/tertiary), not pure white.
- **Shadows:** barely visible on dark — lean on tinted surfaces + hairline borders for separation instead.
- **Containers:** `accent-surface` becomes a dark tinted tone, not a pale one.
Always design both modes from the start; verify contrast in each.

---

## 6. Gradients — accent, not default

Gradients are a slop tell when on every card/button. Tasteful use:
- Reserve for **hero/brand moments** (a balance card, a paywall header), not the whole UI.
- Subtle: two close steps of the accent scale, or accent→darker-accent — low contrast, smooth.
- Avoid rainbow / unrelated-hue gradients. Mesh/radial tints can add life behind a hero if *barely perceptible*.
- A faint background tint or low-opacity noise/grain often reads richer than a literal gradient.

---

## 7. Per-framework wiring

- **Compose:** `ColorScheme.fromSeed(seed)` (light+dark) or a custom `lightColorScheme/darkColorScheme`; expose extra roles via a custom `Colors` data class / `CompositionLocal` if you need the full ladder beyond M3 roles. Reference `colorScheme.*` only.
- **SwiftUI:** asset catalog color set with light/dark variants + an `Accent` color; semantic system colors for neutrals; a small `Color` extension for ladder roles. `.tint(.accent)`.
- **Flutter:** `ColorScheme.fromSeed(seedColor:, brightness:)` for light+dark; a `ThemeExtension` for the extra accent-ladder roles. Read via `Theme.of(context).colorScheme.*` / extension.
- **RN/Expo:** a token object with `light`/`dark` maps mirroring the ladder; switch on `useColorScheme()`. With NativeWind, define as Tailwind `colors` + CSS vars and use `dark:` variants. Never inline hex in components.

---

## Color checklist

- [ ] One accent hue; expanded into the semantic ladder (accent / text / subtle / surface / border / on-accent), referenced by role.
- [ ] Palette derived from a perceptual tonal scale (fromSeed / OKLCH), not hand-picked hexes.
- [ ] Neutrals are tinted (toward brand), not dead gray; off-white/off-black backgrounds.
- [ ] Functional colors used only for status; finance red/green = gain/loss only.
- [ ] Dark mode re-mapped (elevated tinted surfaces, desaturated accent, text ramp), not inverted.
- [ ] Gradients only on hero/brand moments, subtle, same-hue.
- [ ] All pairings pass contrast; components reference roles, never raw hex.
