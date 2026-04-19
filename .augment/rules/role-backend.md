---
type: "manual"
description: "Backend rules for Supabase (Postgres, RLS, Edge Functions, Auth, Storage) and AWS Lambda chatbot. Loaded by the backend-dev subagent."
---

# Role Rules — Backend

Applies to: anything under `supabase/` in any project, Edge Functions,
SQL migrations, RLS policies, and the WhatsApp chatbot Lambda in
`dumpster-whatsapp-chatbot/`.

## 1. Database & SQL
- Postgres is the source of truth. Keep business logic at the DB layer where it makes RLS enforcement easier (policies, `SECURITY DEFINER` functions).
- Every migration is **idempotent where possible** and reversible. Always include a comment block at the top explaining intent.
- Use `uuid` primary keys via `gen_random_uuid()`, never `serial` for new tables.
- Timestamps: `created_at timestamptz not null default now()`, `updated_at timestamptz` with a trigger.
- Soft deletes via `deleted_at timestamptz null`, not a boolean.

## 2. Row-Level Security (RLS)
- **RLS is enabled on every table** — `alter table <t> enable row level security;`.
- At least one policy per table. Default stance: **deny**.
- Service-role access is explicit: `using (auth.role() = 'service_role')` only where needed.
- Policies reference `auth.uid()` or helper functions; avoid raw subqueries over `auth.users`.
- Add a test in `supabase/tests/rls/` for every new policy (using pg_tap or a SQL smoke test).

## 3. Edge Functions
- One function per concern. Functions live in `supabase/functions/<name>/index.ts`.
- Deno TypeScript, ESM imports via `https://esm.sh/` or `jsr:`.
- Input validation with **Zod** at the top of the handler.
- Return `{ data, error }` shape matching the Supabase client convention.
- Secrets via `Deno.env.get('NAME')`. Document required env vars in a comment block at the top.
- Always set CORS headers; prefer a shared `_shared/cors.ts` utility if the project has one.

## 4. Auth
- Use Supabase Auth primitives. Do not roll custom JWT logic.
- Phone OTP, email magic-link, and OAuth flows must handle rate-limiting; surface friendly errors to the client.
- RBAC lives in a `user_roles` table with RLS. Never trust a role claim from the client.

## 5. Stripe & Payments
- Webhook signature verification is non-optional — reject unsigned payloads.
- Idempotency: every mutating webhook handler checks for a stored `event.id` before acting.
- Never log full card data or PII; log only the `event.id` and `event.type`.

## 6. AWS Lambda (WhatsApp Chatbot)
- Handler in `index.js` stays a thin entrypoint. Business logic in `src/` modules with unit tests.
- Secrets via **AWS Secrets Manager** or Lambda environment variables — never checked into git.
- Cold-start sensitive: keep dependencies lean, avoid heavy top-level imports.
- All WhatsApp webhook events get signature-verified before processing.

## 7. Testing
- Unit-test business logic modules with **Vitest** or Node's built-in test runner.
- Integration tests for Edge Functions run against a **local Supabase** (`supabase start`) — never against production.
- RLS tests live alongside policies and run in CI on every PR that touches SQL.

## 8. Observability
- Log structured JSON: `{ level, msg, requestId, userId, ... }`. No bare `console.log` in production code.
- Errors include enough context to debug without leaking PII.
- For Edge Functions, include the function name and a request ID in every log line.

## 9. Migrations Workflow
- Generate with `supabase migration new <name>`.
- Review SQL locally with `supabase db diff` before committing.
- Run `/db-migrate` (slash command) to produce the migration + RLS policy + RLS test in one pass.
