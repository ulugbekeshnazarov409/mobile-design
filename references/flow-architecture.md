# Flow Architecture — build the whole journey, not one screen

When a user asks for "a booking screen" or "a transfer feature", a real product designer thinks about the **whole journey**: the sequence of screens, what carries between them, where it branches, and how someone backs out. Screen-aware AI builds one box; flow-aware AI builds the connected experience. This file maps the canonical flows so the skill can scaffold an entire feature, navigation included.

> When a request implies a multi-step task ("checkout", "onboarding", "sign up", "send money"), build the **flow** (screens + navigation + shared state + exits), not just the one screen named. Confirm scope, then scaffold the sequence.

---

## Canonical flows (sequence + branches + carried state)

### Airbnb-style booking
```
Search (dates, guests, location)
  → Results (list/map, filters)        ← carries: query
  → Listing detail (gallery, price)    ← carries: selected listing
  → Reserve (dates, guests confirm)    ← carries: listing + dates
  → Pay (method, price breakdown)      ← carries: booking draft
  → Confirmation (receipt, next steps)
Branches: no results → adjust search; unavailable dates → back to detail;
          payment fail → retry (keep booking); not-logged-in → auth interstitial → resume.
Exits: back at each step (preserve choices); close → discard with guard.
```

### Telegram-style messaging
```
Chat list
  → Conversation (messages, input)     ← carries: chat id
  → Media viewer / attachment          ← carries: message
  → Contact / group profile            ← carries: peer
  → Shared media, settings
Branches: new chat → contact picker → conversation; offline → queue sends;
          forward → chat picker → back. Exits: back preserves scroll position.
```

### Revolut-style money transfer
```
Accounts / home (balance hero)
  → Transfer (recipient, amount)       ← carries: source account
  → Review (fees, rate, confirm)       ← carries: transfer draft
  → Auth (biometric / PIN)             ← carries: draft
  → Processing → Success (receipt)
Branches: insufficient funds → back to amount; recipient not found → add;
          auth fail → retry; network drop in processing → safe state (don't double-send).
Exits: cancel at any pre-auth step discards; post-auth is committed.
```

### Onboarding → first run
```
Splash → Value carousel (3–4) → Auth (sign up / in)
  → Permissions (notif/location, contextual, one at a time)
  → Personalization (optional)
  → Home (with a first-use hint, not a wall of tooltips)
Branches: skip → minimal path to Home; permission denied → graceful fallback;
          returning user → straight to auth/Home.
Exits: skip always available; back between carousel pages.
```

### E-commerce checkout
```
Cart → (auth if needed) → Shipping → Payment → Review → Processing → Confirmation
Branches: edit cart mid-flow; promo code; address validation fail; payment decline (retry, keep cart);
          guest vs account. Exits: back preserves cart; abandon → cart persists for later.
```

### Content creation (post/upload)
```
Compose (text/media) → edit/preview → metadata (tags, visibility) → publishing → published / failed (retry, keep draft)
Branches: save draft; discard guard; upload progress + cancel; media too large.
```

---

## What carries between screens (shared state)

For each flow, decide **what data persists across steps** and where it lives — don't re-fetch or lose it:
- A **flow/draft object** (booking, transfer, order) built up step by step, held in flow-scoped state (a nav-graph-scoped ViewModel / route params / a context/store), cleared on completion or abandon.
- Selections (dates, recipient, method) survive back navigation — going back and forward must not reset choices.
- Match the project's state tool (ViewModel/SavedState, Riverpod/Bloc, Zustand/Redux/Context, SwiftUI `@Observable`/environment) — don't introduce a new one.

---

## Navigation shape per flow

- **Linear multi-step (checkout, transfer, onboarding):** a **stack** with a progress indicator; back goes one step; a top-level "cancel/close" with a discard guard.
- **Hub-and-spoke (settings, profile):** a root list pushing detail screens.
- **Tabbed root + per-tab stacks:** the app scaffold; each tab keeps its own back stack.
- **Modal/sheet for focused sub-tasks:** transfer amount entry, filters, quick create — present over context, dismiss returns.
- **Interstitials:** auth/permission gates that **resume** the original flow after completing (don't dump the user at Home).

Wire with the framework's router (Navigation-Compose graphs, SwiftUI `NavigationStack` paths, go_router, Expo Router / React Navigation). Keep the back stack sane: completing a flow usually **pops the whole flow** (you don't want Back to walk through checkout again) — replace to the result screen.

---

## Building a flow (process)

1. **Name the flow** and its end goal (what "done" means).
2. **List the screens** in sequence + the branches (error/empty/auth/permission detours).
3. **Define the carried state** (the draft object) and where it lives.
4. **Choose the navigation shape** (stack/modal/interstitial) + how it ends (pop-to-result).
5. **Build each screen** with full interaction states (`interaction-patterns.md`).
6. **Wire transitions** (motion §6 hero where it fits) and exits (back/cancel/guard).
7. **Verify the journey end-to-end**, including every branch, offline, and abandon.

---

## Flow checklist

- [ ] Built the whole journey the request implies, not just the named screen (scope confirmed).
- [ ] Screens sequenced; branches (error/empty/auth/permission/offline) mapped.
- [ ] Shared/draft state defined, carried across steps, survives back, cleared on done/abandon.
- [ ] Navigation shape chosen; flow pops cleanly on completion (no walking back through it).
- [ ] Interstitials resume the original flow.
- [ ] Every step has a back/cancel with a discard guard where needed.
- [ ] End-to-end verified across all branches + abandon + offline.
