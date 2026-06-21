# Internationalization — multilingual the right way

AI ships English-only apps with a thousand hardcoded strings, then "translates" by gluing fragments together (`"You have " + n + " messages"`), invents one plural form, and pins labels to fixed widths that explode the moment Russian or Uzbek text — both ~30–40% longer than English — lands in them. This file is the **engineering layer**: how to wire up a real i18n stack, key and interpolate strings, pluralize correctly (Russian needs `one/few/many`), format numbers/dates/relative-time per locale, and keep layouts and screen readers intact under translation. Formatting *conventions* for the UZ market (so'm, +998, `DD.MM.YYYY`) live in `locale-uz.md` — this file is the plumbing that delivers them.

> The non-negotiable: **every user-visible string is a key in a resource file, resolved at render with the active locale.** No string literal in JSX. No concatenation. If you can't find the key, it isn't translated.

---

## Setup — expo-localization + i18next (RN-first)

`expo-localization` reads the device locale/region; `i18next` + `react-i18next` resolve keys with interpolation, plurals, and fallback. Use `intl-pluralrules` to polyfill `Intl.PluralRules` on Hermes/older RN.

```bash
npx expo install expo-localization
npm i i18next react-i18next intl-pluralrules
```

```tsx
// i18n/index.ts
import 'intl-pluralrules';
import i18n from 'i18next';
import { initReactI18next } from 'react-i18next';
import { getLocales } from 'expo-localization';
import uz from './locales/uz.json';
import ru from './locales/ru.json';
import en from './locales/en.json';

// device language -> our supported set, else fallback
const deviceLng = getLocales()[0]?.languageCode ?? 'en';
const supported = ['uz', 'ru', 'en'] as const;
const initial = (supported as readonly string[]).includes(deviceLng) ? deviceLng : 'ru';

i18n.use(initReactI18next).init({
  resources: { uz: { translation: uz }, ru: { translation: ru }, en: { translation: en } },
  lng: initial,
  fallbackLng: 'ru',        // partial translations fall back, never show a raw key
  interpolation: { escapeValue: false }, // RN already escapes; don't double-encode
  returnNull: false,
});
export default i18n;
```

`getLocales()[0]` also gives `regionCode` (`UZ`), `currencyCode` (`UZS`), and `textDirection` — use `regionCode` to pick currency/format defaults, not the language. Initialize i18n **before** the first render (top of `app/_layout.tsx`), or the first paint flashes fallback strings.

## Persisted user override

Many UZ users switch uz ↔ ru freely (`locale-uz.md`), so an in-app language switch is mandatory — never rely on OS locale alone. This is **cross-cutting config**: keep `language` in a small persisted store (Zustand + AsyncStorage/MMKV) and expose it via Context, per `state-and-architecture.md`. Device locale is only the *default*; the user's choice wins and survives restart.

```tsx
// store/locale.ts
import { create } from 'zustand';
import { persist, createJSONStorage } from 'zustand/middleware';
import AsyncStorage from '@react-native-async-storage/async-storage';
import i18n from '@/i18n';

type S = { lng: string | null; setLng: (l: string) => void };
export const useLocale = create<S>()(persist(
  (set) => ({
    lng: null, // null = follow device until user picks
    setLng: (l) => { i18n.changeLanguage(l); set({ lng: l }); },
  }),
  { name: 'locale', storage: createJSONStorage(() => AsyncStorage),
    onRehydrateStorage: () => (s) => { if (s?.lng) i18n.changeLanguage(s.lng); } },
));
```

## Resource files — one per language

Mirror the same key tree across `uz.json`, `ru.json`, `en.json`. Namespace by feature; never leave a key missing (it falls back, but missing keys are translation debt).

```jsonc
// locales/en.json
{
  "auth": {
    "login": "Log in",
    "otpSent": "We sent a code to {{phone}}"     // interpolation, NOT concatenation
  },
  "cart": {
    "itemCount_one": "{{count}} item",
    "itemCount_other": "{{count}} items"
  }
}
```

```tsx
const { t } = useTranslation();
<Text>{t('auth.otpSent', { phone: '+998 90 123 45 67' })}</Text>
```

| Rule | Why |
| --- | --- |
| Key by **meaning**, not text: `cart.empty.title` not `noItemsYet` | survives copy changes |
| Pass variables via interpolation `{{name}}` | translators reorder words; concat can't |
| **One key = one full sentence** | never split a sentence across `<Text>` or components — word order differs per language |
| Keep punctuation/units *inside* the string | `"{{n}} so'm"` placement differs by locale |

## Pluralization — Russian is the hard case

English has 2 forms (`one`/`other`); **Russian has 3+** (`one`/`few`/`many`) keyed by the last digit(s); Uzbek is effectively 1–2 forms. Never hardcode `n === 1 ? x : y` — that's wrong for ru and many languages. i18next picks the form via CLDR rules from `Intl.PluralRules`; you just supply the suffixed keys.

```jsonc
// ru.json — 1 товар / 2 товара / 5 товаров
"cart": {
  "itemCount_one":  "{{count}} товар",
  "itemCount_few":  "{{count}} товара",
  "itemCount_many": "{{count}} товаров"
}
```

```tsx
t('cart.itemCount', { count: 1 });  // ru -> "1 товар"
t('cart.itemCount', { count: 3 });  // ru -> "3 товара"
t('cart.itemCount', { count: 5 });  // ru -> "5 товаров"
```

```tsx
// Which form will CLDR pick? Verify, don't guess.
new Intl.PluralRules('ru').select(2);  // 'few'
new Intl.PluralRules('ru').select(11); // 'many'
new Intl.PluralRules('uz').select(2);  // 'other'
```

Use `Intl.PluralRules` directly only when formatting outside i18next; inside, the `_one/_few/_many/_other` suffix convention is enough.

## Locale-aware formatting — Intl, not manual

Format with `Intl`, parameterized by the **active locale**, so grouping, separators, and order follow CLDR. Don't `.replace()` your way to a format. For UZ specifics (space grouping, no decimals, `so'm` after the amount, `DD.MM.YYYY`, 24h) defer to `locale-uz.md` — this is the mechanism.

```tsx
const lng = i18n.language;

// Numbers — ru/uz use space grouping
new Intl.NumberFormat(lng).format(1499000);                       // "1 499 000"

// Currency — but UZS often shown without ISO symbol; see locale-uz.md
new Intl.NumberFormat(lng, { style: 'currency', currency: 'UZS',
  maximumFractionDigits: 0 }).format(1499000);

// Dates — DD.MM.YYYY in CIS
new Intl.DateTimeFormat(lng, { day: '2-digit', month: '2-digit',
  year: 'numeric' }).format(new Date());                          // "21.06.2026"

// Relative time — "2 soat oldin" / "2 часа назад"
const rtf = new Intl.RelativeTimeFormat(lng, { numeric: 'auto' });
rtf.format(-2, 'hour');                                           // ru: "2 часа назад"
```

Build these once (formatters are expensive) and memoize per locale. Re-create them when `i18n.language` changes.

## Layout under translation

Text *expands*: ru/uz run 30–40% longer than en, German/Finnish worse. Design every label, button, tab, and chip to grow. This intersects `real-world-constraints.md` (overflow, dynamic type) and `typography-systems.md` (the ramp must hold in every language).

- **No fixed-width labels or buttons.** Let them grow / wrap; size to content with sensible `maxWidth`, not a hardcoded `width`.
- **Truncate deliberately:** `numberOfLines={1} ellipsizeMode="tail"` for single-line lists; allow 2 lines for titles. Never clip mid-word silently.
- **Tabs/segmented controls** are the first to break — short keys, scrollable tabs, or icons + labels.
- **Test at the longest language**, not English. A screen that fits in en but overflows in ru is broken.
- Pair with **Dynamic Type / font scaling** — translation expansion *and* large text stack; verify both at once.

## RTL — support it even if you ship LTR now

Uzbek (Latin/Cyrillic) and Russian are LTR, so you don't flip today — but the *Arabic-script* Uzbek and broader Arabic/Farsi markets are RTL, and building RTL-clean costs almost nothing up front.

```tsx
import { I18nManager } from 'react-native';
// Enable when an RTL language is active; requires an app reload to take effect.
I18nManager.allowRTL(true);
if (isRTLLanguage(lng) !== I18nManager.isRTL) {
  I18nManager.forceRTL(isRTLLanguage(lng));
  // then reload the app (expo-updates reloadAsync / dev reload)
}
```

- Use **logical** props — `marginStart`/`marginEnd`, `paddingStart/End`, `textAlign: 'left'` (auto-flips) — never raw `marginLeft`/`Right`.
- **Mirror directional icons** (back arrows, chevrons, progress) under RTL; leave non-directional ones (clock, checkmark) alone: `transform: [{ scaleX: I18nManager.isRTL ? -1 : 1 }]`.
- Numbers and `+998 …` phone strings stay LTR even inside RTL text.

## Fonts — cover every script you ship

A font that renders Latin but not Cyrillic shows tofu (□) for ru/uz-Cyrillic. Before choosing a typeface, confirm it covers **Latin + Cyrillic** (and Arabic if targeting those markets), including UZ specials `ʻ ў қ ғ ҳ`. Inter, Roboto, Noto Sans, and SF/Roboto system fonts cover Latin+Cyrillic; verify the *weights* you load do too. Test the whole type ramp in every language — see `typography-systems.md`.

## Accessibility — the screen reader reads the translation

Screen readers announce whatever string you render, so localized strings are read aloud automatically *if* you never bake text into images and you set the locale right. Set the document/app language so VoiceOver/TalkBack pick the correct **TTS voice and pronunciation** — a Russian string read by an English voice is garbled. `accessibilityLabel` must also come from `t()`, never a hardcoded literal. Pair with `accessibility-deep.md`.

```tsx
<Pressable accessibilityLabel={t('a11y.closeSheet')} accessibilityRole="button" />
```

## Cross-framework — same discipline, native tooling

| Platform | Strings | Plurals | Formatting |
| --- | --- | --- | --- |
| **Compose** | `res/values-<lang>/strings.xml`, `stringResource(R.string.x)` | `<plurals>` + `pluralStringResource` (CLDR-aware) | `NumberFormat`/`DateFormat` with `Locale` |
| **SwiftUI** | String Catalog `.xcstrings`, `Text("key")` / `LocalizedStringKey` | Catalog "Vary by Plural" (CLDR) | `.formatted(...)`, `Date.FormatStyle`, `Locale` |
| **Flutter** | `intl` + `.arb` files + `flutter gen-l10n` → `AppLocalizations.of(context)` | ICU `{count, plural, ...}` in ARB | `intl` `NumberFormat`/`DateFormat(locale)` |

All three: never concatenate, key by meaning, let the framework's CLDR plural engine choose the form, and format via the locale-aware API — identical rules, different files.

## Quality checklist (i18n)

- [ ] i18n initialized before first render; `expo-localization` sets the default, fallback language configured (no raw keys ever shown).
- [ ] Every visible string is a `t()` key in `uz/ru/en` resource files; zero hardcoded literals in JSX, including `accessibilityLabel`.
- [ ] Variables passed via interpolation `{{x}}`; no string concatenation; no sentence split across components.
- [ ] Plurals use `_one/_few/_many/_other` suffixes; Russian forms verified against `Intl.PluralRules('ru')`.
- [ ] Numbers/currency/dates/relative-time formatted via `Intl` parameterized by active locale; formatters memoized; UZ conventions per `locale-uz.md`.
- [ ] Layout tested at the **longest** language: no fixed-width labels, deliberate truncation, tabs/buttons grow; checked with large Dynamic Type.
- [ ] Logical start/end spacing throughout; RTL path (`I18nManager`, mirrored icons) ready even if shipping LTR.
- [ ] Loaded font weights cover Latin + Cyrillic (`ʻ ў қ ғ ҳ`) — no tofu; ramp verified in all languages.
- [ ] App/document language set so screen-reader TTS uses the correct voice and pronunciation.
