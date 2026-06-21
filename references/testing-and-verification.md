# Testing & Verification — prove it works

"It compiles" is not "it works", and "it works on my simulator" is not "it ships". Premium UI is verified with **evidence**: it runs on a real device, holds up across sizes and dark mode, survives the largest text scale and RTL, degrades gracefully offline, and reads correctly under a screen reader. This file is the verification + UI-testing layer — the cheap manual pass you should always run, the automated tests worth writing, and the gate that must be green before declaring done.

> Cross-ref: `error-handling-and-diagnostics.md` (the lint/typecheck/console gate this builds on), `accessibility-deep.md` (what a11y tests assert), `real-world-constraints.md` (the device/text/RTL matrix), `performance-and-quality.md` (jank/60fps verification).

---

## 1. The manual verification matrix — always run this first

This is the highest-value, lowest-cost pass. Most "premium" regressions (clipped text, invisible dark-mode borders, broken RTL, unreachable buttons) are caught in minutes by physically exercising these dimensions. Do it — or explicitly tell the user it's outstanding. Don't claim a screen is done without it.

| Dimension | How to check | What breaks if you skip it |
| --- | --- | --- |
| Real device / sim | Run on an actual device or simulator, not just preview/canvas | Touch targets, perf, safe-area insets only show on real hardware |
| Light + dark | Toggle system theme; flip in-app if supported | Hardcoded colors, invisible borders, low-contrast text in dark |
| Smallest phone | iPhone SE / small Android (~360dp wide) | Overflow, clipped labels, 2-line buttons, cramped spacing |
| Tablet / large | iPad / large Android, landscape | Stretched full-width content, no max-width, broken nav |
| Largest Dynamic Type | Settings → max accessibility text size | Truncation, overlap, fixed-height rows clipping |
| RTL | Pseudo-locale / force RTL (Arabic) | Hardcoded left/right padding, mis-mirrored chevrons/back |
| Offline | Airplane mode mid-flow | Infinite spinners, no error state, lost input |
| Slow network | Network Link Conditioner / throttle | No skeletons, layout jump on load, double-submit |
| Screen reader | Swipe-through with VoiceOver / TalkBack | Unlabeled controls, wrong order, decorative noise, no announcements |
| Rotation / multitask | Rotate; iPad split view | State loss, layout collapse |

Toggles: **iOS** — Accessibility Inspector (text size slider, color filters), Simulator → Features → Toggle Appearance, `Edit Scheme → Run → Options` for "Right-to-left layout" pseudolanguage and Dynamic Type. **Android** — Developer Options → "Force RTL layout direction", Settings → Display → Font/Display size, dark theme toggle; `adb shell settings put global` for animations. **RN/Expo** — `I18nManager.forceRTL(true)`, Expo Go appearance toggle.

---

## 2. Component testing (React Native) — @testing-library/react-native

Query by **role and accessible label**, not test IDs where possible — this ties testability directly to accessibility (see `accessibility-deep.md`): if your test can't find the button by its label, neither can a screen-reader user. Use `findBy*` (async) for anything that appears after a state change.

```tsx
import { render, screen, fireEvent, waitFor } from '@testing-library/react-native';
import { LoginScreen } from './LoginScreen';

test('submits and shows success', async () => {
  render(<LoginScreen onSubmit={jest.fn()} />);

  // query by role + accessible name (a11y == testability)
  const email = screen.getByLabelText('Email');
  const submit = screen.getByRole('button', { name: 'Log in' });

  fireEvent.changeText(email, 'a@b.com');
  fireEvent.press(submit);

  // async result — findBy* waits for it to appear
  expect(await screen.findByText('Welcome back')).toBeOnTheScreen();

  // validation path
  await waitFor(() => expect(screen.queryByText('Required')).toBeNull());
});
```

Queries: `getBy*` (must exist now), `queryBy*` (may be absent — assert null), `findBy*` (async, retries). Prefer `getByRole`/`getByLabelText`/`getByText` over `getByTestId`. Use `@testing-library/jest-native` matchers (`toBeOnTheScreen`, `toBeDisabled`, `toHaveAccessibilityValue`). Mock network at the boundary (MSW or jest mocks), not the component.

---

## 3. E2E with Maestro — the easiest real-flow test

