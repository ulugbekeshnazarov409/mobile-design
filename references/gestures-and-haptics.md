# Gestures & Haptics — half of the premium feeling

Premium apps respond to *touch*, not just taps: swipe to delete, drag to dismiss, pull to refresh, long-press menus — each with the right physics and a matching haptic. Haptics are the most-skipped premium detail; a well-placed light tap on selection makes an app feel real. **Gesture + Motion + Haptic is one trinity** — never ship a gesture without its feedback.

> Pair this with `motion-recipes.md`. A gesture that moves something must use a motion recipe; the moment it commits must fire a haptic.

---

## 1. Haptic mapping (which feedback for which action)

Use the right *weight* — over-buzzing is as bad as none. Map by meaning, not by every tap:

| Action | Haptic |
| --- | --- |
| Selection change (tab, segment, picker, toggle row) | **selection / light** |
| Button press (primary action) | **light impact** |
| Toggle on/off, checkbox | **light/medium impact** |
| Pull-to-refresh crosses threshold | **medium impact** |
| Sheet snaps to a detent | **light impact** |
| Swipe action reveals/commits (delete) | **medium impact** |
| Long-press menu opens | **medium impact** |
| Success (payment, send, save) | **success notification** |
| Warning (validation, limit) | **warning notification** |
| Error (failed action) | **error notification** |
| Drag pickup / reorder grab | **medium impact** |

Rules: **don't** haptic on scroll, on every keystroke, or on passive UI. **Do** haptic on commit moments and selection. One clear tap > constant buzzing.

### Per-framework APIs
- **iOS / SwiftUI:** `.sensoryFeedback(.selection / .impact(weight:) / .success / .warning / .error, trigger:)`; or `UIImpactFeedbackGenerator(style:)` / `UINotificationFeedbackGenerator()` — prepare() then trigger.
- **Android / Compose:** `LocalHapticFeedback.current.performHapticFeedback(HapticFeedbackType.LongPress / TextHandleMove)`; or `view.performHapticFeedback(HapticFeedbackConstants.*)` (richer set: CONFIRM, REJECT, CLOCK_TICK, SEGMENT_TICK on newer APIs).
- **Flutter:** `HapticFeedback.selectionClick() / lightImpact() / mediumImpact() / heavyImpact()`. (Notification-style success/error → use a package or platform channel.)
- **RN / Expo:** `expo-haptics` — `Haptics.selectionAsync()`, `Haptics.impactAsync(ImpactFeedbackStyle.Light|Medium|Heavy)`, `Haptics.notificationAsync(NotificationFeedbackType.Success|Warning|Error)`.

---

## 2. Press (the baseline gesture)

```
press in  → scale 0.98 + opacity 0.9 + LIGHT haptic
press out → spring back (snappy)
long press → scale 0.96 + MEDIUM haptic → reveal menu
```
See `motion-recipes.md §3` for full per-framework code. This is non-negotiable on every interactive element.

---

## 3. Swipe actions (rows: delete, archive, etc.)

```
drag follows finger with resistance (rubber-band past threshold)
reveal action background (color + icon) as it opens
cross commit threshold → MEDIUM haptic
release past threshold → snap to action / execute; release before → spring back
```
- **SwiftUI:** `.swipeActions(edge:) { Button(role: .destructive){…} }` (system handles feel) — for custom, `DragGesture` + offset.
- **Compose:** `SwipeToDismissBox` / `AnchoredDraggable`; fire haptic when state passes threshold.
- **Flutter:** `Dismissible` (with `confirmDismiss`) or `flutter_slidable`; haptic in `onUpdate`/threshold.
- **RN:** `react-native-gesture-handler` `Swipeable` / reanimated `Gesture.Pan()`; haptic when `translationX` crosses threshold.

Always animate the row's exit on commit (don't hard-remove) — see motion §9.

---

## 4. Drag to dismiss (sheets, full-screen modals, image viewers)

```
drag down follows finger; background scrim fades with progress; content scales slightly (0.95)
past threshold (≈ 100–150px or velocity) → dismiss with downward motion
below threshold → spring back to place
LIGHT haptic on successful dismiss
```
- SwiftUI sheets do this natively; custom via `DragGesture` + offset + opacity. Compose `ModalBottomSheet` (drag built-in) or `AnchoredDraggable`. Flutter `DraggableScrollableSheet` / gesture + `Navigator.pop`. RN `@gorhom/bottom-sheet` (built-in) or reanimated pan.

---

## 5. Pull to refresh (make it feel weighted)

```
overscroll past top → custom indicator appears, tracks drag (rubber-band)
cross threshold → MEDIUM haptic + indicator "armed"
release → spring + run refresh; indicator spins; settle back on complete
```
- SwiftUI `.refreshable { await load() }`. Compose `PullToRefreshBox`. Flutter `RefreshIndicator` / `CupertinoSliverRefreshControl`. RN `RefreshControl` on the list, or reanimated custom for a branded feel.
Add the threshold haptic even when using the system control where possible — it's the detail that sells it.

---

## 6. Long-press / context menu

```
long press (~400ms) → MEDIUM haptic + element lifts (scale 1.02 + shadow) → menu appears (scale/fade from the element)
```
- SwiftUI `.contextMenu` (native preview + haptic). Compose `combinedClickable(onLongClick=)` + `DropdownMenu`. Flutter `onLongPress` + `showMenu`/`CupertinoContextMenu`. RN gesture-handler `Gesture.LongPress()` + menu lib.

---

## 7. Drag to reorder

```
long-press grab → MEDIUM haptic + item lifts (scale + shadow), others make space (animated)
drag → live reflow; LIGHT/segment haptic as it passes each slot
drop → spring into place + LIGHT haptic
```
- SwiftUI `List` `.onMove` (with `EditMode`) or `draggable`/`dropDestination`. Compose `LazyColumn` + `Modifier.draggable`/reorderable lib. Flutter `ReorderableListView`. RN `react-native-draggable-flatlist`.

---

## 8. Other gestures (use where they fit)

- **Double-tap to like** (+ heart pop + light haptic) — social/media.
- **Pinch to zoom** on images/maps (gesture-handler / native).
- **Horizontal swipe between tabs/pages** (paged scroll, synced indicator).
- **Edge swipe back** (iOS — never break it; it's expected).

Don't pile gestures on; add the ones the pattern expects. Hidden gestures need a visible affordance or they won't be discovered.

---

## Gestures & haptics checklist

- [ ] Every tappable: press scale + light haptic.
- [ ] Gestures the pattern expects are present (swipe-delete on rows, drag-dismiss on sheets, pull-refresh on lists).
- [ ] Each gesture has resistance/rubber-band and a clear threshold.
- [ ] A haptic fires at every **commit** moment (threshold cross, snap, success/error) — mapped by weight.
- [ ] No haptic spam (not on scroll/keystroke/passive UI).
- [ ] Gesture-moved elements use a motion recipe; commits animate (no hard cut).
- [ ] iOS edge-swipe-back intact; gestures don't trap the user.
