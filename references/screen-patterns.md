# Screen Patterns — Mobbin-style layout anatomy

Real apps reuse a small set of screen skeletons. When the user asks for a screen ("a chat like Telegram", "an onboarding flow", "a settings page"), match it to a pattern below, copy the anatomy, then fill it with framework components from `components.md`. Anatomy is described top→bottom as you'd lay it out.

General rules for every screen:
- Respect **safe areas** (top notch/status bar, bottom home indicator/nav bar).
- One screen = one job. One primary action, clearly dominant.
- Scroll content lives between a fixed top bar and (optionally) a fixed bottom bar/CTA.
- Primary CTA sits at the **bottom**, full-width, thumb-reachable, often pinned above the safe area.
- Include empty / loading / error variants for any data-driven screen.

---

## 1. App scaffold (tabbed root)

The container most apps live in.

```
┌─────────────────────────┐
│ status bar (safe)       │
│ ┌─────────────────────┐ │
│ │  Top app bar / large │ │  ← per-tab top bar
│ │  title              │ │
│ └─────────────────────┘ │
│                         │
│     active tab content  │  ← swap by tab
│                         │
│ ┌─────────────────────┐ │
│ │ ◉  ○   ○   ○         │ │  ← bottom nav (3–5 items)
│ └─────────────────────┘ │
│ home indicator (safe)   │
└─────────────────────────┘
```
- Android: `Scaffold` + `NavigationBar` (M3), top `TopAppBar`, optional `FloatingActionButton`.
- iOS: `TabView` + `NavigationStack` per tab, large titles.
- Flutter: `Scaffold` + `NavigationBar` + `IndexedStack`.
- RN/Expo: Expo Router tabs (`(tabs)/_layout`) or React Navigation bottom tabs.
- 3–5 destinations max. Selected item: filled icon + label + accent. Don't hide labels unless space-critical.

---

## 2. Onboarding / carousel

Sell the value, then a single CTA.

```
status bar
[ skip ]                        ← top-right, low emphasis
        ┌──────────┐
        │  illo /   │           ← hero illustration or screenshot
        │  image    │
        └──────────┘
   Big headline (Display/Large Title)
   One supporting sentence, secondary color
        ● ○ ○                   ← page dots
[      Continue / Get started  ] ← primary, full-width, pinned bottom
   secondary text link (Log in)
```
- 3–4 pages, swipeable + dots. Headline ≤ 5 words. Body ≤ 2 lines.
- Last page CTA changes to the real action. Keep "Skip" available.

---

## 3. Auth (sign in / sign up)

```
[ ← back ]
  Title: "Welcome back"          ← Headline
  subtitle, secondary
  ┌─────────────────────────┐
  │ Email                    │   ← outlined/filled text field
  └─────────────────────────┘
  ┌─────────────────────────┐
  │ Password           👁     │
  └─────────────────────────┘
                Forgot password? ← right-aligned link
[        Sign in            ]    ← primary, full-width
  ──────  or  ──────              ← divider with label
[  Continue with Apple     ]     ← provider buttons (outlined)
[  Continue with Google    ]
  No account?  Sign up            ← bottom, centered
```
- Inline validation under fields; never block typing.
- Disable primary until valid; show loading spinner in the button on submit.
- iOS: use Sign in with Apple styling; respect provider brand guidelines.

---

## 4. Feed / list (home)

```
┌ Top bar: Title (large) ······· 🔍 🔔 ┐
│ [ Chips: All  Following  News ]      │ ← optional filter row, horizontal scroll
│ ┌──────────────────────────────────┐ │
│ │ ◯  Title line              ·  •   │ │ ← list/card row
│ │     secondary · metadata          │ │
│ └──────────────────────────────────┘ │
│ ┌── card ──────────────────────────┐ │
│ │ [ image 16:9 ]                    │ │
│ │ Title                             │ │
│ │ body preview, 2 lines max         │ │
│ │ ◯ author · time      ♡  💬  ↗      │ │
│ └──────────────────────────────────┘ │
└ (FAB ⊕ bottom-right on Android)      ┘
```
- Mix row density: don't make every item a giant card. Group with section headers or sticky dates.
- Pull-to-refresh; infinite scroll with a footer spinner.
- **Empty state**: centered icon + one line + a button. **Skeletons** while loading (shimmer rows), not a blank screen.

---

## 5. Detail screen