Maestro is the lowest-friction E2E option: flows are **plain YAML**, run against the installed app on device/simulator with no build-tool coupling, auto-wait for elements, and tolerate minor UI changes. Ideal for verifying the critical user journey (launch → auth → primary action). Reach for it first for E2E.

```yaml
# flows/login.yaml
appId: com.example.app
---
- launchApp:
    clearState: true
- assertVisible: "Log in"
- tapOn: "Email"
- inputText: "a@b.com"
- tapOn: "Password"
- inputText: "secret123"
- tapOn: "Log in"
- assertVisible: "Welcome back"      # primary success state
- tapOn:
    id: "tab-profile"                 # by accessibility id
- assertVisible: "Edit profile"
```

Run: `maestro test flows/login.yaml`; record with `maestro studio`. Selectors match visible text or accessibility id/label — another reason labels must be correct. Use `runFlow`/subflows for shared steps, `env` for data.

**Detox** is the heavier RN alternative: gray-box, synchronizes with the app's async work, but needs a native debug build, more config, and CI plumbing. Choose Detox when you need deterministic sync with native animations/timers or tight RN integration; choose Maestro for fast, robust black-box flows.

---

## 4. Visual regression — snapshots & golden tests

Catches unintended pixel/layout drift. Powerful but maintenance-heavy: snapshots must be reviewed on update or they rubber-stamp regressions.

| Approach | Tool | Pros | Cons |
| --- | --- | --- | --- |
| Render-tree snapshot | jest `toMatchSnapshot` (RN) | Free, fast, no device | Not pixels; noisy diffs; easy to blind-update |
| Golden image | Flutter `matchesGoldenFile` | True pixels, per-platform | Font/AA differences across machines → CI image consistency needed |
| Story screenshots | Storybook + screenshot (e.g. Chromatic / reg tools) | Per-component, real render, review UI | Infra + cost; flaky without pinned env |
| iOS snapshot | `swift-snapshot-testing` | Pixel + accessibility/text-size variants | Record on one ref env |

Rules: snapshot **small components and states**, not whole flaky screens. Pin font/locale/timezone. Always **review** a changed golden — never `--update` blindly. Generate variants (dark, largest text, RTL) as separate snapshots to lock those dimensions in.

---

## 5. Accessibility testing — automated + human

Automated checks find missing labels and contrast failures; only a real swipe-through proves the screen is *usable*. Do both. See `accessibility-deep.md` for what correct semantics look like.

- **Android — Accessibility Scanner** (Play Store app): scans a live screen for small targets, low contrast, missing labels. **TalkBack**: swipe right through every element — each stop should read label + role + state, in logical order, no decorative noise.
- **iOS — Accessibility Inspector** (Xcode → Developer Tools): audit button runs WCAG-style checks; inspect each element's label/traits/value. **VoiceOver**: swipe-through on device.
- **Automated assertions in test:** RN `getByRole`/`toHaveAccessibilityState`; Compose `assert(hasContentDescription)`/`onNodeWithContentDescription`; Flutter `meetsGuideline(textContrastGuideline)`, `androidTapTargetGuideline`, `labeledTapTargetGuideline`; XCUITest can query `accessibilityLabel`.
- Assert: every interactive element has a name; loading/results are announced (live region); focus order is logical; nothing color-only.

```dart
// Flutter automated a11y guidelines
testWidgets('meets a11y guidelines', (tester) async {
  final handle = tester.ensureSemantics();
  await tester.pumpWidget(const MyApp());
  await expectLater(tester, meetsGuideline(textContrastGuideline));
  await expectLater(tester, meetsGuideline(androidTapTargetGuideline));
  await expectLater(tester, meetsGuideline(labeledTapTargetGuideline));
  handle.dispose();
});
```

---

## 6. Per-framework UI test snippets

### Jetpack Compose — semantics test (`createComposeRule`)
```kotlin
@get:Rule val rule = createComposeRule()

@Test fun loginShowsErrorOnEmpty() {
  rule.setContent { LoginScreen() }
  rule.onNodeWithText("Log in").performClick()
  rule.onNodeWithText("Email is required").assertIsDisplayed()
  rule.onNodeWithContentDescription("Email").performTextInput("a@b.com")
  rule.onNode(hasText("Log in") and hasClickAction()).assertIsEnabled()
}
```
Find by semantics (`onNodeWithText`, `onNodeWithContentDescription`, `hasRole`), act (`performClick`/`performTextInput`/`performScrollTo`), assert (`assertIsDisplayed`, `assertIsEnabled`, `assertIsSelected`). Same semantics tree the screen reader uses.

