# Real-World Constraints — the conditions AI forgets

AI designs for the demo: perfect data, English, a fast network, an iPhone-sized screen, default text size. Real users are offline, on Arabic with huge text on a small Android, or a tablet in landscape. Premium apps survive all of it. This file is the checklist of conditions to design for so the screen doesn't break in the wild.

> Cross-ref: `interaction-patterns.md` (offline/error/loading states), `ui-restraint-and-accessibility.md` (dynamic type, targets).

---

## 1. Network reality

- **Offline:** detect connectivity; show a clear state/banner; queue actions and retry; never silently fail or hang. (RN: `@react-native-community/netinfo`; Flutter: `connectivity_plus`; native: reachability APIs.)
- **Slow network:** skeletons + progressive load; stale-while-revalidate (show cached, refresh in background); timeouts with retry; image placeholders (blurhash).
- **Flaky mid-action:** don't double-submit; show in-flight state; safe recovery (a payment that times out must not double-charge).

## 2. Internationalization & RTL

- **RTL (Arabic, Hebrew, Farsi):** layouts mirror — leading/trailing not left/right, chevrons flip, text aligns right. Use **start/end** not **left/right** in styles. Test mirrored.
  - Compose: `LocalLayoutDirection`, `Modifier.padding(start=…)`; SwiftUI: leading/trailing + `.environment(\.layoutDirection, .rightToLeft)` to test; Flutter: `Directionality`, `EdgeInsetsDirectional`; RN: `I18nManager`, use `start/end`, `writingDirection`.
- **String length:** translations are longer (German ~+35%); never assume text fits — allow wrap/truncate, avoid fixed-width labels. Don't concatenate sentences from fragments.
- **Formats:** dates, numbers, currency, first/last name order vary by locale — use locale-aware formatters, not hardcoded formats.

## 3. Text scaling / Dynamic Type

- Users set large fonts (accessibility). Text must **scale** — don't lock sizes in fixed-height boxes that clip.
- Layouts must reflow at the largest sizes (test at max). Prefer intrinsic heights; let rows grow.
- Respect the OS setting (don't disable font scaling); icons/targets stay ≥ minimum.

## 4. Screen sizes & form factors

- **Small phones:** verify the smallest common width; content must not overflow; CTAs reachable.
- **Tablets / large screens / web:** don't stretch a phone layout edge-to-edge. Use **two-pane** (list-detail) at expanded widths, max content width, more columns. (Compose `WindowSizeClass`; SwiftUI size classes / `NavigationSplitView`; Flutter `LayoutBuilder`/`MediaQuery`; RN dimensions/breakpoints.)
- **Foldables:** handle fold/unfold, hinge, posture changes; resize gracefully.
- **Orientation:** support or lock intentionally; don't break on rotate; preserve state across config changes.
- **Safe areas / notches / cutouts / nav bars:** respect insets everywhere (also bottom gesture bar).

## 5. Content reality

- **Long content:** long names/titles/messages → truncate or wrap predictably; never overflow or push the layout.
- **Empty:** no data is a *designed* state (icon + line + action), not a blank screen.
- **Lots of data:** virtualize lists; paginate; handle "1,000+ items" without jank.
- **Missing media:** broken/absent images → placeholder + graceful fallback (initials avatar, etc.).
- **User-generated edge cases:** emoji, mixed scripts, huge numbers, zero/negative values.

## 6. Permissions & system state

- **Permission denied** (camera/location/notifications/contacts): graceful fallback + a path to re-enable in Settings; never a dead end.
- **Interruptions:** calls, low battery, backgrounding mid-flow — preserve state, resume cleanly.
- **System theme / contrast / reduce-motion / reduce-transparency:** honor all.

---

## Bake it into the workflow

For any screen, ask: *what happens when…* offline? RTL? text at 200%? a 7-year-old Android? a tablet? a 40-character name? no permission? If any answer is "it breaks," it's not done.

---

## Real-world checklist

- [ ] Offline + slow + flaky network handled (state, retry, no double-submit).
- [ ] RTL mirrors correctly (start/end, flipped icons); translated strings fit (wrap/truncate).
- [ ] Text scales to largest Dynamic Type without clipping; layout reflows.
- [ ] Works on smallest phone and adapts (two-pane/max-width) on tablet/large screens; foldable/rotation safe.
- [ ] Safe-area insets respected (top + bottom gesture bar).
- [ ] Long content truncates/wraps; empty/missing-media/lots-of-data states handled.
- [ ] Permission-denied has a graceful fallback + recovery path; interruptions preserve state.
- [ ] System settings (theme, contrast, reduce-motion) honored.
