# Motion Recipes — premium *feel*, not just animation

A static screen can be 8/10. The same screen with the right motion is 10/10 — Linear feels precise, Revolut feels luxe, Duolingo feels alive *because of timing, springs, and choreography*. AI animates poorly: wrong durations, no exit, everything at once, bounce everywhere. This file is concrete, copy-paste recipes with the *values* that read premium.

> Two rules above all: **(1) everything that enters must also leave** (AI always forgets exit). **(2) Enter decelerates, exit accelerates.**

---

## 1. Timing recipes (pick by size of the moving thing)

| Tier | Duration | Use |
| --- | --- | --- |
| **Micro** | 80–120ms | press/tap feedback, tiny toggles, ripple |
| **Small** | 180–220ms | chips, checkboxes, small fades, color changes |
| **Medium** | 240–320ms | sheets, cards, list items, content swaps |
| **Large** | 350–500ms | full-screen transitions, hero/shared element |

Rule: bigger travel/area = longer duration. Never animate a small element for 500ms (feels sluggish) or a full screen in 100ms (feels jarring).

---

## 2. Spring recipes (the feel)

Pick a spring personality; don't leave defaults. Approximate cross-framework targets:

| Spring | Feel | Response / damping | Use |
| --- | --- | --- | --- |
| **Gentle** | calm, soft settle | resp ~0.5, damp ~0.9 | content, sheets, calm apps (Notion) |
| **Snappy** | quick, precise, no wobble | resp ~0.30, damp ~0.85 | pro tools, buttons (Linear, Stripe) |
| **Playful** | bouncy, energetic | resp ~0.4, damp ~0.6 | games, delight moments (Duolingo) |
| **Heavy** | weighty, premium | resp ~0.45, damp ~0.8 | finance cards, big elements (Revolut) |

Default to **Snappy** for interaction, **Gentle** for content. Reserve **Playful** bounce for intentionally playful apps — bounce on a banking app feels cheap.

---

## 3. Press feedback (Motion + Touch + Haptic — the trinity)

The single highest-ROI premium upgrade. Every tappable:
```
press down → scale 0.98 (+ slight opacity 0.9) + LIGHT haptic
release    → spring back to 1.0 (snappy)
```

**Compose**
```kotlin
val interaction = remember { MutableInteractionSource() }
val pressed by interaction.collectIsPressedAsState()
val scale by animateFloatAsState(if (pressed) 0.98f else 1f, spring(0.85f, 380f), label = "press")
val haptic = LocalHapticFeedback.current
Box(Modifier.graphicsLayer { scaleX = scale; scaleY = scale }
    .clickable(interaction, indication = ripple()) {
        haptic.performHapticFeedback(HapticFeedbackType.TextHandleMove); onClick()
    })
```

**SwiftUI**
```swift
struct PressStyle: ButtonStyle {
    func makeBody(configuration: Configuration) -> some View {
        configuration.label
            .scaleEffect(configuration.isPressed ? 0.98 : 1)
            .opacity(configuration.isPressed ? 0.9 : 1)
            .animation(.spring(response: 0.3, dampingFraction: 0.85), value: configuration.isPressed)
    }
}
// Button(...).buttonStyle(PressStyle()).sensoryFeedback(.impact(weight: .light), trigger: tapped)
```

**Flutter**
```dart
AnimatedScale(
  scale: _pressed ? 0.98 : 1.0,
  duration: const Duration(milliseconds: 100),
  curve: Curves.easeOut,
  child: GestureDetector(
    onTapDown: (_) { setState(() => _pressed = true); HapticFeedback.selectionClick(); },
    onTapUp: (_) => setState(() => _pressed = false),
    onTapCancel: () => setState(() => _pressed = false),
    child: child,
  ),
);
```

**RN / Expo (Reanimated)**
```tsx
const s = useSharedValue(1);
const style = useAnimatedStyle(() => ({ transform: [{ scale: s.value }] }));
return (
  <Pressable
    onPressIn={() => { s.value = withTiming(0.98, { duration: 100 }); Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light); }}
    onPressOut={() => { s.value = withSpring(1, { damping: 15, stiffness: 300 }); }}>
    <Animated.View style={style}>{children}</Animated.View>
  </Pressable>
);
```

---

## 4. Choreography (parent first, children stagger)

Don't fire everything at once. Lists and groups animate in sequence:
```
Parent / container moves first
Children follow, staggered 20–40ms apart (cap total stagger ~300ms)
Enter: fade + translateY 8–12px → 0 (decelerate)
```

