# Example Screens — complete gold-standard implementations

A few end-to-end, reference-quality screens with tokens, hierarchy, every state, motion, haptics and a11y already wired together — because concrete beats abstract. **Adapt these, don't paste verbatim**: remap onto the project's real token names, drop states the screen genuinely lacks, swap copy/data shapes. Pattern-match the *bar*, not the literal pixels.

> Cross-ref: `screen-patterns.md` (skeletons these fill), `design-tokens-starter.md` (every token name used below), `data-and-forms.md` (validation + query/mutation), `interaction-patterns.md` (press/haptic), `visual-hierarchy.md` (primary-action decisions).

Each screen states a **goal**, the **hierarchy decision** (the one dominant action), one framework in full, then a short note on the others. Frameworks rotate for full coverage.

---

## 1. Sign-in screen — React Native / Expo (SDK 56, Reanimated 3)

**Goal:** email + password, validate inline, one obvious "Sign in", surface server errors without blame.

**Hierarchy decision:** the **Sign in button** is the only filled accent element on screen — full-width, pinned in the thumb arc above the keyboard. Everything else (logo, fields, "Forgot password", "Create account") is lower contrast so the eye lands on the CTA. The button is the loading surface too; we never show a separate spinner overlay.

Uses `useTheme()` from `design-tokens-starter.md`, `react-hook-form` + `zod` per `data-and-forms.md`, `expo-haptics`, `react-native-keyboard-controller` for keyboard-aware padding.

