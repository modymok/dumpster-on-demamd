---
type: "manual"
description: "Frontend web rules for React + Vite portals (partner, municipal, system admin, driver-pwa, v0-nefaya). Loaded by the frontend-dev subagent."
---

# Role Rules — Frontend (Web)

Applies when working in: `dumpster-partner-portal/`, `dumpster-driver-pwa/`,
`dumpster-admin-portal/`, `v0-nefaya/`.

## 1. Stack Contract
- **React 18** (functional components + hooks only, no class components).
- **Vite** for bundling. Next.js only inside `v0-nefaya/`.
- **React Router v6/v7** for routing in Vite apps.
- **TailwindCSS v3** for styling. No inline `style={{}}` except for dynamic values that can't be expressed as classes.
- **Zustand** for client state. **TanStack Query (React Query)** for server state.
- **React Hook Form** + **Zod** for forms and validation. Never write an ad-hoc validator.

## 2. Components
- One component per file. File name = component name in PascalCase.
- Props interface named `<Component>Props`, exported when consumed outside.
- Prefer composition over prop explosion — if a component takes more than ~8 props, split it.
- Accessibility: every interactive element must render a native `<button>`, `<a>`, or `<input>` (or use a headless lib). No click handlers on `<div>`.

## 3. Styling
- Order Tailwind classes: layout → box → typography → color → state → responsive.
- Use design tokens (CSS variables) for colors and spacing; don't hardcode hex values in JSX.
- Direction (RTL / LTR): see §9 for the full policy. Never use `ml-*` / `mr-*` / `pl-*` / `pr-*` / `left-*` / `right-*` / `text-left` / `text-right` on user-facing layouts.
- Dark mode is a future concern — do not add `dark:` variants unless the project already uses them.

## 4. Data Fetching
- All Supabase reads/writes go through the typed client in `src/lib/supabase.ts`.
- Wrap queries in React Query hooks: `useXxxQuery`, `useXxxMutation`. No raw `useEffect(() => fetch(...))`.
- Invalidate relevant query keys after mutations; never call `window.location.reload()`.
- Realtime subscriptions are set up in a single place per feature and cleaned up on unmount.

## 5. i18n
- Every string in JSX goes through `t('namespace.key')`.
- Update both `locales/en.json` and `locales/ar.json` in the same change.
- Pluralization uses i18next's `count` interpolation, not string concatenation.
- Numbers, dates, and currency formatted via `Intl.*` with the active locale.

## 6. Performance
- Memoize lists > 50 items with `React.memo` on row components.
- Lazy-load routes with `React.lazy` + `Suspense`.
- Images use `loading="lazy"` and explicit `width`/`height` to avoid CLS.
- Bundle splitting: keep per-route chunk size under ~250 KB gzipped; flag larger.

## 7. Testing
- **Vitest** + **React Testing Library**.
- Test user-visible behavior (`findByRole`), not implementation details.
- One test file per component, co-located as `Component.test.tsx`.
- Mock Supabase with `vi.mock('@/lib/supabase', ...)`.

## 8. Design Fluency
- Run `/polish` (Impeccable skill) on any new screen or major UI change before opening the PR.
- Respect the portal's existing visual language: spacing scale, type ramp, and component set.

## 9. Direction (RTL / LTR)
- Set `<html dir>` at the root from the active locale: `ar` → `rtl`, `en` → `ltr`. Toggle at the document root, never per-component.
- Use **logical Tailwind utilities** exclusively on the horizontal axis:
  - Spacing: `ms-*` / `me-*` / `ps-*` / `pe-*` — never `ml-*` / `mr-*` / `pl-*` / `pr-*`.
  - Position: `start-*` / `end-*` — never `left-*` / `right-*`.
  - Text: `text-start` / `text-end` — never `text-left` / `text-right`.
  - Borders & radius: `border-s`, `border-e`, `rounded-s`, `rounded-e`.
- **Directional icons** (chevrons, arrows, back/next, sort carets) must flip in RTL. Apply `rtl:-scale-x-100` or swap the icon based on `i18n.dir()`. Do **not** rotate 180°.
- **Flexbox**: prefer `flex-row` — it auto-reverses under `dir="rtl"`. Use `flex-row-reverse` only when the logical order must be reversed regardless of locale.
- **LTR-only inputs** (phone, email, URL, credit card, numeric codes) stay LTR even on an RTL page: set `dir="ltr"` on the `<input>`.
- **Mixed-direction text** in a sentence: wrap the foreign segment in `<bdi>` or `<span dir="auto">` so the browser's bidi algorithm doesn't reorder punctuation.
- **CSS-in-JS / computed styles**: never hardcode `left` / `right` / `marginLeft` / `marginRight`. Use logical properties (`inset-inline-start`, `margin-inline-start`) or read `i18n.dir()` at runtime.
- **Testing**: every new screen is verified in both `ltr` and `rtl` before merge. Add a Playwright/RTL snapshot or a Storybook toggle that flips `dir` and asserts the layout still holds.

