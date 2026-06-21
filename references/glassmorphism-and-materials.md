# Glassmorphism & System Materials — iOS Liquid Glass and translucent surfaces

Translucent "glass" surfaces (blurred backdrop + thin tint + fine border + soft inner light) are a premium signature when used **sparingly** — a floating nav bar, a sheet over content, a now-playing bar. Overused, they wreck contrast and performance and read as a gimmick. This file is the disciplined recipe layer; apply it from the premium-polish pass, not by default.

> Use glass for **chrome that floats over content** (bars, sheets, controls). Never for primary reading surfaces, dense lists, or text-heavy cards — contrast and legibility lose.

---

## When glass works vs when it fails

| ✅ Good fit | ❌ Wrong fit |
| --- | --- |
| Floating top/bottom bar over scrolling content | Full-screen background of a form/article |
| Bottom sheet / popover over a photo or map | Body text card (text on blur = low contrast) |
| Media controls, now-playing, overlay HUD | Dense data tables, settings lists |
| One hero control that should feel "above" | Every card on the screen (kills hierarchy) |

Hard rules:
- **Contrast still must pass.** Text over glass needs a legibility scrim (a subtle solid fill or darkening layer) so it hits WCAG. Don't trust blur alone.
- **Respect Reduce Transparency / Reduce Motion.** When the OS flag is on, fall back to a **solid** surface — no blur. (SwiftUI materials do this automatically; manual blur must check `UIAccessibility.isReduceTransparencyEnabled` / `AccessibilityInfo`.)
- **Perf.** Real-time blur is GPU-expensive. One or two glass layers per screen, never a scrolling list of them. Prefer the platform's native material (cheap, cached) over a hand-rolled blur view.
- **Layer the depth, don't fake it.** Glass = backdrop blur + ~6–12% tint + 0.5px hairline border (top edge slightly brighter) + a soft shadow under it. A flat semi-transparent rectangle is not glass.

---

## SwiftUI (native materials + iOS 26 Liquid Glass)

System `Material` is the right default — adaptive, accessibility-aware, cheap.

```swift
// Floating bar / card with system material
VStack { /* controls */ }
    .padding(16)
    .background(.ultraThinMaterial, in: RoundedRectangle(cornerRadius: 24, style: .continuous))
    .overlay(
        RoundedRectangle(cornerRadius: 24, style: .continuous)
            .strokeBorder(.white.opacity(0.12), lineWidth: 0.5)   // hairline top-light
    )
    .shadow(color: .black.opacity(0.18), radius: 20, y: 8)
```
Material tiers (thin → thick): `.ultraThinMaterial`, `.thinMaterial`, `.regularMaterial`, `.thickMaterial`, `.bar`. Thinner = more background bleeds through.

**iOS 26 Liquid Glass** — use the first-party API; don't reinvent it with blur:
```swift
// Single element
SomeControl()
    .glassEffect(.regular, in: .capsule)          // .regular / .clear; tint with .tint(...)

// Group elements so they morph/merge together
GlassEffectContainer {
    HStack { buttonA; buttonB; buttonC }
}
```
Guidance: glass belongs on the **navigation/control layer**, floating above content — not on the content itself. Let standard bars (`.toolbar`, `TabView`, `.searchable`) adopt system glass automatically rather than hand-building it.

---

## Jetpack Compose

Compose has no built-in backdrop blur for arbitrary content pre-Android 12. Two paths:

```kotlin
// 1) Tonal/translucent surface (cheapest, works everywhere) — "frosted-ish", not true blur
Surface(
    color = MaterialTheme.colorScheme.surface.copy(alpha = 0.72f),
    tonalElevation = 3.dp,
    shape = RoundedCornerShape(24.dp),
    border = BorderStroke(0.5.dp, Color.White.copy(alpha = 0.12f)),
) { /* content */ }
```

```kotlin
// 2) Real backdrop blur — use the Haze library (handles API levels + RenderEffect)
//    implementation("dev.chrisbanes.haze:haze:<latest>")
val hazeState = remember { HazeState() }
Box {
    LazyColumn(Modifier.haze(hazeState)) { /* scrolling content */ }   // source
    TopAppBar(
        modifier = Modifier.hazeChild(hazeState, style = HazeStyle(tint = Color.White.copy(0.1f), blurRadius = 20.dp)),
        // ...
    )
}
```
Avoid `Modifier.blur()` on the content itself (it blurs the source, not a backdrop). On API < 31 `RenderEffect` is unavailable → Haze falls back to a translucent scrim; design for that fallback. Respect `Settings.Global` reduce-transparency intent → solid surface.

---

## Flutter

```dart
import 'dart:ui';

ClipRRect(
  borderRadius: BorderRadius.circular(24),
  child: BackdropFilter(
    filter: ImageFilter.blur(sigmaX: 20, sigmaY: 20),
    child: Container(
      decoration: BoxDecoration(
        color: Colors.white.withOpacity(0.10),                    // tint
        borderRadius: BorderRadius.circular(24),
        border: Border.all(color: Colors.white.withOpacity(0.14), width: 0.5),
      ),
      child: /* content */,
    ),
  ),
)
```
`BackdropFilter` blurs **everything painted behind it** — it must sit above the content in the stack and be clipped to its shape, or the blur bleeds. For iOS feel, `CupertinoNavigationBar` / `showCupertinoModalPopup` already render system blur. Check `MediaQuery.of(context).disableAnimations` / accessibility and drop to solid when set.

---

## React Native / Expo

```tsx
import { BlurView } from 'expo-blur';            // managed-friendly; or @react-native-community/blur (bare)
import { AccessibilityInfo } from 'react-native';

<BlurView
  intensity={40}                                 // 0–100
  tint="systemMaterial"                           // light | dark | default | systemMaterial* (iOS)
  style={{ borderRadius: 24, overflow: 'hidden', borderWidth: StyleSheet.hairlineWidth, borderColor: 'rgba(255,255,255,0.14)' }}>
  {/* content */}
</BlurView>
```
- **Android caveat:** `expo-blur` blur is weaker / less consistent on Android — verify on a real device; many ship a **solid translucent fallback** on Android and reserve true glass for iOS.
- Add a faint `expo-linear-gradient` top-light over the blur for the "liquid" sheen; keep it ≤ 10% opacity.
- Honor `AccessibilityInfo.isReduceTransparencyEnabled()` (iOS) → render a solid surface instead of `BlurView`.
- For glass bottom sheets use `@gorhom/bottom-sheet` with a `BlurView` background component, not a hand-rolled modal.

---

## Quality checklist (glass)

- [ ] Glass only on floating chrome (bar/sheet/control) — not reading surfaces or every card.
- [ ] Text over glass passes contrast (added scrim if needed), verified in light **and** dark.
- [ ] Reduce Transparency / accessibility flag → solid fallback wired.
- [ ] ≤ 1–2 blur layers per screen; no scrolling list of blurred items (perf).
- [ ] Real depth: backdrop blur + tint + 0.5px top-light border + soft drop shadow — not a flat alpha rectangle.
- [ ] Android RN/Compose fallback designed (blur weaker/absent on older APIs).
- [ ] Used native material (`.material` / `Material` / Cupertino / system bars) before hand-rolling blur.