```tsx
import { useState } from 'react';
import { View, Text, TextInput, Pressable, ActivityIndicator } from 'react-native';
import { SafeAreaView } from 'react-native-safe-area-context';
import { KeyboardAwareScrollView } from 'react-native-keyboard-controller';
import Animated, { useAnimatedStyle, useSharedValue, withSpring, withTiming } from 'react-native-reanimated';
import * as Haptics from 'expo-haptics';
import { useForm, Controller } from 'react-hook-form';
import { zodResolver } from '@hookform/resolvers/zod';
import { z } from 'zod';
import { useTheme } from '@/theme/useTheme';

const Schema = z.object({
  email: z.string().min(1, 'Enter your email').email('That email looks off'),
  password: z.string().min(8, 'At least 8 characters'),
});
type Form = z.infer<typeof Schema>;

export default function SignIn() {
  const t = useTheme();
  const [serverError, setServerError] = useState<string | null>(null);
  const [submitting, setSubmitting] = useState(false);
  const { control, handleSubmit, formState: { errors } } = useForm<Form>({
    resolver: zodResolver(Schema), mode: 'onBlur',
  });

  // CTA press-scale (snappy spring, not a fade)
  const scale = useSharedValue(1);
  const btnStyle = useAnimatedStyle(() => ({ transform: [{ scale: scale.value }] }));

  async function onSubmit(values: Form) {
    setServerError(null);
    setSubmitting(true);
    await Haptics.impactAsync(Haptics.ImpactFeedbackStyle.Light);
    try {
      await signIn(values);                                   // your API call
      await Haptics.notificationAsync(Haptics.NotificationFeedbackType.Success);
    } catch (e) {
      setServerError('Email or password is incorrect.');      // never reveal which
      await Haptics.notificationAsync(Haptics.NotificationFeedbackType.Error);
    } finally {
      setSubmitting(false);
    }
  }

  const Field = ({ name, label, ...props }: any) => (
    <Controller control={control} name={name} render={({ field: { onChange, onBlur, value } }) => {
      const err = errors[name as keyof Form]?.message;
      return (
        <View style={{ gap: t.space.xs }}>
          <Text style={[t.type.label, { color: t.colors.textSecondary }]}>{label}</Text>
          <TextInput
            value={value} onChangeText={onChange} onBlur={onBlur}
            placeholderTextColor={t.colors.textTertiary}
            accessibilityLabel={label}
            style={[t.type.body, {
              height: 52, paddingHorizontal: t.space.base,
              backgroundColor: t.colors.surface, color: t.colors.textPrimary,
              borderRadius: t.radius.md, borderWidth: 1,
              borderColor: err ? t.colors.danger : t.colors.border,
            }]}
            {...props}
          />
          {err ? (
            <Text accessibilityLiveRegion="polite" style={[t.type.caption, { color: t.colors.danger }]}>{err}</Text>
          ) : null}
        </View>
      );
    }} />
  );

  return (
    <SafeAreaView edges={['top', 'bottom']} style={{ flex: 1, backgroundColor: t.colors.background }}>
      <KeyboardAwareScrollView
        bottomOffset={t.space.lg}
        contentContainerStyle={{ flexGrow: 1, padding: t.space.lg, justifyContent: 'center', gap: t.space.lg }}
        keyboardShouldPersistTaps="handled"
      >
        <View style={{ gap: t.space.xs, marginBottom: t.space.base }}>
          <Text style={[t.type.title, { color: t.colors.textPrimary }]}>Welcome back</Text>
          <Text style={[t.type.body, { color: t.colors.textSecondary }]}>Sign in to continue.</Text>
        </View>

        {serverError && (
          <View accessibilityRole="alert" style={{
            flexDirection: 'row', gap: t.space.sm, padding: t.space.md,
            backgroundColor: t.colors.dangerContainer, borderRadius: t.radius.md,
          }}>
            <Text style={[t.type.label, { color: t.colors.danger, flex: 1 }]}>{serverError}</Text>
          </View>
        )}

        <View style={{ gap: t.space.base }}>
          <Field name="email" label="Email" keyboardType="email-address" autoCapitalize="none"
                 autoComplete="email" textContentType="username" returnKeyType="next" />
          <Field name="password" label="Password" secureTextEntry autoComplete="password"
                 textContentType="password" returnKeyType="go" onSubmitEditing={handleSubmit(onSubmit)} />
          <Pressable hitSlop={8} onPress={() => {/* nav to reset */}}
            style={{ alignSelf: 'flex-end' }} accessibilityRole="button">
            <Text style={[t.type.label, { color: t.colors.accent }]}>Forgot password?</Text>
          </Pressable>
        </View>

        <Animated.View style={btnStyle}>
          <Pressable
            disabled={submitting}
            onPressIn={() => { scale.value = withSpring(0.97, { stiffness: 300, damping: 30 }); }}
            onPressOut={() => { scale.value = withSpring(1, { stiffness: 300, damping: 30 }); }}
            onPress={handleSubmit(onSubmit)}
            accessibilityRole="button"
            accessibilityState={{ disabled: submitting, busy: submitting }}
            accessibilityLabel="Sign in"
            style={{
              height: 52, borderRadius: t.radius.md, alignItems: 'center', justifyContent: 'center',
              flexDirection: 'row', gap: t.space.sm,
              backgroundColor: submitting ? t.colors.accentSubtle : t.colors.accent,
              ...t.elevation.e2,
            }}>
            {submitting && <ActivityIndicator color={t.colors.onAccent} />}
            <Text style={[t.type.bodyStrong, { color: submitting ? t.colors.accent : t.colors.onAccent }]}>
              {submitting ? 'Signing in…' : 'Sign in'}
            </Text>
          </Pressable>
        </Animated.View>

        <View style={{ flexDirection: 'row', justifyContent: 'center', gap: t.space.xs }}>
          <Text style={[t.type.body, { color: t.colors.textSecondary }]}>New here?</Text>
          <Pressable hitSlop={8} accessibilityRole="button"><Text style={[t.type.bodyStrong, { color: t.colors.accent }]}>Create account</Text></Pressable>
        </View>
      </KeyboardAwareScrollView>
    </SafeAreaView>
  );
}
```

**Why this reads premium, not AI-generic:** 52pt tap targets; inline errors with a red border that resolves on fix, not an alert dialog; `accessibilityLiveRegion`/`role="alert"` so the error is announced; the button *becomes* the spinner instead of stacking a modal; press uses a snappy spring scale + light haptic, success/error use distinct notification haptics; `textContentType`/`autoComplete` so iOS/Android offer saved passwords; server error stays vague ("Email or password is incorrect") — never leaks which field.

