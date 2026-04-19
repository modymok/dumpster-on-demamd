---
type: "manual"
description: "Mobile rules for Expo / React Native in dumpster-user-app and dumpster-driver-app. Loaded by the mobile-dev subagent."
---

# Role Rules — Mobile (Expo / React Native)

Applies to: `dumpster-user-app/` and `dumpster-driver-app/`.

## 1. Stack Contract
- **Expo SDK** (managed workflow unless the project explicitly requires bare).
- **React Native** with **Expo Router** for file-based routing in `app/`.
- **NativeWind** for styling (Tailwind-in-RN). No raw `StyleSheet.create` unless NativeWind can't express it.
- **Zustand** for client state. **TanStack Query** for server state.
- **React Hook Form** + **Zod** for forms.
- **Supabase JS** client configured with `AsyncStorage` (or `expo-secure-store` for auth session).

## 2. Navigation
- File-based routes under `app/`. Nested routes use folders + `_layout.tsx`.
- Use typed routes — `href` values must satisfy the generated `Href<T>` type.
- Deep links: register schemes in `app.config.js`; test on a real device before shipping.

## 3. Styling & Layout
- Use NativeWind classes with **logical utilities only** on the horizontal axis (`ms-*`, `me-*`, `ps-*`, `pe-*`, `start-*`, `end-*`). Direction (RTL / LTR) policy — see §13.
- Safe-area handling via `react-native-safe-area-context`. Every screen root uses `<SafeAreaView>` or `useSafeAreaInsets`.
- Avoid absolute positioning for layout; prefer flexbox.
- Images use `expo-image` with explicit `width`/`height`; prefer `contentFit="cover"` over `resizeMode`.

## 4. Platform Differences
- Use `Platform.OS` checks sparingly; prefer Expo modules that abstract differences.
- iOS: verify safe areas on notch + Dynamic Island devices.
- Android: verify status bar color and keyboard avoidance.
- Test on at least one physical device before releasing a feature that touches camera, location, push, or payments.

## 5. Permissions
- Request permissions **just in time**, with a rationale screen first.
- Never call a permission API on app launch unless essential.
- Gracefully degrade if denied — don't block the user from the rest of the app.

## 6. Secrets & Env
- Public config via `EXPO_PUBLIC_*` (safe to ship to the client).
- Secrets (API signing keys, service roles) **never** go in `EXPO_PUBLIC_*`.
- Build-time secrets via **EAS secrets** (`eas secret:create`).
- Supabase anon key is fine in client; `service_role` is **forbidden** on mobile.

## 7. Offline & Network
- All server calls go through a typed client. Show optimistic UI where safe.
- Handle offline via TanStack Query's offline mode + a top-level connectivity banner.
- Retries: exponential backoff, capped at 3 attempts for user-initiated requests.

## 8. Push Notifications
- Use `expo-notifications` + Expo push service (or FCM/APNs via EAS).
- Store device tokens in Supabase with RLS scoped to the owning user.
- Every notification type has a typed payload schema and a corresponding deep link.

## 9. Performance
- Lists use `FlatList` or `FlashList` with `keyExtractor` and item memoization.
- Avoid re-renders: derive state with `useMemo`, event handlers with `useCallback` only where profiling shows a win.
- Image assets are sized appropriately — no 4K source images rendered at 100×100.

## 10. Builds & Releases
- **EAS Build** for binaries; profiles defined in `eas.json`.
- **EAS Update** for OTA updates; pin the runtime version and test before publishing to production channels.
- App version bumps follow semver. Build numbers auto-increment via EAS.
- Run `/eas-build` (if defined in the project's local `.augment/commands/`) to kick off a build.

## 11. Testing
- **Vitest** or **Jest** depending on the project (see `role-tester.md`).
- Component tests via React Native Testing Library.
- Mock Expo modules with the official `jest-expo` preset or `vi.mock`.
- E2E (when set up): Maestro flows in `.maestro/` or Detox in `e2e/`.

## 12. Accessibility
- Every interactive element has `accessibilityLabel` and `accessibilityRole`.
- Support dynamic text sizing (`allowFontScaling`).
- Test with VoiceOver (iOS) and TalkBack (Android) for critical flows.

## 13. Direction (RTL / LTR)
- Enable RTL in `app.config.js`: set `android.supportsRtl: true` and call `I18nManager.allowRTL(true)` at app entry.
- **Apply direction from the active language**: on locale change to Arabic, call `I18nManager.forceRTL(true)`; on change back to English, `I18nManager.forceRTL(false)`. React Native does not reflow live — prompt the user and call `Updates.reloadAsync()` from `expo-updates`.
- Read direction at runtime via `I18nManager.isRTL`. Do **not** cache it in a module-level constant — it changes on reload.
- **NativeWind**: use the `rtl:` variant for direction-specific overrides. Prefer logical utilities (`ms-*`, `me-*`, `ps-*`, `pe-*`, `start-*`, `end-*`) — NativeWind maps them to `marginStart` / `marginEnd` / etc. in RN.
- **StyleSheet / inline styles**: use `marginStart` / `marginEnd` / `paddingStart` / `paddingEnd` / `start` / `end`. Never `marginLeft` / `marginRight` / `left` / `right` for horizontal layout.
- **Directional icons** (chevrons, back arrows, sort carets): flip via `transform: [{ scaleX: I18nManager.isRTL ? -1 : 1 }]`, or render a mirrored asset. Do **not** rotate 180°.
- **FlatList / ScrollView**: `inverted` is for chronological data (chat bubbles) only — never a substitute for RTL.
- **TextInput**:
  - Mixed / freeform content (names, addresses): let `textAlign` default to the system direction.
  - LTR-only content (phone, email, OTP, card number, numeric codes): set `textAlign="left"` and `writingDirection="ltr"`.
- **Animations & gestures**: any `translateX`, `scrollX`, or swipe offset respects `I18nManager.isRTL` — invert the sign in RTL. Back-swipe direction auto-flips in React Navigation / Expo Router; verify on device.
- **Testing**: run the app in Arabic on both iOS and Android for every PR that touches layout. Screenshot diffs required for checkout, booking, and profile flows.

