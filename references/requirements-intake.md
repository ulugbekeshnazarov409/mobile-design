# Requirements Intake — ask before you build

A vague brief ("build me a fitness app", "make a delivery app") leaves domain, audience, style, and scope unstated — and guessing wrong means rebuilding. A short, batched, multiple-choice intake costs the user ten seconds and pins down the few decisions that actually change the design. Keep it **fast**: batch all questions at once, attach a sensible default to each so the user can reply "yes, defaults," and **skip it entirely** when the brief is already specific. Intake informs the PLAN block (`SKILL.md` Step C) — it never blocks a clear request.

> Rule of thumb: ask only the questions whose answers would change what you build. If you already know the answer from the brief, don't ask it — confirm it in the plan instead.

---

## When to run intake vs skip

| Situation | Action |
| --- | --- |
| Brief names a domain but not style/audience/scope ("build a fitness app") | **Run intake** (batched MC) — domain known, fill the rest |
| Brief is broad and underspecified ("make me an app", "design something cool") | **Run intake** — almost everything is unstated |
| Style/mood/reference unstated and it changes the whole look | **Ask** (at least the style + color question) |
| UZ/CIS market plausible but unconfirmed (so'm? +998? uz/ru?) | **Ask** the locale question — it changes formatting & components |
| Brief already specifies domain + style + platform + scope | **Skip** — go straight to PLAN; one-line confirm if anything's fuzzy |
| Tiny one-component / one-screen ask ("style this button", "a Compose card") | **Skip** — build it; a one-line plan is enough |
| Existing repo with established stack/tokens/style | **Skip stack/style questions** — detect from repo (`project-awareness.md`), ask only product-level gaps |
| User is clearly iterating on something already built | **Skip** — don't re-interrogate; just build the change |

**Decision rule:** run intake when **domain, purpose, style, or audience are unstated or ambiguous** AND the ask is a screen/flow/app (not a single component). Otherwise skip or one-line confirm. Never answer a clear request with a wall of questions.

---

## How to ask

- **Batch everything into one message.** One round-trip, not a back-and-forth interrogation. 3–6 questions max — only the ones that change the design.
- **Each question = 2–4 concrete options + an "Other" escape.** Multiple choice, not open-ended; concrete labels beat abstract ones.
- **Recommend a default** per question (mark it ★) inferred from the brief, so the user can reply "go with the defaults" and you proceed immediately.
- **Prefer the host's structured multiple-choice question UI** when the environment provides one (e.g. a question/option picker) — fall back to a plain numbered list otherwise.
- **Cut questions you can already answer.** If the brief says "Revolut-style dark fintech for Uzbekistan," only scope remains open.
- **Lead with the recommendation:** "I'll assume X, Y, Z (defaults below) unless you change anything — reply to confirm or pick alternatives."

---

## The question bank

Ask only the subset that's still open. Each row: the question, concrete options, and the default heuristic.

### 1. App purpose / domain
*What kind of app is this?* (Bu qanday ilova?)

| Option | Implies |
| --- | --- |
| FinTech / Wallet (Hamyon, to'lov) | balances, cards, transfers, security signals, tabular figures |
| E-commerce / Marketplace (Savdo) | catalog, cart, checkout flow, product cards |
| Social / Community (Ijtimoiy) | feed, profiles, messaging, content creation |
| Delivery / Logistics (Yetkazib berish) | map UI, order tracking, ETA, address picker |
| Real estate (Ko'chmas mulk) | listings, map+filters, gallery detail |
| Health / Fitness (Sog'liq / Fitnes) | tracking, charts, streaks, sessions |
| Productivity / Tools (Ish unumdorligi) | lists, tasks, dense data, keyboard-first |
| Media / Content (Media) | player, browse/discover, episodes |
| Other (Boshqa) | describe in one line |

### 2. Core jobs / key features (multi-select)
*What are the 1–3 things a user must be able to do?* (Foydalanuvchi nima qila olishi kerak?) e.g. for fitness: "log a workout," "see progress," "follow a plan." This sets the screen list and the primary action per screen. Keep it to 1–3 — more is a Stage problem (see Q7).

### 3. Target platform
*Which platform(s)?* (Qaysi platforma?) — ties to **DETECT** in `SKILL.md` (in an existing repo, detect instead of asking).

| Option | Stack |
| --- | --- |
| iOS only | SwiftUI |
| Android only | Jetpack Compose (Kotlin) |
| Cross-platform — Flutter | Flutter / Dart |
| Cross-platform — React Native / Expo | RN + detected UI library |
| ★ Not sure | recommend Compose **or** SwiftUI for a single native target; Flutter/Expo if both matter |

### 4. Visual style / mood
*What should it feel like?* (Qanday taassurot qoldirsin?) — ties to `color-systems.md` + `platform-personality.md`.

| Option | Direction |
| --- | --- |
| Minimal & clean (Sodda) | airy spacing, restrained accent, neutral surfaces |
| Premium & luxurious (Premium) | deep darks, refined type, subtle gradients/materials |
| Playful & friendly (Quvnoq) | rounded shapes, warm accents, lively motion |
| Bold & expressive (Jasur) | large type, saturated color, strong contrast |
| Corporate & trustworthy (Ishonchli) | conservative palette, structured, calm |

### 5. Color direction
*Accent hue + theme?* (Asosiy rang + mavzu?) — ties to `color-systems.md` (one accent → semantic ladder; true dark mode).

| Accent | Theme |
| --- | --- |
| Blue / Teal / Green / Purple / Orange / Brand-color | Light only · Dark only · ★ Both |

Default: **Both** themes, accent inferred from domain (fintech→deep blue/green, fitness→energetic green/orange, social→violet). One accent, expanded — never raw per-element hex.

### 6. Reference apps
*"Make it like…"* (…ga o'xshatib) — name an app whose feel to borrow; pull its DNA from `case-studies.md`. Suggested anchors: **Linear** (focused tools), **Revolut** (fintech), **Telegram** (messaging), **Airbnb** (marketplace/booking), **Uber** (map/delivery). Default: pick the closest anchor for the chosen domain + mood.

### 7. Product maturity
*Which stage?* (Qaysi bosqich?) — ties to `product-maturity.md`; controls how much UI to build.

| Option | UI complexity |
| --- | --- |
| ★ MVP (Stage 0) | one core flow, minimal nav, one CTA/screen |
| Growth (Stage 1) | tab bar, search, discovery, onboarding |
| Scale (Stage 2) | power features, density, customization |

Default to the **earliest plausible stage** when unstated — cheaper to add than to remove.

### 8. Audience & region / locale
*Who and where?* (Kim uchun va qayerda?) — ties to `locale-uz.md`.

| Option | Implies |
| --- | --- |
| General / international | USD/EUR, intl phone, en |
| ★ UZ / CIS market | so'm (UZS) formatting, `+998` phone mask, Uzcard/Humo cards, viloyat→tuman pickers, uz-Latin / uz-Cyrillic / ru language |

Given this skill's primary users, **default to UZ-market** unless the brief reads international.

### 9. Scope
*Single screen or a full flow?* (Bitta ekranmi yoki to'liq oqimmi?) — ties to `flow-architecture.md`.

| Option | Build |
| --- | --- |
| Single screen | one screen + all its states |
| ★ Full flow | screen sequence + navigation + carried state + exits (if the feature implies steps: checkout, onboarding, transfer, booking) |

---

## How intake feeds the PLAN block

Map each answer straight into `SKILL.md` Step C plan fields — every answer has a home, so the plan reflects intent, not a guess.

| Intake question | PLAN field |
| --- | --- |
| 3 Platform | **Stack** (`<language> / <framework>`) |
| 4 Mood + 5 Color + 6 Reference | **Visual** (accent ladder + type voice; platform personality) |
| 7 Maturity | **Maturity** (Stage 0/1/2/3) |
| 1 Domain + 2 Core jobs | **Screen** (pattern from `screen-patterns.md`) |
| 2 Core jobs + 9 Scope | **States** (which empty/loading/error/offline to include) and flow vs single screen |
| 8 Locale | **locale** line (so'm / +998 / uz·ru, or international) |

If a question was skipped, fill its plan field with your stated assumption (e.g. "Maturity: assumed Stage 0 MVP") so the user can still correct it in one reply.

---

## Quality checklist (intake)

- [ ] Decision rule applied: ran intake only when domain/purpose/style/audience were unstated or ambiguous; skipped for specific briefs and one-component asks.
- [ ] Questions batched into **one** message (3–6 max), each with concrete options + an "Other" escape and a ★ recommended default.
- [ ] Used the host's structured multiple-choice UI when available; numbered list otherwise.
- [ ] Asked only questions whose answers change the design; questions answerable from the brief were confirmed, not asked.
- [ ] Never blocked a clear request with questions; a clear brief went straight to PLAN.
- [ ] Every answer (or stated assumption) mapped into a PLAN field: Stack, Visual, Maturity, Screen, States, locale.
- [ ] Locale resolved (UZ-market default vs international) before money/phone/card UI is built.
- [ ] Scope resolved (single screen vs full flow) before laying out.
