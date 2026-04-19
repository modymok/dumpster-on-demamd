# Secrets Inventory — Dumpsters on Demand

**Purpose.** Canonical list of every secret the six projects + subagents + slash
commands reference *by name*. This file contains **no values**, only
identifiers, scope, and handling rules.

**Storage of record.** Augment Secrets Manager. Mirrors:
- Supabase Vault (server-side runtime)
- EAS Secrets (Expo mobile builds)
- AWS Secrets Manager (Lambda runtime)
- GitHub Actions Secrets (CI only — read-only scope)

**Rotation.** All secrets rotate on a 90-day cadence unless flagged `long-lived`
below. Rotation is tracked in the `security-expert` subagent's audit trail.

**Handling legend.**
- 🟢 **public-safe** — may appear in client bundles (still never committed)
- 🟡 **server-only** — server code, Edge Functions, Lambda, CI; never in a client bundle
- 🔴 **BLOCKED** — currently referenced in a way that violates `rules/main.md §2/§11`; must remediate before agents touch this code

---

## 1. Supabase (all 6 projects)

| Key | Scope | Handling | Notes |
|---|---|---|---|
| `SUPABASE_URL` | all | 🟢 | Project REST/Realtime endpoint |
| `SUPABASE_ANON_KEY` | all | 🟢 | Public anon JWT for RLS-gated access |
| `SUPABASE_SERVICE_ROLE_KEY` | server-side only (chatbot Lambda, Edge Functions, migrations, tests) | 🟡 | Bypasses RLS. **Never** ship to any client bundle |
| `SUPABASE_ACCESS_TOKEN` | local CLI / CI migrations | 🟡 | `supabase` CLI personal access token |
| `SUPABASE_DB_PASSWORD` | migrations / `psql` | 🟡 | Postgres superuser password |
| `VITE_SUPABASE_URL` | partner-portal, driver-pwa | 🟢 | Client-side alias of `SUPABASE_URL` |
| `VITE_SUPABASE_ANON_KEY` | partner-portal, driver-pwa | 🟢 | Client-side alias of `SUPABASE_ANON_KEY` |
| `EXPO_PUBLIC_SUPABASE_URL` | user-app, driver-app | 🟢 | Expo client-side alias |
| `EXPO_PUBLIC_SUPABASE_ANON_KEY` | user-app, driver-app | 🟢 | Expo client-side alias |
| `VITE_SUPABASE_SERVICE_ROLE_KEY` | **partner-portal (currently)** | 🔴 | **BLOCKED** — service_role must never be prefixed `VITE_*`. See §Remediation |

## 2. Payments — Moyasar (user-app)

| Key | Scope | Handling |
|---|---|---|
| `EXPO_PUBLIC_MOYASAR_PUBLISHABLE_KEY` | user-app | 🟢 |
| `MOYASAR_SECRET_KEY` | server-side (Edge Function for refunds/capture) | 🟡 |
| `MOYASAR_WEBHOOK_SECRET` | server-side (Edge Function webhook receiver) | 🟡 |

> **Stripe is not currently wired in.** Root rules reference it for future work. When introduced, add: `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET`.

## 3. Meta / WhatsApp Cloud API (whatsapp-chatbot)

| Key | Scope | Handling |
|---|---|---|
| `WHATSAPP_ACCESS_TOKEN` | chatbot Lambda | 🟡 |
| `WHATSAPP_PHONE_NUMBER_ID` | chatbot Lambda | 🟡 |
| `WEBHOOK_VERIFY_TOKEN` | chatbot Lambda (Meta webhook handshake) | 🟡 |
| `META_APP_SECRET` | chatbot Lambda (webhook signature verification) | 🟡 |

## 4. OpenAI (user-app support bot, driver-app, whatsapp-chatbot)

| Key | Scope | Handling |
|---|---|---|
| `OPENAI_API_KEY` | server-side (Edge Functions + Lambda) | 🟡 |
| `OPENAI_DEFAULT_ASSISTANT_ID` | chatbot | 🟡 |
| `OPENAI_DRIVER_ASSISTANT_ID` | chatbot | 🟡 |
| `OPENAI_PARTNER_ASSISTANT_ID` | chatbot | 🟡 |
| `OPENAI_ASSISTANT_ID` | user-app server | 🟡 |
| `OPENAI_API_URL` | user-app server (override) | 🟡 |

## 5. AWS (chatbot Lambda + partner-portal uploads)

