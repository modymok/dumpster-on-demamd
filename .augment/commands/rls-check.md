---
description: "Lightweight RLS coverage check across Supabase SQL — flags tables without RLS, missing policies, and unjustified using(true)."
argument-hint: "[project-name]"
---

# /rls-check

Fast-path RLS sanity check using the **security-expert** subagent. Narrower than `/security-audit` — focuses only on Postgres RLS coverage.

## Inputs
- `$1` (optional): project name (e.g. `dumpster-partner-portal`). Defaults to the project in CWD, or scans all `supabase/migrations/` folders if run from repo root.

## Steps
1. **Load rules**: `.augment/rules/main.md` + `.augment/rules/role-security.md` (§2 RLS Audit Checklist).
2. **Locate SQL**:
   - `supabase/migrations/*.sql` in the target project (or every project if scope is `repo`).
   - Optionally query the live database via the `supabase` Management API in **read-only** mode if the user has approved it.
3. **For every table `create`**, verify the migration also contains:
   - `alter table <t> enable row level security;`
   - At least one `create policy …` referencing `<t>`.
4. **For every existing policy**, flag:
   - `using (true)` without a written justification comment above it.
   - Policies that expose PII to the `authenticated` role without narrowing by `auth.uid()`.
   - `SECURITY DEFINER` functions without a corresponding `revoke execute from public`.
5. **Verify RLS tests** exist in `supabase/tests/rls/` for every new policy in the diff.

## Boundaries
- Read-only. Never modifies SQL or policies.
- Not a substitute for `/security-audit` — does not check secrets, auth, deps, or bundles.

## Output
Markdown report:
- Table summary: `✅ protected` / `❌ missing RLS` / `⚠️ weak policy`.
- Per-table findings with file/line citations.
- Handoff list: tables that need a **backend-dev** follow-up.
