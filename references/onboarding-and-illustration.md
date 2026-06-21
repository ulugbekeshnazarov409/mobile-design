# Onboarding, Empty States & Illustration — the first impression

The first 30 seconds decide whether someone keeps your app. AI defaults to the worst version of this moment: a forced 3-slide stock carousel ("Welcome!" / "Track everything!" / "Get started"), sad-face empty states, and generic clip-art illustration. Premium apps earn attention, show value before features, and make blank screens *inviting*. This file makes onboarding, empty states, illustration, and delight purposeful — not decorative.

> Two rules above all: **(1) onboarding is value, not a feature tour** — show what the user *gets*, not what the app *does*. **(2) an empty state is a call to action, not an apology** — explain + hand over the button that fills it.

See also: `references/flow-architecture.md` (where onboarding sits in the journey), `references/screen-patterns.md` (layout scaffolds), `references/motion-recipes.md` (timing/springs), `references/interaction-patterns.md` (CTA + press feel), `references/psychology.md` (motivation, first-action), `references/notifications-and-native.md` (in-context permission priming).

---

## 1. Onboarding principles (most apps need *less* than they think)

| Principle | Do | Anti-pattern |
| --- | --- | --- |
| **Earn attention** | First slide states one concrete benefit | "Welcome to MyApp 👋" (says nothing) |
| **Value-first** | Show the outcome the user wants | Feature tour of every tab |
| **Minimal** | 2–3 slides *max*; one screen is often best | 5–7 swipes before the app |
| **Always skippable** | Persistent "Skip" top-right | Forced swipe-through, no escape |
| **Single CTA** | One primary action per slide / final "Get started" | Two competing buttons |
| **Permissions later** | Ask in-context at point of use | Notif + location prompts on slide 2 |
| **Progress visible** | Animated dots show length up-front | Hidden length feels endless |

The best onboarding for a tool app is often **zero slides** — drop straight into a populated/sample state and teach by doing. Reserve carousels for apps where the *value needs framing* (habit, health, finance). Never gate the app behind account creation if you can defer it (let people try, sign up to save).

Do **not** request permissions here. Notification, location, contacts, camera prompts belong at the moment they're needed, primed by a custom screen first — see `references/notifications-and-native.md`.

---

## 2. Paged onboarding (React Native) — FlatList + Reanimated parallax

Horizontal `FlatList` with `pagingEnabled`, a shared scroll value, and `interpolate` for per-slide parallax/scale. Crisp at 60fps because everything runs on the UI thread.

```tsx
import { FlatList, View, Text, Pressable, useWindowDimensions } from 'react-native';
import Animated, {
  useSharedValue, useAnimatedScrollHandler, useAnimatedStyle, interpolate, Extrapolation,
} from 'react-native-reanimated';

const AFlatList = Animated.createAnimatedComponent(FlatList);
const SLIDES = [
  { key: '1', title: 'See where your money goes', body: 'Every transaction, sorted automatically.' },
  { key: '2', title: 'Reach goals faster', body: 'Set a target. We do the math.' },
];

export function Onboarding({ onDone }: { onDone: () => void }) {
  const { width } = useWindowDimensions();
  const x = useSharedValue(0);
  const index = useSharedValue(0);
  const ref = useRef<FlatList>(null);

  const onScroll = useAnimatedScrollHandler((e) => {
    x.value = e.contentOffset.x;
    index.value = Math.round(e.contentOffset.x / width);
  });

  return (
    <View style={{ flex: 1 }}>
      <Pressable onPress={onDone} style={{ alignSelf: 'flex-end', padding: 16 }}
        accessibilityRole="button" accessibilityLabel="Skip onboarding">
        <Text style={{ color: '#8A8A8E' }}>Skip</Text>
      </Pressable>

      <AFlatList
        ref={ref} data={SLIDES} horizontal pagingEnabled
        showsHorizontalScrollIndicator={false}
        onScroll={onScroll} scrollEventThrottle={16}
        keyExtractor={(it) => it.key}
        renderItem={({ item, index: i }) => <Slide item={item} i={i} x={x} width={width} />}
      />

      <Dots count={SLIDES.length} x={x} width={width} />
      <Cta index={index} count={SLIDES.length}
        onNext={() => ref.current?.scrollToOffset({ offset: (index.value + 1) * width, animated: true })}
        onDone={onDone} />
    </View>
  );
}

function Slide({ item, i, x, width }) {
  const art = useAnimatedStyle(() => {
    const range = [(i - 1) * width, i * width, (i + 1) * width];
    return {
      transform: [
        { translateX: interpolate(x.value, range, [width * 0.3, 0, -width * 0.3], Extrapolation.CLAMP) }, // parallax
        { scale: interpolate(x.value, range, [0.8, 1, 0.8], Extrapolation.CLAMP) },
      ],
      opacity: interpolate(x.value, range, [0, 1, 0], Extrapolation.CLAMP),
    };
  });
  return (
    <View style={{ width, alignItems: 'center', justifyContent: 'center', padding: 32 }}>
      <Animated.View style={art}>{/* <SvgArt name={item.key} /> or <Lottie/> */}</Animated.View>
      <Text style={{ fontSize: 26, fontWeight: '700', marginTop: 32 }}>{item.title}</Text>
      <Text style={{ fontSize: 16, color: '#6E6E73', textAlign: 'center', marginTop: 8 }}>{item.body}</Text>
    </View>
  );
}
```

