---
description: "Scaffold and implement a frontend component, section, or full screen. Routes to frontend-dev (web) or mobile-dev (Expo) and enforces i18n EN/AR, RTL, and accessibility rules."
argument-hint: "<target-path-or-project> [\"<ui-requirement-or-figma-link>\"]"
---

# /build-ui

Implement a new UI piece end-to-end using the appropriate UI subagent. Automatically applies all i18n, RTL, and accessibility rules.

## Inputs
- `$1` (required): target path or project identifier. Formats accepted:
  - `dumpster-user-app/app/checkout/summary.tsx` — explicit file (mobile).
  - `dumpster-partner-portal/src/pages/Drivers/NewDriver.tsx` — explicit file (web).
  - `dumpster-partner-portal` — project only; will prompt for the screen path.
- `$2` (required): UI requirement — **one of**:
  - A prose description: `"checkout summary screen with cart list, total, promo code input, and Apple Pay button"`.
  - A Figma URL: `https://www.figma.com/file/...?node-id=...`.
  - A local path to a spec: `docs/specs/checkout-summary.md`.

If either argument is missing or ambiguous, **stop and ask**:
1. Which project? (list the 6 dumpster-* projects)
2. What screen/component?
3. Do you have a Figma link, a prose spec, or a written requirement?

## Routing
| Target project | Subagent | Styling |
|---|---|---|
| `dumpster-user-app`, `dumpster-driver-app` | **mobile-dev** | NativeWind + Expo Router |
| `dumpster-partner-portal`, `dumpster-driver-pwa`, `dumpster-admin-portal`, `v0-nefaya` | **frontend-dev** | Tailwind + React + Vite (Next.js in v0-nefaya) |

## Steps
1. **Confirm target and requirements** (prompt if missing).
2. **Load rules**: `.augment/rules/main.md` + the role file for the picked subagent (`role-frontend.md` or `role-mobile.md`).
3. **Fetch the design**:
   - If `$2` is a Figma URL → use the Figma MCP (`get_design_context`) to pull the design context and assets.
   - If `$2` is a local spec → read it.
   - If `$2` is prose → proceed with text only.
4. **Research the codebase** (`codebase-retrieval`): find existing components, hooks, design tokens, and patterns in the target project. Reuse over reinvent.
5. **Scaffold the component/screen**:
   - One component per file, PascalCase name, matching the project's file layout.
   - Props interface `<Component>Props`.
   - No raw JSX strings — route every user-facing string through `t('namespace.key')`.
   - Tailwind/NativeWind **logical utilities only** on the horizontal axis (`ms-*`, `me-*`, `ps-*`, `pe-*`, `start-*`, `end-*`, `text-start`, `text-end`).
   - Accessible semantics: native `<button>`, `<a>`, `<input>` (web) or `accessibilityRole`/`accessibilityLabel` (mobile).
6. **Wire data** (if applicable): TanStack Query hooks against the typed Supabase client. Never raw `useEffect + fetch`.
7. **Forms**: React Hook Form + Zod — no ad-hoc validators.
8. **i18n**: add new keys to **both** `locales/en.json` and `locales/ar.json` in the same change. Arabic values may be `TODO: translate` placeholders flagged in the PR.
9. **Verify RTL**: render under `dir="rtl"` (web) or `I18nManager.forceRTL(true)` (mobile) and visually confirm — no overlap, mirrored icons where needed.
10. **Write a test** alongside the component (Vitest + RTL, or Jest for driver-app): user-visible behavior, not implementation details.
11. **Run** `/polish` on the new screen before handing off.

## Boundaries
- This command builds **UI only**. Any new backend table/endpoint → stop and recommend `/build-feature` instead.
- Do not introduce a new styling system or state library — use what the project already uses.
- Do not refactor unrelated code.

## Output
- List of created/modified files.
- i18n key diff (`en.json` + `ar.json`).
- Screenshots (LTR + RTL) from the Playwright or iOS-simulator MCP.
- Test file path + coverage summary.
- PR-ready `### Agent-Explanation` block (≤ 120 words).