| Key | Scope | Handling |
|---|---|---|
| `AWS_REGION` | Lambda runtime, CI | 🟢 (non-secret) |
| `AWS_ACCESS_KEY_ID` | CI deploy role only | 🟡 `long-lived` |
| `AWS_SECRET_ACCESS_KEY` | CI deploy role only | 🟡 `long-lived` |
| `AWS_LAMBDA_ROLE_ARN` | CI | 🟢 (non-secret identifier) |
| `AWS_S3_BUCKET` | partner-portal uploads | 🟢 (non-secret identifier) |
| `AWS_S3_ENDPOINT` | partner-portal uploads | 🟢 |
| `VITE_AWS_ACCESS_KEY_ID` | **partner-portal (currently)** | 🔴 **BLOCKED** |
| `VITE_AWS_SECRET_ACCESS_KEY` | **partner-portal (currently)** | 🔴 **BLOCKED** |
| `VITE_AWS_S3_BUCKET` | partner-portal | 🟢 |
| `VITE_AWS_S3_ENDPOINT` | partner-portal | 🟢 |
| `VITE_AWS_REGION` | partner-portal | 🟢 |
| `VITE_AWS_S3_USE_PATH_STYLE` | partner-portal | 🟢 (non-secret flag) |

## 6. OTP / SMS — Authentica (chatbot)

| Key | Scope | Handling |
|---|---|---|
| `AUTHENTICA_API_KEY` | chatbot Lambda | 🟡 |
| `AUTHENTICA_SENDER_NAME` | chatbot Lambda | 🟢 (non-secret) |

## 7. Google Maps (all location features)

| Key | Scope | Handling |
|---|---|---|
| `GOOGLE_MAPS_API_KEY` | server-side geocoding | 🟡 (restrict by IP / referrer) |
| `EXPO_PUBLIC_GOOGLE_MAPS_API_KEY` | driver-app, user-app | 🟢 (restrict by bundle ID) |
| `VITE_GOOGLE_MAPS_API_KEY` | partner-portal, driver-pwa | 🟢 (restrict by HTTP referrer) |

## 8. reCAPTCHA (driver-app, driver-pwa)

| Key | Scope | Handling |
|---|---|---|
| `VITE_RECAPTCHA_SITE_KEY` | client | 🟢 |
| `RECAPTCHA_SECRET_KEY` | server-side Edge Function verifier | 🟡 |

## 9. EAS / Expo (user-app, driver-app)

| Key | Scope | Handling |
|---|---|---|
| `EXPO_TOKEN` | CI only | 🟡 `long-lived` |
| `EXPO_PUBLIC_EAS_PROJECT_ID` | driver-app, user-app | 🟢 (non-secret identifier) |
| `EXPO_PUBLIC_API_URL` | user-app | 🟢 |

## 10. Driver tuning knobs (driver-app + driver-pwa)

Non-secret configuration; documented for completeness so the agents know what
to configure in each environment rather than hardcode.

`VITE_ASSIGNMENT_EXPIRY_MINUTES`, `VITE_AUTO_ACCEPT_DELAY_SECONDS`,
`VITE_CODE_CHARS`, `VITE_CODE_LENGTH`, `EXPO_PUBLIC_ASSIGNMENT_EXPIRY_MINUTES`,
`EXPO_PUBLIC_AUTO_ACCEPT_DELAY_SECONDS`, `EXPO_PUBLIC_LOCATION_DISTANCE_FILTER`,
`EXPO_PUBLIC_LOCATION_UPDATE_INTERVAL`  — all 🟢.

## 11. Test fixtures (CI + local only — never prod)

`TEST_EMAIL`, `TEST_PASSWORD`, `TEST_PHONE`, `TEST_OTP` — 🟡, short-lived test
accounts only; must not map to any real user.

## 12. Security tooling (CI)

| Key | Scope | Handling |
|---|---|---|
| `SNYK_TOKEN` | CI | 🟡 |
| `GITGUARDIAN_API_KEY` | CI | 🟡 |
| `GITHUB_TOKEN` | CI (auto-provided) | 🟡 |

## 13. Observability (to be wired — not yet in code)

`SENTRY_DSN` (per project), `POSTHOG_API_KEY`, `POSTHOG_HOST`. Add when
instrumentation lands.

---

## Remediation queue (🔴 items)

1. **partner-portal — remove `VITE_SUPABASE_SERVICE_ROLE_KEY` from client bundle.** Move every call site that uses it into a Supabase Edge Function invoked from the client with the user's JWT. Owner: `backend-dev`. Blocker for `/security-audit` pass.
2. **partner-portal — remove `VITE_AWS_ACCESS_KEY_ID` / `VITE_AWS_SECRET_ACCESS_KEY` from client bundle.** Replace direct S3 client with presigned-URL flow generated server-side, or Supabase Storage. Owner: `backend-dev` + `security-expert`.
3. **Rotate all three leaked keys** (Supabase `service_role`, AWS access key pair) *immediately after* removal — assume the current values are compromised because they shipped to browsers.

These three items gate Step 7 (turning the subagents loose on real work).

## Agent usage contract

- Agents reference secrets **by key name only** — never paste values into code, comments, tests, or PR descriptions.
- Any new secret added to this doc requires a matching entry in Augment Secrets Manager *before* the code that consumes it is merged.
- Any PR that introduces a new `process.env.*`, `import.meta.env.*`, `EXPO_PUBLIC_*`, `VITE_*`, or `NEXT_PUBLIC_*` reference without updating this file fails `/pr-review`.