- **Compose:** `LazyColumn` items with `animateItem()`; for entrance, `AnimatedVisibility` + per-index `delayMillis`.
- **SwiftUI:** `.transition(.move(edge:).combined(with: .opacity))` + per-index `.animation(.spring.delay(Double(i) * 0.03))`.
- **Flutter:** `flutter_staggered_animations` or `AnimationController` with `Interval(i*0.05, …)`; `AnimatedList` for insert/remove.
- **RN:** Reanimated `entering={FadeInDown.delay(i*30).springify()}`.

Cap the stagger — 50 items × 40ms = 2s of waiting. Stagger the first ~8 visible, then instant.

---

## 5. Sheet present / dismiss

```
Enter: translateY from 100%→0 over 280–320ms, decelerate + gentle spring; scrim fades 0→0.4
Exit:  translateY 0→100% over 200ms, accelerate; scrim fades out
Snap between detents: spring + light haptic on snap
```
- Compose `ModalBottomSheet`, SwiftUI `.presentationDetents`, Flutter `showModalBottomSheet(isScrollControlled)`, RN `@gorhom/bottom-sheet` `snapPoints` — all support this; tune the spring + add a haptic on snap.

---

## 6. Hero / shared element (list → detail)

The tapped element *becomes* the detail — image + title persist across the push.
- **Compose:** Shared element transitions (`SharedTransitionLayout` + `sharedElement`).
- **SwiftUI:** `matchedGeometryEffect(id:in:)` across the two states / `.navigationTransition(.zoom)`.
- **Flutter:** `Hero(tag:)` on both screens (container transform).
- **RN:** `react-native-reanimated` shared element transitions / `react-native-shared-element`.
Duration: Large tier (350–450ms), standard/emphasized easing. This single move reads instantly premium.

---

## 7. Numeric roll (counts, balances, prices)

Numbers should *roll*, not hard-cut — essential for finance.
- **SwiftUI:** `.contentTransition(.numericText())` + `withAnimation`.
- **Compose:** `AnimatedContent` with a slide/fade `transitionSpec` on the digits.
- **Flutter:** `AnimatedSwitcher` with `SlideTransition`, or a tween over the value + tabular figures.
- **RN:** animate the value with `withTiming` + a rolling-digit component; tabular font.
Always pair with **tabular figures** so digits don't shift width.

---

## 8. Skeleton shimmer (loading)

A subtle light sweep across placeholder blocks shaped like the real layout.
- Compose: animated brush/`Modifier.placeholder` shimmer. SwiftUI: `.redacted(.placeholder)` + a moving gradient mask. Flutter: `shimmer` package or `LinearGradient` + `AnimationController`. RN: `react-content-loader` or Reanimated gradient sweep.
- Sweep ~1000–1400ms, ease-in-out, looping. Color: a hair lighter than the surface, low contrast — barely there.

---

## 9. Exit animations (the thing AI forgets)

For **every** enter, define the exit:
| Enter | Exit |
| --- | --- |
| fade in 200ms | fade out 150ms |
| slide up + fade | slide down + fade, faster |
| scale 0.95→1 | scale 1→0.95 + fade |
| sheet up | sheet down |

Compose `AnimatedVisibility(enter=, exit=)`; SwiftUI `.transition` (asymmetric in/out); Flutter `AnimatedSwitcher`/`AnimatedList` removeItem; RN `exiting={FadeOutDown}`. A list item that animates in but vanishes instantly on delete feels broken.

---

## 10. Reduce Motion (always)

Honor the OS setting — degrade transforms/springs to simple fades (or none), keep functional state changes.
- Compose: check accessibility animation scale. SwiftUI: `@Environment(\.accessibilityReduceMotion)`. Flutter: `MediaQuery.disableAnimations`. RN: `AccessibilityInfo.isReduceMotionEnabled`.

---

## Motion checklist

- [ ] Durations match the size tier (micro→large); nothing sluggish or jarring.
- [ ] Springs have a chosen personality (snappy/gentle/playful/heavy), not defaults.
- [ ] Every tappable has press scale + haptic.
- [ ] Groups/lists are choreographed (parent → staggered children, capped).
- [ ] Sheets/heroes/numerics use the right recipe; numbers roll with tabular figures.
- [ ] **Every enter has a matching exit.**
- [ ] Reduce Motion degrades gracefully.
- [ ] 60fps — animations run on UI/compositor thread (see `premium-polish.md` §10).