**Animated progress dots** — width + opacity interpolate as the active page approaches:

```tsx
function Dots({ count, x, width }) {
  return (
    <View style={{ flexDirection: 'row', justifyContent: 'center', gap: 8, height: 24 }}>
      {Array.from({ length: count }).map((_, i) => {
        const style = useAnimatedStyle(() => {
          const range = [(i - 1) * width, i * width, (i + 1) * width];
          return {
            width: interpolate(x.value, range, [8, 24, 8], Extrapolation.CLAMP),
            opacity: interpolate(x.value, range, [0.4, 1, 0.4], Extrapolation.CLAMP),
          };
        });
        return <Animated.View key={i} style={[{ height: 8, borderRadius: 4, backgroundColor: '#0A84FF' }, style]} />;
      })}
    </View>
  );
}
```

**CTA** — label flips from "Next" to "Get started" on the last slide; drive label off `index` with `useDerivedValue` or a small `runOnJS` state sync. Keep it one full-width primary button (see `references/interaction-patterns.md` for press scale + haptic).

**Other frameworks (brief):**
- **Compose** — `HorizontalPager` (Foundation); read `pagerState.currentPageOffsetFraction` and `lerp` for parallax/scale; `PagerState` for dots.
- **SwiftUI** — `TabView { … }.tabViewStyle(.page)`; bind `selection`; `GeometryReader` minX for parallax; `.indexViewStyle` for dots.
- **Flutter** — `PageView` + `PageController`; `AnimatedBuilder` on `controller.page` for parallax; `smooth_page_indicator` for dots.

---

## 3. Empty states done right — context decides the content

An empty state has three jobs: **say why it's empty**, **say what value appears here**, **give the one action that fills it**. Never "No data." Match the *cause* — first-run, filtered-no-results, error, and cleared are four different screens.

| Type | Tone | Headline + body | Primary action |
| --- | --- | --- | --- |
| **First run** (never had data) | Inviting, opportunity | "Add your first transaction to start tracking." | "Add transaction" (the real CTA) |
| **Filtered / no results** | Neutral, helpful | "No results for 'sushi'." | "Clear filters" / "Search again" |
| **Search not started** | Quiet, guiding | "Search for a city to begin." | (focus the input) |
| **Error / failed load** | Honest, recoverable | "Couldn't load your feed." | "Retry" (+ cached if any) |
| **Cleared / all done** | Positive, rewarding | "Inbox zero. You're all caught up." | secondary or none |
| **Offline** | Reassuring | "You're offline. Showing saved items." | "Try again" |

Rules:
- The CTA must be the **actual primary action** of the screen, not a link to a help page.
- First-run empty states are an onboarding surface — this is where you *teach by doing* instead of a carousel.
- "All done" / cleared states earn a small, warm illustration or line; don't make success look like failure.
- Don't reuse one generic empty component for all four — the copy and CTA change meaning entirely.
- Keep body to one line; never a paragraph. See `references/screen-patterns.md` for the centered empty-state layout (icon/art → headline → body → CTA, vertically centered, generous padding).

---

## 4. Illustration approach — SVG for art, Lottie for motion, restraint always

Illustration is optional. A clean iconographic mark (a single line-icon in your accent, 64–96dp, low opacity) beats clip-art every time. Reach for full illustration only when it carries brand or softens an empty/error moment.

| Need | Use | Why |
| --- | --- | --- |
| Crisp, scalable static art | **react-native-svg** (`<Svg>` paths) / inline SVG | resolution-independent, themeable, tiny |
| Motion / celebration / loading | **Lottie** (`lottie-react-native`, files via `@lottiefiles`) | vector animation, small payload |
| Raster art / photos | **expo-image** | caching, blurhash placeholder, fast decode |
| Quick empty-state mark | one icon (`@expo/vector-icons`) in accent | zero asset weight, on-brand |

On-brand discipline:
- **One accent color** through all illustration; don't ship a rainbow. Tie line/fill to your token palette (`references/color-systems.md`, `references/design-tokens-starter.md`).
- **Respect dark mode** — never ship a white-background or hardcoded-stroke illustration onto a dark surface. Use `currentColor` / theme tokens in SVG, or provide light/dark variants. A bright white illustration on a dark screen is the #1 tell of an unpolished app.
- Keep stroke weight and corner radius consistent with the rest of the UI (one visual language).
- Decorative art is **decorative** — no text baked into the bitmap (breaks localization + Dynamic Type).