**Other frameworks, key differences:**
- **Compose:** `OutlinedTextField(isError = …, supportingText = {…})` gives error styling for free; gate the button with `enabled`, swap label for `CircularProgressIndicator(Modifier.size(20.dp))`. Keyboard insets via `Modifier.imePadding()`. Haptics via `LocalHapticFeedback.current.performHapticFeedback(HapticFeedbackType.LongPress)`.
- **SwiftUI:** `@FocusState` to move between fields; `.textContentType(.password)` for Keychain autofill; validate in a `@Observable` view model; `.sensoryFeedback(.error, trigger:)` (iOS 17+) for haptics; `.disabled(submitting)` + `ProgressView()` inside the button label.
- **Flutter:** `Form` + `TextFormField(validator:)` + `GlobalKey<FormState>`; `HapticFeedback.lightImpact()`; `Padding(MediaQuery.viewInsetsOf(context).bottom)` for the keyboard; show a `CircularProgressIndicator` swapped into the `FilledButton` child.

---

## 2. Feed / list screen — Jetpack Compose (M3)

**Goal:** scrollable card feed with author + content + actions, pull-to-refresh, and honest loading / empty states.

**Hierarchy decision:** **the content of each card** is primary; the row's avatar + name is the secondary anchor; the action row (like / comment / share) is tertiary, low-contrast until pressed. There is no full-screen FAB competing with the list — a "compose" FAB is allowed but sits at `e3` so it floats above scroll without stealing the first read.

Uses M3 `pullToRefresh` (material3 1.3+), `LazyColumn` with stable keys, and a shimmer skeleton that matches the real card silhouette (per `performance-and-quality.md` — skeletons mirror final layout, never a generic spinner).