```
[ ← ]                      [ ♡  ⋯ ]   ← transparent over hero, then solid on scroll
┌──────────────────────────────────┐
│        hero image / cover         │  ← collapses on scroll (large→small title)
└──────────────────────────────────┘
  Title (Headline)
  ◯ subtitle · meta · rating
  ── divider ──
  Body sections with clear headings
  [ chips / tags ]
  ...
[   Primary action (Buy / Book)    ]  ← pinned bottom bar, price on left + CTA right
```
- Collapsing top bar (hero shrinks, title fades into the bar).
- Pinned bottom action bar for the money/primary action; secondary actions inline.

---

## 6. Chat / messaging (Telegram/WhatsApp-style)

```
[ ← ] ◯ Name                  📞 ⋮     ← top bar: avatar + name + presence
        last seen / typing…
┌──────────────────────────────────┐
│                  ┌──────────────┐ │
│                  │ my message    │ │ ← outgoing: accent bubble, right
│                  └──────────────┘ │   tail, time + ✓✓ read ticks
│ ┌──────────────┐                  │
│ │ their message │                 │ ← incoming: neutral surface, left
│ └──────────────┘                  │
│        — Today —                  │ ← date separator, centered pill
└──────────────────────────────────┘
[ ＋ ] [ Message…            ] [ 🎤/➤ ] ← input bar pinned bottom, grows w/ text
```
- Outgoing bubbles: `primary`/accent container, on-primary text, aligned right. Incoming: `surfaceContainer`, aligned left. Max bubble width ~75–80%.
- Bubble radius ~18, with a smaller corner on the tail side. Group consecutive messages (tighten spacing, show avatar/time once).
- Input bar: multiline-growing field, attach button, send button that swaps to mic when empty. Keyboard-avoiding; list scrolls to bottom on new message and auto-pins when already at bottom.
- Timestamps subtle; status ticks for outgoing. Date separators as centered pills.

---

## 7. Profile

```
[ ← ]                         [ ⚙ ]
        ◯ large avatar
        Display name (Title)
        @handle · subtitle
   [ Edit profile ]  [ Share ]     ← paired secondary buttons
   ┌──────┬──────┬──────┐
   │ 128  │  2.4k │  310 │          ← stat row (count over label)
   │ Posts│ Foll. │ Foll.│
   └──────┴──────┴──────┘
   [ Posts | Media | Likes ]        ← segmented / tab strip
   …grid or list of content…
```
- Center the identity block. Stats in equal columns with count (bold) over label (secondary).
- Tabs switch content below; keep header visible or collapse on scroll.

---

## 8. Settings (grouped list)

```
[ ← ] Settings
  ◯  Name                        >   ← account row (avatar + chevron)
       email
  ── ACCOUNT ──                       ← section header, uppercase secondary
  🔔 Notifications                >
  🔒 Privacy                      >
  ── PREFERENCES ──
  🌙 Dark mode              [ ⬤]      ← inline toggle
  🌐 Language            English >
  ── ABOUT ──
  ⓘ  Version                  1.2.0
  [  Log out  ]                       ← destructive, red, centered
```
- Group rows into labeled sections (iOS inset-grouped style works on both platforms visually).
- Row anatomy: leading icon (tinted) · label · trailing control (chevron / value / switch). 48–56dp tall.
- Destructive actions use `error`/red and sit alone at the bottom.

---

## 9. Search

```
[ 🔍 Search…                    ✕ ]  ← focused search field, top
  Recent                    Clear
  ⏱ query one
  ⏱ query two
  ── (on typing) ──
  🔍 result row · category
  ── (results) ──
  [ All | People | Tags ]  filter chips
  result list / grid
```
- Field auto-focuses; show recents, then suggestions as you type, then results. Debounce.
- Empty results: clear "No results for '…'" + a suggestion. Cancel/clear always reachable.

---

## 10. Checkout / paywall

```
[ ✕ ]
  Headline: "Go Pro"
  ✓ benefit one
  ✓ benefit two
  ✓ benefit three
  ┌─────────┐ ┌─────────┐
  │ Monthly │ │ Yearly  │  ← selectable plan cards, one pre-selected (best value badge)
  │ $9.99   │ │ $59 -40%│
  └─────────┘ └─────────┘
[      Start free trial        ]  ← primary, full-width, pinned
  Restore · Terms · Privacy        ← tiny legal row
```
- One plan pre-selected with a "Best value" badge. Selected card: accent border + check.
- Benefits as a short check-list. Trust row (restore/terms) small at the very bottom.

---

## Choosing & combining

Most apps compose these: a tab scaffold (#1) holding a feed (#4) → detail (#5), a chat list → chat (#6), a profile (#7) → settings (#8). Build the scaffold first, then each screen, then wire navigation per the framework reference. Always include the non-happy states.
