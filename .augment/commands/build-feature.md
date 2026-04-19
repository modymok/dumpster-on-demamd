---
description: "End-to-end feature development. Coordinates backend-dev (migrations, RLS, Edge Functions) with frontend-dev or mobile-dev (UI) inside a specific dumpster-* project."
argument-hint: "<project-name> \"<feature-description>\""
---

# /build-feature

Ship a feature from database to UI inside a single `dumpster-*` project, coordinating **backend-dev** with **frontend-dev** or **mobile-dev**.

## Inputs
- `$1` (required): the target project.
  - Allowed: `dumpster-user-app`, `dumpster-partner-portal`, `dumpster-driver-app`, `dumpster-driver-pwa`, `dumpster-admin-portal`, `dumpster-whatsapp-chatbot`, `v0-nefaya`.
- `$2` (required): feature description — prose or a link to a spec.

If either is missing, **stop and ask**:
1. Which project? (show the 7-project list)
2. What is the feature? (prose or spec link)
3. Does it need new DB tables / columns? (auto-detected from the description, confirm with user)
4. Does it need a new Edge Function or an extension to an existing one?
5. Is any PII / payment / auth change involved? (if yes, require `/security-audit` gate at the end)

## Routing Matrix
| Target project | UI subagent | Backend subagent | Notes |
|---|---|---|---|
| `dumpster-user-app`, `dumpster-driver-app` | **mobile-dev** | **backend-dev** | Expo + Supabase |
| `dumpster-partner-portal`, `dumpster-driver-pwa`, `dumpster-admin-portal`, `v0-nefaya` | **frontend-dev** | **backend-dev** | Vite/Next + Supabase |
| `dumpster-whatsapp-chatbot` | — | **backend-dev** | Lambda + Supabase, no UI |

## Steps

### Phase 1 — Plan (both subagents confer)
1. Load rules: `.augment/rules/main.md` + `role-backend.md` + (the UI role file for the target project).
2. Research existing schema, tables, Edge Functions, and similar features with `codebase-retrieval`.
3. Produce a **written plan** listing:
   - New / modified tables with columns + RLS policies.
   - New / modified Edge Functions with input / output types.
   - UI surfaces (screens, components) with their data requirements.
   - i18n key additions.
   - Tests (RLS + unit + component).
4. **Present the plan to the user and wait for approval** before writing any code.

### Phase 2 — Backend (backend-dev)
1. Use `/db-migrate` internally to scaffold the migration + RLS policy + RLS test.
2. If an Edge Function is needed:
   - Create under `<project>/supabase/functions/<name>/index.ts`.
   - Zod-validate inputs, return `{ data, error }`, structured JSON logs.
   - Document env vars at the top.
3. Add unit tests for any business logic module (`src/` next to the function).
4. If Stripe or auth is touched → request `/security-audit` before Phase 3.

### Phase 3 — Frontend / Mobile (frontend-dev OR mobile-dev)
1. Use `/build-ui` internally for each screen/component needed.
2. Wire the new Edge Function / typed Supabase queries via TanStack Query hooks.
3. Add i18n keys to both `en.json` and `ar.json`.
4. Verify RTL on web (`dir="rtl"`) or mobile (`I18nManager.forceRTL(true)`).
5. Write component tests with the project's test framework.
6. Run `/polish` on every new screen.

### Phase 4 — Gate (before PR)
- `/test-cover` on every changed file until ≥ 80%.
- `/i18n-audit` on the target project.
- `/rls-check` if SQL changed.
- `/security-audit` if Phase 1 flagged PII / payments / auth.
- `/pr-review` as the last step before requesting human review.

## Boundaries
- Only one `dumpster-*` project per invocation. For cross-project work, run the command per project.
- No scope creep — stick to the approved plan; if requirements grow, stop and ask.
- `service_role` never leaves the server side. Mobile / web clients use the anon key + RLS only.

## Output
- Phase 1: plan document posted for approval.
- Phase 2: migration path, RLS test path, Edge Function path (if any).
- Phase 3: UI file list, i18n diff, screenshots (LTR + RTL).
- Phase 4: gate results table + final PR description with `### Agent-Explanation`.