```kotlin
import androidx.compose.animation.*
import androidx.compose.foundation.layout.*
import androidx.compose.foundation.lazy.*
import androidx.compose.material3.*
import androidx.compose.material3.pulltorefresh.PullToRefreshBox
import androidx.compose.runtime.*
import androidx.compose.ui.*
import androidx.compose.ui.draw.clip
import androidx.compose.ui.graphics.Brush
import androidx.compose.ui.platform.LocalHapticFeedback
import androidx.compose.ui.hapticfeedback.HapticFeedbackType
import androidx.compose.ui.semantics.*
import androidx.compose.ui.unit.dp

sealed interface FeedState {
    data object Loading : FeedState
    data object Empty : FeedState
    data class Error(val msg: String) : FeedState
    data class Ready(val posts: List<Post>) : FeedState
}
data class Post(val id: String, val author: String, val handle: String, val body: String, val likes: Int, val liked: Boolean)

@OptIn(ExperimentalMaterial3Api::class)
@Composable
fun FeedScreen(state: FeedState, refreshing: Boolean, onRefresh: () -> Unit, onLike: (String) -> Unit) {
    Scaffold(
        topBar = {
            CenterAlignedTopAppBar(
                title = { Text("Home", style = MaterialTheme.typography.titleLarge) },
                colors = TopAppBarDefaults.centerAlignedTopAppBarColors(
                    containerColor = MaterialTheme.colorScheme.background)
            )
        }
    ) { inner ->
        PullToRefreshBox(isRefreshing = refreshing, onRefresh = onRefresh,
            modifier = Modifier.padding(inner).fillMaxSize()) {
            when (state) {
                FeedState.Loading -> LazyColumn(
                    contentPadding = PaddingValues(Space.base),
                    verticalArrangement = Arrangement.spacedBy(Space.md)
                ) { items(6) { SkeletonCard() } }

                FeedState.Empty -> EmptyState(
                    title = "Nothing here yet",
                    body = "Posts from people you follow will show up here.")

                is FeedState.Error -> ErrorState(state.msg, onRetry = onRefresh)

                is FeedState.Ready -> LazyColumn(
                    contentPadding = PaddingValues(Space.base),
                    verticalArrangement = Arrangement.spacedBy(Space.md)
                ) {
                    items(state.posts, key = { it.id }) { post ->
                        PostCard(post, onLike, Modifier.animateItem())   // M3: animateItem (was animateItemPlacement)
                    }
                }
            }
        }
    }
}

@Composable
private fun PostCard(post: Post, onLike: (String) -> Unit, modifier: Modifier = Modifier) {
    val haptic = LocalHapticFeedback.current
    Card(
        modifier = modifier.fillMaxWidth().semantics(mergeDescendants = true) {},
        shape = MaterialTheme.shapes.large,                       // radius.lg
        colors = CardDefaults.cardColors(containerColor = MaterialTheme.colorScheme.surface),
        elevation = CardDefaults.cardElevation(defaultElevation = 1.dp)   // e1; lean on surface+outline
    ) {
        Column(Modifier.padding(Space.base), verticalArrangement = Arrangement.spacedBy(Space.sm)) {
            Row(verticalAlignment = Alignment.CenterVertically,
                horizontalArrangement = Arrangement.spacedBy(Space.sm)) {
                Avatar(post.author)                              // 40.dp circle, initials fallback
                Column {
                    Text(post.author, style = MaterialTheme.typography.bodyLarge.copy(
                        fontWeight = androidx.compose.ui.text.font.FontWeight.SemiBold))
                    Text("@${post.handle}", style = MaterialTheme.typography.labelSmall,
                        color = MaterialTheme.colorScheme.onSurfaceVariant)
                }
            }
            Text(post.body, style = MaterialTheme.typography.bodyLarge,
                color = MaterialTheme.colorScheme.onSurface)
            Row(horizontalArrangement = Arrangement.spacedBy(Space.lg)) {
                val likeLabel = if (post.liked) "Unlike" else "Like"
                TextButton(
                    onClick = { haptic.performHapticFeedback(HapticFeedbackType.LongPress); onLike(post.id) },
                    modifier = Modifier.semantics { contentDescription = "$likeLabel, ${post.likes} likes" }
                ) {
                    Text(if (post.liked) "♥ ${post.likes}" else "♡ ${post.likes}",
                        color = if (post.liked) MaterialTheme.colorScheme.primary
                                else MaterialTheme.colorScheme.onSurfaceVariant,
                        style = MaterialTheme.typography.labelLarge)
                }
                TextButton(onClick = { /* comment */ }) { Text("Reply",
                    style = MaterialTheme.typography.labelLarge,
                    color = MaterialTheme.colorScheme.onSurfaceVariant) }
            }
        }
    }
}

@Composable
private fun SkeletonCard() {
    val brush = rememberShimmerBrush()                           // animated linear-gradient sweep
    Card(shape = MaterialTheme.shapes.large,
        colors = CardDefaults.cardColors(containerColor = MaterialTheme.colorScheme.surface)) {
        Column(Modifier.padding(Space.base), verticalArrangement = Arrangement.spacedBy(Space.sm)) {
            Row(horizontalArrangement = Arrangement.spacedBy(Space.sm)) {
                Box(Modifier.size(40.dp).clip(MaterialTheme.shapes.large).background(brush))
                Column(verticalArrangement = Arrangement.spacedBy(6.dp)) {
                    Box(Modifier.height(14.dp).width(120.dp).clip(MaterialTheme.shapes.small).background(brush))
                    Box(Modifier.height(12.dp).width(80.dp).clip(MaterialTheme.shapes.small).background(brush))
                }
            }
            Box(Modifier.height(14.dp).fillMaxWidth().clip(MaterialTheme.shapes.small).background(brush))
            Box(Modifier.height(14.dp).fillMaxWidth(0.7f).clip(MaterialTheme.shapes.small).background(brush))
        }
    }
}

@Composable
private fun EmptyState(title: String, body: String) =
    Column(Modifier.fillMaxSize().padding(Space.xl),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center) {
        Text(title, style = MaterialTheme.typography.titleLarge)
        Spacer(Modifier.height(Space.sm))
        Text(body, style = MaterialTheme.typography.bodyLarge,
            color = MaterialTheme.colorScheme.onSurfaceVariant,
            textAlign = TextAlign.Center)
    }

@Composable
private fun ErrorState(msg: String, onRetry: () -> Unit) =
    Column(Modifier.fillMaxSize().padding(Space.xl),
        horizontalAlignment = Alignment.CenterHorizontally,
        verticalArrangement = Arrangement.Center) {
        Text("Couldn't load your feed", style = MaterialTheme.typography.titleLarge)
        Spacer(Modifier.height(Space.sm))
        Text(msg, style = MaterialTheme.typography.bodyLarge,
            color = MaterialTheme.colorScheme.onSurfaceVariant, textAlign = TextAlign.Center)
        Spacer(Modifier.height(Space.base))
        FilledTonalButton(onClick = onRetry) { Text("Try again") }
    }
```

