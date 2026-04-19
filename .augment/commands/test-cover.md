---
description: "Raise unit/integration test coverage on a target file or module to the 80% gate."
argument-hint: "<file-or-module-path>"
---

# /test-cover

Bring a target file's coverage up to the 80% gate using the **tester** subagent.

## Inputs
- `$1` (required): a file path (e.g. `src/components/CheckoutForm.tsx`) or a module directory. If omitted, **stop and ask**.

## Steps
1. **Confirm target project**: infer from `$1`'s path prefix. If ambiguous, ask.
2. **Load rules**: `.augment/rules/main.md` + `.augment/rules/role-tester.md`.
3. **Pick the framework** for the project (see `role-tester.md` §1 Framework Matrix): Vitest / Jest / Node test runner.
4. **Run current coverage**:
   - Project-appropriate command — e.g. `npm test -- --coverage <file>` or `pnpm test --coverage <file>`.
   - Capture the current line/branch coverage for `$1`.
5. **Read `$1`** and identify uncovered branches: error paths, guard clauses, empty-state UI, edge cases in form validation, permission gates.
6. **Author new tests** alongside existing ones (co-located `<name>.test.ts(x)`):
   - One `describe` per public function/component.
   - `it` names describe the behavior, not the function name.
   - AAA pattern with blank lines.
   - Mock Supabase / Stripe / network per `role-tester.md` §5.
   - Freeze time with `vi.useFakeTimers()` for date-sensitive assertions.
7. **Re-run coverage**. If below 80% on changed lines, iterate.
8. **Report**:
   - Before / after coverage numbers.
   - New test file paths.
   - Any branches that are genuinely impossible to test (annotate with a rationale comment in the source).

## Boundaries
- **Tests only** — tester cannot edit production source. If a branch is untestable because of a bug, hand off to the responsible subagent with a minimal repro.
- Do not lower coverage thresholds or add `c8 ignore` / `istanbul ignore` to hit the gate.

## Output
- `before → after` coverage table (lines / branches / functions).
- New test file(s) with AAA-structured cases.
- Handoff items (if any) routed to **frontend-dev** / **backend-dev** / **mobile-dev**.
