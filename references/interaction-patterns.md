# Interaction Patterns — design the flow, not just the screen

AI builds screens. Real products build **flows**. A login screen isn't "email + password + button" — it's a system of states: typing, validating, submitting, error, locked-out, offline, success, back, cancel. The difference between *screen-aware* and *flow-aware* is the difference between a mockup and a product. This file makes the skill think in flows and edge states.

> Rule: for every screen you build, enumerate its **states and exits** before you call it done. The happy path is maybe 30% of the work.

---

## Every screen is a state machine

Don't design "the screen" — design every state it can be in. Minimum states to consider:

| State | Question | Don't forget |
| --- | --- | --- |
| **Initial / idle** | what shows before any input? | placeholders, defaults |
| **Loading** | first load? | skeleton, not blank/spinner |
| **Loaded / content** | the happy path | the obvious one |
| **Empty** | no data? | icon + one line + action |
| **Error** | request failed? | message + **retry** |
| **Offline** | no network? | banner/state + retry/queue |
| **Partial / slow** | slow network? | progressive load, stale-while-revalidate |
| **Submitting** | action in flight? | disable + spinner in button, prevent double-submit |
| **Success** | action done? | feedback (haptic/toast/transition), then where? |
| **Disabled** | preconditions unmet? | why it's disabled, how to enable |
| **Permission denied** | needs camera/location/notif? | request + graceful fallback |

If you only built "loaded", you built ~30% of the screen.

---

## Per-flow state coverage (recipes)

### Auth (login / OTP)
Not `Login → Home`. The real flow:
```
Form (idle)
  → validating (inline, per-field, non-blocking)
  → submitting (button spinner, fields locked)
  → ERROR: wrong credentials (inline, keep input), network fail (retry), rate-limited (cooldown msg)
  → OTP screen → resend (cooldown timer), wrong code (clear + shake + haptic), expired (resend)
  → SUCCESS → Home (with transition)
Exits: back, cancel, "forgot password", switch to sign-up, biometric shortcut
```
Must-haves: never lose typed input on error; disable submit while in flight; resend cooldown; autofill/SMS-OTP autofill; show/hide password.

### Forms / create flows
```
Editing → field validation (inline) → review (optional) → submit (in flight) → success / error
Edge: unsaved-changes guard on back/cancel; partial save/draft; field-level + form-level errors;
      long content; required vs optional clear; keyboard "next/done" wiring.
```

### Checkout / payment
```
Cart → method select → review → processing (don't let them double-pay) → success / decline / error
Edge: payment failure (clear reason + retry, don't lose cart), 3DS/redirect, network drop mid-pay,
      insufficient funds, success → receipt/confirmation. SUCCESS haptic; lock UI during processing.
```

### Lists / feeds
```
Loading (skeleton) → content → (empty | error | offline)
Edge: pull-to-refresh, infinite scroll (footer loader + end-of-list state), failed page load (inline retry),
      optimistic updates (like/delete) with rollback on failure, stale-while-revalidate.
```

### Chat
```
Sending (optimistic, clock icon) → sent (✓) → delivered (✓✓) → read → FAILED (retry affordance)
Edge: offline queue, typing indicator, scroll-pin vs new-message-while-scrolled-up, media upload progress.
```

### Search
```
Idle (recents) → typing (debounced) → results | no-results | error
Edge: clear/cancel always reachable; debounce; "no results for 'x'" + suggestion; recent/clear.
```

---

## Transitions between states (make them legible)

- **Idle → loading:** skeletons appear (no layout jump). 
- **Loading → content:** content fades/staggers in (motion §4), skeleton fades out.
- **Any → error:** inline where possible (not a full-screen takeover for a recoverable error); retry obvious.
- **Submitting → success:** feedback moment (haptic + checkmark/toast), then navigate — don't just silently jump.
- **Optimistic actions:** update immediately, reconcile on response, **roll back visibly** on failure.

Never hard-jump between states with no transition — it reads as a bug.

---

## Prevent the classic flow bugs

- ❌ Double-submit (tapping pay/send twice) → disable + guard in flight.
- ❌ Losing user input on error/rotation/back → preserve state.
- ❌ Dead-end errors (no retry) → always a way forward.
- ❌ Trapping the user (no back/cancel/exit) → always an exit.
- ❌ Silent failures → every failure has visible feedback.
- ❌ No offline handling → detect + inform + queue/retry.
- ❌ Unsaved-changes lost on back → confirm guard.
- ❌ Success with no confirmation → acknowledge before moving on.

---

## Interaction checklist (per screen)

- [ ] Enumerated all states (initial/loading/loaded/empty/error/offline/submitting/success/disabled).
- [ ] Every state designed, not just the happy path.
- [ ] Every entry has an exit (back/cancel/dismiss); user never trapped.
- [ ] Errors are recoverable (retry/resend), input preserved, double-submit blocked.
- [ ] Offline + slow-network considered.
- [ ] State transitions animated/legible; optimistic updates roll back on failure.
- [ ] Success moments acknowledged (feedback) before navigating.

For multi-screen journeys, hand off to `flow-architecture.md`.