**Why this reads premium:** skeleton silhouettes mirror the real card (avatar circle + two name lines + two body lines) so the swap-in is seamless, not a centered spinner; `key = { it.id }` + `animateItem()` makes new posts slide in instead of teleporting; the like action carries a haptic and a merged `contentDescription` ("Unlike, 24 likes") so TalkBack reads the whole card as one node; cards sit at `e1` (lean on surface + outline) rather than drop-shadowed boxes; empty/error are *designed* screens with a retry, not a toast. `EmptyState` and `ErrorState` are different — empty means "succeeded, nothing to show", error means "failed, here's a retry".

**Other frameworks, key differences:**
- **SwiftUI:** `List` with `.listStyle(.plain)` + `.refreshable { await reload() }` (built-in pull-to-refresh); `redacted(reason: .placeholder)` for skeletons; `ContentUnavailableView("No posts", systemImage: "tray")` (iOS 17+) is the canonical empty state.
- **Flutter:** `RefreshIndicator` + `ListView.builder`; `shimmer` package or an `AnimatedBuilder` gradient for skeletons; `Card` with `elevation: 1`; one `Sliver`-based `CustomScrollView` if you need a collapsing header.
- **RN/Expo:** `FlashList` (not `FlatList`) for recycling, `onRefresh`/`refreshing` props, `ListEmptyComponent` for empty, a `MotiView`/Reanimated shimmer for skeletons.

---

## 3. Settings screen — SwiftUI (grouped list, large title)

**Goal:** account + preferences + destructive actions, scannable, native-feeling, dark-mode-correct via semantic colors.

**Hierarchy decision:** there is **no single CTA** — a settings screen's job is navigation and toggles, so hierarchy comes from *grouping and the destructive row*. The "Sign out" / "Delete account" rows are the only colored (`.red`) elements and live in their own trailing section, visually quarantined so they're never tapped by accident.

Uses `Form` + `Section` (system grouped insets, separators, and dark mode for free), SF Symbols for row icons, `@AppStorage`/`@Observable` for live toggles, and `.confirmationDialog` for the destructive path. Lean on **semantic** system colors (`.primary`, `.secondary`, `Color(.systemGroupedBackground)`) so appearance is automatic.

```swift
import SwiftUI

struct SettingsScreen: View {
    @AppStorage("pushEnabled") private var pushEnabled = true
    @AppStorage("haptics") private var haptics = true
    @AppStorage("appearance") private var appearance = "system"
    @State private var confirmingSignOut = false
    let user: User

    var body: some View {
        NavigationStack {
            Form {
                // Account header — tappable profile row
                Section {
                    NavigationLink(value: Route.account) {
                        HStack(spacing: Theme.Space.md) {
                            Avatar(user)                       // 56pt circle
                            VStack(alignment: .leading, spacing: 2) {
                                Text(user.name).font(.headline)
                                Text(user.email).font(.subheadline).foregroundStyle(.secondary)
                            }
                        }
                        .padding(.vertical, 6)
                    }
                    .accessibilityHint("Opens account details")
                }

                Section("Preferences") {
                    Toggle(isOn: $pushEnabled) {
                        Label("Notifications", systemImage: "bell.badge")
                    }
                    Toggle(isOn: $haptics) {
                        Label("Haptic feedback", systemImage: "hand.tap")
                    }
                    Picker(selection: $appearance) {
                        Text("System").tag("system")
                        Text("Light").tag("light")
                        Text("Dark").tag("dark")
                    } label: {
                        Label("Appearance", systemImage: "circle.lefthalf.filled")
                    }
                }
                .tint(.accent)                                  // toggles/picker use brand accent

                Section("Support") {
                    NavigationLink { HelpView() } label: {
                        Label("Help center", systemImage: "questionmark.circle")
                    }
                    Link(destination: URL(string: "mailto:support@app.com")!) {
                        Label("Contact us", systemImage: "envelope")
                    }
                    LabeledContent {
                        Text(appVersion).foregroundStyle(.secondary).monospacedDigit()
                    } label: {
                        Label("Version", systemImage: "info.circle")
                    }
                }

                // Destructive section — quarantined, the only red on screen
                Section {
                    Button(role: .destructive) {
                        confirmingSignOut = true
                    } label: {
                        Label("Sign out", systemImage: "rectangle.portrait.and.arrow.right")
                    }
                }
            }
            .navigationTitle("Settings")
            .navigationBarTitleDisplayMode(.large)             // large title that shrinks on scroll
            .scrollContentBackground(.hidden)
            .background(Color(.systemGroupedBackground))
            .confirmationDialog("Sign out of your account?",
                                isPresented: $confirmingSignOut, titleVisibility: .visible) {
                Button("Sign out", role: .destructive) { signOut() }
                Button("Cancel", role: .cancel) {}
            } message: {
                Text("You'll need to sign in again to continue.")
            }
        }
    }
}
```

