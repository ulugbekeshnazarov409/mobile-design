# Typography Systems — type *is* the brand voice

Two apps can use the same layout; the one with intentional type feels designed and the other feels like a template. AI's tell: system font, default tracking, one weight, sizes picked at random. Premium type is a **system with a voice** — Linear is tight and precise, Airbnb is warm and friendly, Revolut is financial and exact. This file gives type an identity and the craft to back it.

> Cross-ref: `design-foundations.md` (type scale/roles), `premium-polish.md` §6 (typography craft).

---

## 1. Type DNA — pick a voice

Decide the typographic personality up front (from the brand / case study), then every choice follows it:

| Voice | Feel | Choices |
| --- | --- | --- |
| **Precise / pro** (Linear, Stripe) | sharp, dense, engineered | grotesk (Inter/Geist-like), semibold headings, **tight** tracking on large, compact line-height |
| **Friendly / warm** (Airbnb, Duolingo) | approachable, soft | humanist or rounded face, medium weights, comfortable line-height, a bit more spacing |
| **Financial / exact** (Revolut, Coinbase) | confident, trustworthy | strong sans, **tabular figures everywhere**, slightly condensed numerics, crisp |
| **Calm / editorial** (Notion) | quiet, readable | clean readable face, regular weights, generous line-height, content-first |
| **Expressive / bold** (Arc) | characterful | a display face for headings + neutral body, confident sizing |

Match the voice to the product. Don't default to "Inter, regular, 16" for everything.

---

## 2. Pair intentionally (or go variable)

- **One pairing:** one expressive **display/heading** face + one highly legible **UI/body** face. Don't ship three.
- **Or one great variable font** across weights (Inter, Geist, SF, Roboto Flex) — simplest and cohesive.
- Body face must be legible at 12–14; display face can be characterful.
- Keep it to 2 families max. More = noise.

---

## 3. The weight / tracking / line-height system

Per role, set all three deliberately — this is where premium lives:

| Role | Weight | Tracking | Line-height |
| --- | --- | --- | --- |
| Display / big numerics | Semibold/Bold | **tight** (−1 to −2%) | ~1.05–1.15 |
| Headline / title | Semibold (Medium) | slightly tight (−0.5%) | ~1.2 |
| Body | Regular | default (0) | ~1.4–1.5 |
| Label / button | Medium | default→slightly loose | ~1.2 |
| Caption / overline | Medium | **loose** (+2 to +5%), often uppercase | ~1.3 |

Rules: **tighten large text, loosen small caps/labels** (optical sizing). Never leave display text at default tracking — it's the #1 type tell. Avoid Light weight on small text.

---

## 4. Numerics — tabular figures (non-negotiable for data)

Prices, balances, timers, counters, tables, stats → **tabular (monospaced) figures** so digits don't shift width and numbers don't jitter when they change.
- Compose: `TextStyle(fontFeatureSettings = "tnum")`. SwiftUI: `.monospacedDigit()`. Flutter: `fontFeatures: [FontFeature.tabularFigures()]`. RN: a font with tabular figures / `fontVariant: ['tabular-nums']`.
- Pair with the numeric-roll motion (`motion-recipes.md §7`) so changing values animate cleanly.

---

## 5. Hierarchy through type (not just size)

- 2–3 sizes per screen; differentiate primarily by **weight + color** (the P/S/T ramp from `visual-hierarchy.md`), not by adding sizes.
- A semibold 16 over a regular-muted 14 reads clearer than 20 vs 14 of the same weight/color.
- Comfortable measure (line length); predictable truncation (`numberOfLines`/ellipsis) — never let text break the layout.

---

## 6. Custom font setup per framework

- **Compose:** drop fonts in `res/font`; `val Brand = FontFamily(Font(R.font.brand_regular), Font(R.font.brand_semibold, FontWeight.SemiBold))`; build `Typography` with it; apply via `MaterialTheme(typography = …)`. Variable fonts via `Font(..., variationSettings)`.
- **SwiftUI:** add font files + `Info.plist` `UIAppFonts`; use `.font(.custom("Brand-Semibold", size: …, relativeTo: .title))` so it **scales with Dynamic Type**. Prefer a `Font` extension mapping text styles to your custom faces.
- **Flutter:** declare in `pubspec.yaml` `fonts:` with weights; set `ThemeData(textTheme: …, fontFamily: 'Brand')`; read via `Theme.of(context).textTheme.*`.
- **RN/Expo:** `expo-font` + `useFonts({ 'Brand-Regular': require(...), ... })`; gate render until loaded; reference `fontFamily` in tokens. With NativeWind, set `fontFamily` in `tailwind.config`.

Always respect OS text scaling / Dynamic Type — don't lock sizes in fixed-height boxes that clip.

---

## 7. Common type tells → fix

| Tell | Fix |
| --- | --- |
| Default system font, no identity | choose a voice + face per §1 |
| Display text at default tracking | tighten −1 to −2% |
| Five arbitrary sizes | 2–3 sizes; weight+color for hierarchy |
| Jittering numbers | tabular figures |
| Light weight on small text | Regular/Medium min |
| Three+ fonts | one pairing or one variable font |
| Fixed sizes ignoring accessibility | scale with Dynamic Type / font scale |
| Cramped or balloon line-height | ~1.4 body, ~1.1 display |

---

## Typography checklist

- [ ] A type voice chosen to match the brand/product (not defaulted).
- [ ] ≤2 families (or one variable font); body face legible small.
- [ ] Weight + tracking + line-height set per role; large tightened, small caps loosened.
- [ ] Tabular figures wherever numbers change.
- [ ] Hierarchy via weight+color, 2–3 sizes; truncation handled.
- [ ] Custom fonts wired the framework's way; scales with Dynamic Type / font scale.