**Lottie note (Expo):** `lottie-react-native` requires a **development build** (or bare/EAS); it does **not** run in **Expo Go**. In Expo Go, fall back to a static SVG/expo-image illustration, or gate the Lottie behind a runtime/dev-build check. Pull animations from LottieFiles; keep files small and looped only where it adds value (loaders, success), never a perpetually-animating decoration that drains battery and attention.

---

## 5. Delight moments — earned, sparse, fast

Delight is a seasoning. One well-timed celebration on a *meaningful* first action lands; confetti on every tap is noise. Restraint is what reads premium — see `references/motion-recipes.md` for springs and timing.

| Moment | Treatment | Restraint rule |
| --- | --- | --- |
| First meaningful action (first save, first transfer) | Checkmark draw + soft success haptic | Once, not on repeats |
| Goal completed / streak hit | Brief Lottie or confetti burst (~1s, auto-dismiss) | Playful apps only; reserve for real milestones |
| Form/flow success | Animated check (stroke draw 300–400ms) + label | Don't block the next step |
| Pull-to-refresh / loading | On-brand Lottie loader, not a default spinner | Subtle; don't celebrate routine |
| Micro-reward (XP, badge) | Small count-up + light haptic | No full-screen takeover for tiny wins |

- **Confetti sparingly** (`react-native-confetti-cannon` or a Reanimated/Skia particle burst): only at genuine milestones, ~1s, never repeating.
- **Checkmark draw** beats a static check — animate the path via Reanimated `strokeDashoffset` on an SVG path (300–400ms, gentle).
- Pair visual delight with a **light/success haptic**, never a heavy one (`references/gestures-and-haptics.md`).
- Honor reduce-motion: collapse celebrations to a static end-state (`references/accessibility-deep.md`).

---

## 6. Permission priming inside the onboarding flow

If onboarding ends at a feature that needs a permission, prime it with a **custom screen** *before* the OS dialog — explain the benefit in user terms, with "Allow" (which triggers the native prompt) and "Not now". This single screen roughly doubles grant rates vs. a cold OS prompt, and a "Not now" here doesn't burn your one-shot iOS prompt. Full priming copy, timing, and the native trigger calls live in `references/notifications-and-native.md`. Place the priming screen as a *flow step*, not a slide in the value carousel (`references/flow-architecture.md`).

---

## 7. Accessibility

- **Skippable and reachable** — "Skip" is a real `accessibilityRole="button"` with a label; never a tiny tap target. Onboarding must be completable with VoiceOver/TalkBack and via keyboard/switch.
- **Reading order** — slide content reads title → body → CTA; group with `accessible`/`accessibilityElements` so a swipe doesn't read parallax layers individually.
- **Decorative illustration is marked decorative** — `accessibilityElementsHidden` / `importantForAccessibility="no-hide-descendants"` (RN), `.accessibilityHidden(true)` (SwiftUI), `ExcludeSemantics`/empty `contentDescription` (Compose), `ExcludeSemantics` (Flutter). Don't make a screen reader announce "decorative blob."
- **Meaningful art gets a label** — an illustration that *conveys* the empty state's meaning needs a concise `accessibilityLabel`.
- **Dots aren't the only progress signal** — expose page position to assistive tech (e.g. "Page 2 of 3"); don't rely on color/size alone.
- **Empty-state CTA** is a labeled button announcing its action, focused or easily reachable.
- **Respect reduce-motion + Dynamic Type** — parallax and celebrations degrade gracefully; text in slides scales (no baked-in bitmap text).

---

## Quality checklist (onboarding & empty states)

- [ ] Onboarding is **2–3 slides max** (or zero) and leads with **value, not features**.
- [ ] Persistent, accessible **Skip**; nothing is force-gated that could be deferred.
- [ ] **Single** primary CTA per slide; label becomes "Get started" on the last.
- [ ] Progress dots show length and animate with scroll.
- [ ] **No permission prompts** in the carousel; primed in-context later.
- [ ] Each empty state matches its **cause** (first-run / filtered / error / cleared) with tailored copy + the real CTA.
- [ ] First-run empty state teaches by doing, not by apologizing.
- [ ] Illustration is **on-brand (one accent)**, theme-aware, and **never white-on-dark**.
- [ ] Lottie only where it adds value; Expo Go fallback in place (dev build required).
- [ ] Delight is **rare, fast, and earned** — milestones only, with light haptic, reduce-motion safe.
- [ ] Decorative art marked decorative; meaningful art labeled; flow is screen-reader and switch friendly.
