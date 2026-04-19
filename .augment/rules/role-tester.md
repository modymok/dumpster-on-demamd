---
type: "manual"
description: "Rules for writing and maintaining tests across the monorepo. Loaded by the tester subagent."
---

# Role Rules — Tester

Covers unit, integration, and end-to-end tests across all 6 projects.

## 1. Framework Matrix
| Project | Unit/Integration | E2E |
|---|---|---|
| `dumpster-user-app` | Vitest + React Native Testing Library | Maestro or Detox (when set up) |
| `dumpster-partner-portal` | Vitest + React Testing Library | Playwright (when set up) |
| `dumpster-driver-app` | Jest (legacy) | — |
| `dumpster-driver-pwa` | Vitest + React Testing Library | Playwright (when set up) |
| `dumpster-admin-portal` | Vitest + React Testing Library | Playwright (when set up) |
| `dumpster-whatsapp-chatbot` | Node test runner + Vitest for modules | — |
| `v0-nefaya` | Vitest | — |

Do not introduce a second framework in the same project.

## 2. What to Test
- **Must test**: business logic, RLS policies, form validation, currency/number formatting, i18n key presence, permission gates, error paths.
- **Should test**: component rendering for critical flows (checkout, booking, auth).
- **Skip**: trivial getters, third-party wrappers that are covered upstream, pure styling.

## 3. Structure
- Co-locate: `Component.tsx` next to `Component.test.tsx`.
- Shared test utilities in `tests/utils/` or `src/test-utils/`.
- Fixtures in `tests/fixtures/` — treat them as data, not executable code.
- AAA pattern (Arrange / Act / Assert) with blank lines between sections.

## 4. Naming
- Describe the scenario: `it('rejects checkout when cart is empty', ...)`.
- Don't repeat the function name; describe the behavior.
- Failing assertion messages should tell a human what broke.

## 5. Mocks & Fixtures
- Mock Supabase via `vi.mock('@/lib/supabase', ...)` with a **typed** mock returning the same shape as the real client.
- Stripe: use Stripe's test keys and webhook event fixtures; never live keys.
- Time: freeze with `vi.useFakeTimers()` for any date-sensitive test.
- No real network calls, no real database, no real file system side effects in unit tests.

## 6. Coverage
- **80% line coverage on new/changed code** is the gate.
- Do not lower thresholds to make a red build green — fix the tests.
- `.coverage` / `coverage/` output is gitignored but uploaded as a CI artifact.

## 7. RLS Testing (Backend)
- Every new policy ships with a SQL or pg_tap test in `supabase/tests/rls/` covering:
  - An allowed read/write for the intended role.
  - A denied read/write for a different user/role.
- Tests run against a local Supabase instance in CI.

## 8. Accessibility Testing
- Use `@testing-library/jest-dom` matchers (`toBeVisible`, `toHaveAccessibleName`).
- Add `jest-axe` or `vitest-axe` checks on at least one test per page.

## 9. Flaky Tests
- If a test flakes, **quarantine and investigate** — never silently retry in CI.
- Log quarantined tests in the PR description; open a follow-up ticket.

## 10. Test Data
- Prefer factory functions (`makeUser({ role: 'partner' })`) over ad-hoc literals.
- Never commit real user data, real phone numbers, or real emails in fixtures.
