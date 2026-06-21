# Locale Conventions — Uzbekistan & CIS market UI

Apps built for the UZ/CIS market have concrete formatting and pattern expectations that generic AI UI gets wrong: it dumps `$` signs, US phone masks, and English-only labels. This file captures the local conventions so screens read native to the region. The principles generalize to neighboring CIS markets (KZ, KG, TJ) — swap the currency/codes.

> These are **formatting + pattern** rules, not a translation table. Keep numbers tabular (`typography-systems.md`), money right-aligned, and every locale string externalized (no hardcoded text).

---

## Currency — Uzbek so'm (UZS)

- **Grouping with spaces, not commas:** `1 000 000` — thin/regular space as the thousands separator (CIS standard). Comma grouping (`1,000,000`) reads foreign.
- **Decimals:** so'm is effectively used as a whole number in most apps — **no decimals** for prices/balances. Show `25 000 so'm`, not `25,000.00`.
- **Label:** `so'm` (Latin) or `сўм` (Cyrillic) or `UZS` — match the app's language. Symbol goes **after** the amount with a space: `499 000 so'm`. For tight UI, `UZS 499 000` is acceptable.
- **Tabular figures** for any column of prices/balances so digits align; right-align money in lists and receipts.

```ts
// UZS formatter — space grouping, no decimals
export const fmtUzs = (n: number) =>
  new Intl.NumberFormat('ru-RU', { maximumFractionDigits: 0 }).format(n).replace(/ /g, ' ') + ' so\'m';
// fmtUzs(1499000) -> "1 499 000 so'm"
```
```dart
// Flutter — intl package
import 'package:intl/intl.dart';
final fmt = NumberFormat.decimalPattern('ru'); // space grouping
String uzs(num v) => "${fmt.format(v)} so'm"; // "1 499 000 so'm"
```
```kotlin
// Compose / Kotlin
val symbols = java.text.DecimalFormatSymbols(java.util.Locale("ru")).apply { groupingSeparator = ' ' }
val df = java.text.DecimalFormat("#,##0", symbols)
fun uzs(v: Long) = "${df.format(v)} so'm"   // "1 499 000 so'm"
```
```swift
// SwiftUI
var nf: NumberFormatter { let f = NumberFormatter(); f.numberStyle = .decimal; f.groupingSeparator = " "; f.maximumFractionDigits = 0; return f }
func uzs(_ v: Int) -> String { (nf.string(from: v as NSNumber) ?? "\(v)") + " so'm" }
```

---

## Phone numbers — +998

- Format: **`+998 90 123 45 67`** — country `+998`, 2-digit operator code, then `123 45 67` grouped 3-2-2.
- Operator codes: `90 91` (Beeline/old), `93 94` (Ucell), `99 95` (UMS/Mobiuz), `88` (Humans), `33`, `97`, `98` … Treat input as 9 digits after `+998`.
- Input UX: pre-fill the `+998` prefix as a fixed affix; mask the rest `00 000 00 00`; numeric keypad; validate 9 digits.
- Used heavily for **OTP login** (SMS is the dominant auth in the region) — design the OTP flow (`interaction-patterns.md`): send → 5–6 digit code → resend timer → auto-read where possible.

```
+998 [90] 123 45 67
      └ operator code (2)   └ 7 digits grouped 3-2-2
```

---

## Bank cards — Uzcard, Humo, Visa, Mastercard

Local payment runs on **Uzcard** (`8600…`) and **Humo** (`9860…`); international on Visa/Mastercard. A card UI must detect and brand by prefix.

| Network | BIN prefix | Length | Visual cue |
| --- | --- | --- | --- |
| Uzcard | `8600`, `5614` | 16 | Uzcard logo, blue/teal brand |
| Humo | `9860` | 16 | Humo logo, green/orange brand |
| Visa | `4` | 16 | Visa logo |
| Mastercard | `51`–`55`, `2221`–`2720` | 16 | Mastercard logo |

