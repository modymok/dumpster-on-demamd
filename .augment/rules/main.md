---
type: "always_apply"
description: "Global workspace guardrails for the Dumpsters on Demand monorepo. Applies to every agent, every project, every file."
---

# Dumpsters on Demand — Global Workspace Rules

These rules are **always applied** across all 6 projects in this monorepo:
`dumpster-user-app`, `dumpster-partner-portal`, `dumpster-driver-app`,
`dumpster-driver-pwa`, `dumpster-admin-portal`, `dumpster-whatsapp-chatbot`
(plus `v0-nefaya`).

Role-specific rules live in `.augment/rules/role-*.md` and are loaded by the
matching subagent in `.augment/agents/`.

---

## 1. Operating Principles
1. **Research-first**: use `codebase-retrieval` or `view` before generating new code. Confirm symbols, signatures, and call sites exist.
2. **Single-task focus**: finish one atomic unit of work before starting the next.
3. **Ask on ambiguity**: if more than one reasonable path exists, ask — do not guess.
4. **Respect scope**: do exactly what was requested. No unsolicited refactors, no new docs/README files unless asked.
5. **Downstream audit**: after every edit, search for callers, interface implementers, tests, and config files that also need updating.

## 2. Security & Secrets (non-negotiable)
- **Never** hardcode secrets in source, `.md`, `.env` committed, or fixtures.
- All secrets come from **Augment Secrets Manager** or the runtime's secret store (Supabase Vault, AWS Secrets Manager, EAS env). Reference them by name only.
- The Supabase **`service_role`** key is server-side only. Never ship it to any client bundle (user-app, portals, driver-app, pwa, chatbot-client-code).
- Any change touching auth, payments, PII, or RLS triggers a mandatory `/security-audit` before merge.
- Log redaction: strip tokens, emails, phone numbers, and payment data from logs and error messages.

## 3. Database & Supabase
- **RLS is mandatory** on every table. New tables require a migration that enables RLS and defines at least one policy.
- SQL migrations live in the owning project's `supabase/migrations/` folder and are named `YYYYMMDDHHMMSS_short_description.sql`.
- Prefer Supabase features (Auth, Storage, Realtime, Edge Functions) before rolling custom code.
- All schema changes are reviewed by the **backend-dev** subagent; anything that touches PII also by **security-expert**.

## 4. Internationalization & RTL
- Every user-facing string goes through `i18next`. No raw strings in JSX/TSX.
- Keys are flat, kebab-case namespaces (e.g. `checkout.confirm-button`).
- Both **English** and **Arabic** must be updated in the same commit.
- Arabic must be visually verified in **RTL** layout. Use `dir="rtl"` + logical Tailwind utilities (`ms-*`, `me-*`) instead of `ml-*`/`mr-*`.

## 5. Accessibility
- WCAG **AA** minimum on all portals and the user app.
- All interactive elements need a keyboard path and an accessible name.
- Color contrast ≥ 4.5:1 for body text, 3:1 for large/bold text.

## 6. Code Style
- **TypeScript**: `strict: true`, no `any` without an inline justification comment.
- **Naming**: functions = verbs, components/classes = PascalCase nouns, booleans = `is|has|should|can` prefix.
- **File size**: prefer files under 400 LOC; split when a file owns more than one concept.
- **Comments**: explain *why*, not *what*. Match surrounding density — do not over-comment.

## 7. Commits, Branches, PRs
- **Conventional Commits**: `feat:`, `fix:`, `chore:`, `docs:`, `refactor:`, `test:`, `perf:`, `build:`, `ci:`.
- Add `BREAKING CHANGE:` footer when applicable.
- Branch naming: `<type>/<project>/<short-desc>` — e.g. `feat/user-app/checkout-apple-pay`.
- **No direct pushes to `main`.** All work lands via PR.
- PR must pass: lint, typecheck, unit tests, Snyk, and `/pr-review` from the **code-reviewer** subagent.
- PR description includes an `### Agent-Explanation` section (≤ 120 words) covering *why* + *how*.

## 8. Testing
- Every new module ships with a test in the project's test directory (`__tests__/`, `tests/`, or co-located `.test.ts`).
- **Coverage target: 80%** on new/changed lines. Do not lower thresholds to make a build pass.
- Test framework per project: Vitest (portals, driver-pwa, user-app), Jest (driver-app legacy), Node test runner (whatsapp-chatbot).
- No network, no real Supabase, no real Stripe in unit tests — use mocks/fixtures.

## 9. Monorepo Conventions
- Package managers are **per-project** and must not be mixed:
  - `dumpster-user-app` → **yarn**
  - `dumpster-partner-portal` → **pnpm** (workspace)
  - `dumpster-driver-app`, `dumpster-driver-pwa`, `dumpster-whatsapp-chatbot` → **npm**
  - `v0-nefaya` → **pnpm**
- Never introduce a new lockfile format in an existing project.
- Shared types/contracts belong in a published internal package, not copy-pasted.

## 10. External Services
- **Supabase** is the primary backend. Check existing Edge Functions before creating new ones.
- **Stripe**: use Stripe's published SDKs; never parse webhook payloads manually without signature verification.
- **AWS Lambda** (chatbot): handler code is small and testable; heavy logic lives in `src/` modules, not `index.js`.
- **EAS** (Expo): mobile builds go through EAS profiles in `eas.json`; credentials via EAS secrets.

## 11. Forbidden
- Deprecated or unmaintained packages (flagged by Snyk high/critical).
- Synchronous network calls on the UI thread.
- `console.log` in committed code outside of explicit debug utilities.
- Copying the `service_role` key into any `NEXT_PUBLIC_*`, `VITE_*`, `EXPO_PUBLIC_*`, or `PUBLIC_*` variable.
- Committing files containing `BEGIN PRIVATE KEY`, `sk_live_`, `service_role`, or AWS access keys.

## 12. When a Task Fails
- If you go in circles or hit a dead end, stop and present **2–3 alternatives** to the user, then wait.
