---
description: "Scaffold a Supabase migration + RLS policy + RLS test in one pass. Runs backend-dev under the backend rules."
argument-hint: "<short-description>"
---

# /db-migrate

Create a new Supabase migration with RLS policy and test scaffolding in a single step using the **backend-dev** subagent.

## Inputs
- `$1` (required): a short description in kebab-case, e.g. `add-driver-availability-table` or `add-status-to-orders`.
  - If omitted, **stop and ask** the user what the migration should do.

## Steps
1. **Confirm target project**: infer from CWD. If the user is at the repo root, ask which project the migration belongs to (`dumpster-user-app` / `dumpster-partner-portal` / etc.).
2. **Load rules**: `.augment/rules/main.md` + `.augment/rules/role-backend.md`.
3. **Ask the user** (if not already specified) for:
   - The intent of the migration (what tables/columns, what business purpose).
   - The access pattern (who reads, who writes, what role).
4. **Generate the migration filename**: `<timestamp>_$1.sql` where timestamp is `YYYYMMDDHHMMSS` in UTC. Place it under `<project>/supabase/migrations/`.
5. **Write the migration SQL**:
   - Header comment block: intent, author, date, reversal notes.
   - DDL: `create table`, `alter table`, etc. — use `uuid` PKs via `gen_random_uuid()`, `timestamptz` for times, soft-delete via `deleted_at`.
   - `alter table <t> enable row level security;` for every new table.
   - `create policy …` — at least one per new table, narrowed by `auth.uid()` or a role function.
   - Trigger for `updated_at` if applicable.
6. **Write the RLS test** in `<project>/supabase/tests/rls/<timestamp>_$1.sql`:
   - Verify an allowed read/write for the intended role.
   - Verify a denied read/write for a different user/role.
7. **Print next steps** for the user:
   - `supabase db diff` to review.
   - `supabase migration up` locally to apply.
   - Commit with `feat(db): <description>` in Conventional Commits format.

## Boundaries
- Do **not** run the migration against production or a remote project.
- Do **not** modify unrelated migrations.
- If RLS narrowing is ambiguous, **stop and ask** — never default to `using (true)`.

## Handoff
- Schema touches PII or auth tables → request `/security-audit` before merge.
- Frontend/mobile needs a typed client for the new table → brief **frontend-dev** / **mobile-dev** with the contract.

## Output
- Path to the new migration file.
- Path to the new RLS test file.
- A summary block describing the change, policies, and required env vars (if any).
