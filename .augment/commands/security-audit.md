---
description: "Severity-tagged security audit — secrets scan, RLS correctness, auth flows, Snyk triage, PII handling, Stripe signature verification, client bundle scan. Read-only."
argument-hint: "[scope: repo|project-name|branch|pr-number]"
---

# /security-audit

Run a full security audit using the **security-expert** subagent.

## Inputs
- `$1` (optional): scope of the audit. Defaults to the current working project.
  - `repo` → audit the entire monorepo.
  - `dumpster-user-app` / `dumpster-partner-portal` / etc. → scope to one project.
  - A branch name or PR number → scope to the diff.

## Steps
1. **Load the rules**: `.augment/rules/main.md` + `.augment/rules/role-security.md`.
2. **Determine scope** based on `$1` or CWD.
3. **Invoke the `security-expert` subagent** to run the full audit checklist:
   - **Secrets**: scan for `service_role`, `sk_live_`, `BEGIN PRIVATE KEY`, AWS keys in tracked files, `.env*`, and built client bundles. Verify no secrets in `NEXT_PUBLIC_*`, `VITE_*`, `EXPO_PUBLIC_*`, `PUBLIC_*`.
   - **Supabase RLS**: every table has RLS enabled, ≥ 1 policy, no unjustified `using (true)`, `SECURITY DEFINER` functions have narrow grants, tests exist in `supabase/tests/rls/`.
   - **Auth**: Supabase Auth primitives only; session storage uses `expo-secure-store` on mobile; rate-limiting enabled; RBAC stored in DB.
   - **Payments**: Stripe webhook signature verification present; idempotency on `event.id`; no card data logged.
   - **PII**: logs redact phone/email/address/tokens; data exports use short-lived signed URLs; soft-delete pattern in place.
   - **Dependencies**: run `snyk test` (or read recent CI results); vet any new deps.
   - **Input validation**: all external inputs Zod-validated; no SQL string interpolation; file uploads server-side validated.
   - **Bundle check**: grep built artifacts under `dist/` for leaked secret patterns.
4. **Tag each finding**: `critical:` / `high:` / `medium:` / `low:` / `info:`.
5. **Produce the audit report** with file paths + line numbers + remediation steps.
6. **Route remediations** to the right subagent: frontend-dev / backend-dev / mobile-dev / tester.

## Boundaries
- **No code edits**. Scan-only `launch-process` usage allowed for `snyk`, `gitleaks`, `grep`.
- If a live production secret leak is detected, mark `critical:` and recommend immediate rotation — do **not** rotate automatically.

## Output
- Markdown audit report:
  - Executive summary (counts per severity).
  - Findings grouped by category.
  - Remediation handoff table.
- Incident-level findings at the top with named rotation steps.