- **Grouping:** `0000 0000 0000 0000` (4-4-4-4). Expiry `MM/YY`. Show last-4 in lists: `•••• 4242`.
- **Card visual** (saved-cards / wallet): rounded 16:10-ish card, network logo top-right, masked PAN, holder name, expiry; brand-tinted gradient or solid; selected state = ring/elevation. (See `components.md` card + `glassmorphism-and-materials.md` for a glass card face.)
- Detect network from the BIN **as the user types** to swap the logo and validate length. Use a Luhn check + length, but **Uzcard/Humo are domestic schemes** — validate by prefix + length, don't reject them as "unknown".
- Never store/log full PAN in the UI layer; mask on blur.

```ts
function cardBrand(num: string): 'uzcard'|'humo'|'visa'|'mastercard'|'unknown' {
  const n = num.replace(/\s/g, '');
  if (/^(8600|5614)/.test(n)) return 'uzcard';
  if (/^9860/.test(n)) return 'humo';
  if (/^4/.test(n)) return 'visa';
  if (/^(5[1-5]|222[1-9]|2[3-6]\d\d|27[01]\d|2720)/.test(n)) return 'mastercard';
  return 'unknown';
}
const groupPan = (n: string) => n.replace(/\D/g, '').replace(/(.{4})/g, '$1 ').trim(); // 0000 0000 0000 0000
```

---

## Region selectors — viloyat → tuman (cascading)

Addresses are picked as **viloyat (region) → tuman/shahar (district/city)** — a dependent two-level selector. There are 14 regions (12 viloyat + Karakalpakstan + Tashkent city), each with its own districts.

- **Cascade:** selecting a viloyat filters the tuman list; reset tuman when viloyat changes; disable tuman until a viloyat is chosen.
- Mobile pattern: two stacked rows opening a searchable bottom-sheet picker each (not tiny native dropdowns) — districts can be long lists, so make them searchable.
- Tashkent city is both a region and effectively district-level — handle the special case.
- Store stable IDs, not display strings (labels change with language).

```
[ Viloyat            ▾ ]   → opens searchable sheet (14 regions)
[ Tuman / shahar     ▾ ]   → disabled until region; filtered to that region
```

---

## Language & text

- App languages: **uz (Latin)**, **uz (Cyrillic)**, **ru**, often **en** — many users switch uz↔ru freely. Provide an in-app language switch; don't rely solely on OS locale.
- **No RTL** needed (Latin/Cyrillic), but **text expands** — Russian and Uzbek strings run longer than English; design for overflow/wrapping (`real-world-constraints.md`), don't truncate labels.
- Cyrillic and Latin Uzbek differ in width and some glyphs — verify the chosen font covers both (`ʻ`/`ў`/`қ`/`ғ`/`ҳ`); test the type ramp in all languages.
- Dates: `DD.MM.YYYY` (CIS standard), 24-hour time.
- Numbers: space grouping everywhere (not just money), decimal comma in ru contexts.

---

## Quality checklist (UZ/CIS locale)

- [ ] Money: space grouping, no decimals, `so'm`/`UZS` after the amount, tabular + right-aligned.
- [ ] Phone: `+998` fixed prefix, `00 000 00 00` mask, numeric keypad, 9-digit validation; OTP flow built.
- [ ] Cards: brand detected by BIN (Uzcard/Humo included, not rejected), 4-4-4-4 grouping, `MM/YY`, masked last-4, branded card visual.
- [ ] Region: viloyat→tuman cascade with searchable sheets, dependent reset, IDs stored not labels.
- [ ] Languages uz-Latin/uz-Cyrillic/ru/(en) supported with in-app switch; strings externalized; font covers Cyrillic+Latin.
- [ ] Text-expansion safe (ru/uz longer than en); dates `DD.MM.YYYY`, 24h.