### SwiftUI — XCUITest (and ViewInspector for unit-level)
```swift
func testLoginFlow() {
  let app = XCUIApplication(); app.launch()
  app.textFields["Email"].tap(); app.textFields["Email"].typeText("a@b.com")
  app.buttons["Log in"].tap()
  XCTAssertTrue(app.staticTexts["Welcome back"].waitForExistence(timeout: 5))
}
```
XCUITest drives the app by accessibility identifiers/labels (set `.accessibilityIdentifier`). For fast in-process view assertions without the simulator UI, use **ViewInspector** to inspect the SwiftUI view tree directly.

### Flutter — widget test
```dart
testWidgets('shows error then succeeds', (tester) async {
  await tester.pumpWidget(const MyApp());
  await tester.tap(find.text('Log in'));
  await tester.pump();                         // rebuild
  expect(find.text('Email is required'), findsOneWidget);

  await tester.enterText(find.bySemanticsLabel('Email'), 'a@b.com');
  await tester.tap(find.text('Log in'));
  await tester.pumpAndSettle();                // wait for async + animations
  expect(find.text('Welcome back'), findsOneWidget);
});
```
Widget tests are fast and headless; use `find.bySemanticsLabel`/`find.byTooltip` to keep them tied to a11y. Use `integration_test` to run real flows on a device/emulator.

---

## 7. What to actually assert

Don't test trivia (a label equals its constant). Test that the **screen behaves**:

- **States render:** loading → skeleton/spinner; empty → empty state; error → error + retry; success → content. Each is a real assertion, not just the happy path.
- **Primary action works:** the main CTA fires its handler and produces the expected result/navigation.
- **Navigation:** tapping a row/tab lands on the right destination; back returns state.
- **Form validation:** required/invalid input shows the error; valid input clears it and enables submit; no double-submit.
- **Edge data doesn't crash:** empty list, one item, huge list, very long strings, missing/null fields, emoji/RTL text, 0 and negative numbers.
- **No regression in a11y dimensions:** controls remain labeled; layout survives largest text (assert no truncation where it matters).

---

## 8. The ship gate

Verification builds on the diagnostics gate in `error-handling-and-diagnostics.md` — tests are necessary, not sufficient. **All of these must be green before declaring done:**

1. **Typecheck / compile** — zero errors (`tsc --noEmit`, `flutter analyze`, Xcode build, `./gradlew assembleDebug`).
2. **Lint** — zero warnings (ESLint, `ktlint`, `swiftlint`, `dart analyze`).
3. **Tests** — component/widget/unit suites pass; critical E2E flow passes.
4. **Console clean** — no red/yellow/stray logs while running.
5. **Manual matrix** — §1 exercised (at minimum: device, dark, smallest, largest text, screen-reader swipe).

If any are skipped (e.g. no device available), say so explicitly rather than implying full verification.

---

## Quality checklist (testing & verification)

- [ ] Ran the **manual matrix**: real device, light+dark, smallest phone, tablet, largest Dynamic Type, RTL, offline, slow network.
- [ ] **Swipe-through** done with VoiceOver/TalkBack; every control labeled, order logical, async results announced.
- [ ] Accessibility Scanner / Inspector run with no flagged target-size or contrast issues.
- [ ] **All four states** (loading / empty / error / success) verified — manually and/or asserted in a test.
- [ ] Primary action, navigation, and **form validation** covered by tests.
- [ ] Edge data (empty, single, huge, long/RTL/emoji strings, null fields) does not crash.
- [ ] Critical journey covered by a **Maestro** (or Detox) E2E flow.
- [ ] Visual snapshots/goldens cover key states; changes were **reviewed**, not blind-updated.
- [ ] Tests use **role/label queries** (a11y-tied), not brittle test IDs where avoidable.
- [ ] Ship gate green: typecheck + lint + tests + clean console all pass before declaring done.