**Why this reads premium:** native grouped `Form` gives correct inset separators, section headers, and automatic light/dark — re-implementing this by hand is the #1 AI tell; every row pairs an SF Symbol with its label via `Label` so icons align on a consistent leading rail; toggles flip instantly because `@AppStorage` is the source of truth (no save button); the large title collapses into the nav bar on scroll; destructive actions are `role: .destructive` (system red + bold) *and* gated behind a `.confirmationDialog` with a plain-language consequence; version uses `.monospacedDigit()` so it doesn't jitter. Crucially it uses **semantic** colors (`.secondary`, `systemGroupedBackground`) — never hardcoded grays — so Increase Contrast and dark mode just work.

**Flutter equivalent (short note):** there's no `Form`-grouped equivalent, so build it with `ListView` + `Card`s (one per section, `radius.lg`) or the `flutter/settings_ui`-style pattern: a section is a `Column` of `ListTile`s wrapped in a rounded `Container`/`Card`, header is a small uppercase `Text` in `onSurfaceVariant`. Use `SwitchListTile` for toggles, `ListTile(trailing: Icon(Icons.chevron_right))` for nav rows, and an `AppBar(title: …, largeTitle)` via `SliverAppBar.large` in a `CustomScrollView` to mimic the iOS large title. Destructive row: `ListTile(textColor: theme.colorScheme.error, leading: Icon(Icons.logout, color: error))` behind a `showDialog`/`AlertDialog` confirm. Toggle state in a `ChangeNotifier`/`Riverpod` provider or `shared_preferences`. Keep the destructive action in its own trailing card, same quarantine rule.

---

## Quality checklist (example screens)

- [ ] One screen = one job; the **primary action is unmistakably dominant** (or, for settings, hierarchy comes from grouping + a quarantined destructive row).
- [ ] All relevant states implemented: **loading skeleton that mirrors the final layout**, distinct empty vs error (error has retry), success path with feedback.
- [ ] Every value comes from semantic tokens (`design-tokens-starter.md` names) / native semantic colors — no raw hex, no magic spacing, no hardcoded grays.
- [ ] Tap targets ≥ 44–48pt; press feedback is a spring/scale or system ripple, paired with a **haptic** (light on press, success/error on result).
- [ ] Safe areas + keyboard insets handled (`SafeAreaView`/`imePadding`/`@FocusState`/`viewInsets`).
- [ ] A11y: every control labeled; form errors announced (`role="alert"`/live region); cards merge into one node; `accessibilityState` busy/disabled on the loading CTA.
- [ ] Forms validate inline (border + caption that resolves on fix), server errors stay vague and non-blaming; the button becomes the spinner — no stacked modal.
- [ ] Lists use stable keys + item-placement animation; numbers use tabular/monospaced figures so they don't jitter.
- [ ] Dark mode verified; motion respects reduce-motion; current APIs only (Expo SDK 56, Reanimated 3, Compose M3 `animateItem`/`PullToRefreshBox`, SwiftUI `ContentUnavailableView`/`.confirmationDialog`, Flutter M3).
```