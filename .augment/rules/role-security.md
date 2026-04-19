---
type: "manual"
description: "Security rules for the security-expert subagent. Covers secrets, dependency scanning, RLS, auth flows, and incident response."
---

# Role Rules — Security Expert

Loaded when `/security-audit` is invoked or the `security-expert` subagent is active.

## 1. Secrets Management
- **All secrets** for local dev and CI come from **Augment Secrets Manager**. Reference by name only.
- Runtime secrets:
  - Supabase Edge Functions → **Supabase Vault** or Edge Function secrets.
  - AWS Lambda → **AWS Secrets Manager** or Lambda env vars.
  - Expo builds → **EAS secrets** (`eas secret:create`).
- **Never** allow a secret to land in:
  - Git history (`pre-commit` secret scan required).
  - Any `.md`, `.json`, `.yaml`, `.env*` committed file.
  - Log output (redact before logging).
  - Client bundles — scan built artifacts for `service_role`, `sk_live_`, `AWS_SECRET`.

## 2. Supabase RLS Audit Checklist
Run on every PR that touches SQL:
- [ ] `alter table … enable row level security;` present for new tables.
- [ ] At least one `create policy …` per table.
- [ ] No `using (true)` without a written justification.
- [ ] No policy that exposes PII to an `authenticated` role without narrowing by `auth.uid()`.
- [ ] `security definer` functions are `revoke execute from public` then granted narrowly.
- [ ] Tests exist in `supabase/tests/rls/`.

## 3. Auth Flows
- Supabase Auth only — no custom JWT signing or verification on the client.
- Session storage: secure storage on mobile (`expo-secure-store`), httpOnly cookies on web where feasible.
- Password/OTP rate-limiting: verify Supabase settings; never disable in production.
- MFA: flag any change that weakens MFA requirements.
- RBAC: roles stored in DB + RLS; never trust a role claim from the client.

## 4. Dependency Hygiene
- **Snyk** runs on every PR. Block merge on new high/critical findings.
- Monthly review of advisories in GitHub Dependabot / Snyk dashboard.
- When adding a dependency: check last publish date, maintainers, weekly downloads, open critical issues.
- Prefer packages already used in the repo over adding new ones.

## 5. Input Validation & Injection
- All external input (HTTP body, query, webhook payload) validated with **Zod** before use.
- Never build SQL with string concatenation — use Supabase client methods or parameterized `rpc()`.
- Sanitize any content rendered as HTML; prefer rendering as text.
- File uploads: validate MIME, size, and extension server-side. Store outside public buckets unless intentional.

## 6. Payments (Stripe)
- Webhook signature verification is mandatory.
- Idempotency keys on mutating calls.
- Never log card data; PCI scope stays inside Stripe.
- Refund/void flows require `/security-audit` and a human approval note.

## 7. PII & Data Protection
- PII tables (`users`, `drivers`, `orders` with addresses) require stricter RLS and an audit trail.
- Phone numbers, emails, and addresses are redacted in logs to last 4 chars / hash.
- Data exports go through a service role endpoint with a signed URL that expires ≤ 15 min.
- Deletion requests follow the "soft delete + scheduled hard delete" pattern.

## 8. Client-Side Bundle Checks
Run before release:
- Source maps **not** shipped to production (or uploaded privately to Sentry/Datadog).
- No `service_role` or `sk_live_` strings in built output.
- No debug flags enabled (`DEV=true`, `LOG_LEVEL=debug`).

## 9. Incident Response
If a secret leaks, a RLS hole is found in prod, or a high-severity CVE lands:
1. Rotate the compromised secret immediately (Stripe, Supabase, AWS).
2. Patch the root cause and deploy.
3. Write a short post-incident note (what, impact, fix, prevention) — not a root cause analysis novel.
4. Open a tracking issue; do not close until prevention is in place.

## 10. Tools the Agent Must Know About
- **Snyk** (`snyk test`, `snyk monitor`) — dep + code scan.
- **Gitleaks** or `git secrets --scan` — pre-commit secret scan.
- **Supabase RLS tests** (pg_tap).
- **OWASP ZAP** — optional, for portal scanning.
