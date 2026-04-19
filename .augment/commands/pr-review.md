---
description: "Full PR review against the 10 non-negotiable merge gates. Read-only — posts inline comments and a summary verdict, never edits code."
argument-hint: "[pr-number-or-branch-name]"
---

# /pr-review

Perform a complete pull-request review using the **code-reviewer** subagent.

## Inputs
- `$1` (optional): a PR number (e.g. `123`) or a branch name. If omitted, review the currently checked-out branch against its merge base with `main`.

## Steps
1. **Identify the diff**:
   - If `$1` is a PR number, use `github-api` (`GET /repos/{owner}/{repo}/pulls/{number}`) to fetch PR metadata and the file list.
   - Otherwise, compute the diff from the working branch vs. `origin/main`.
2. **Load the rules**: read `.augment/rules/main.md` and `.augment/rules/role-reviewer.md` into context.
3. **Invoke the `code-reviewer` subagent** with the diff. It will evaluate all 10 merge gates:
   1. Lint passes
   2. Typecheck passes
   3. Tests pass + ≥ 80% coverage on changed lines
   4. Snyk: no new high/critical findings
   5. No committed secrets
   6. i18n: `en.json` + `ar.json` both updated if strings changed
   7. RLS enabled + policy + test for any new table
   8. Migration present + named correctly for any schema change
   9. Conventional Commits format on all commits and the PR title
   10. `### Agent-Explanation` section in PR description (≤ 120 words)
4. **Emit findings** with the conventional prefixes: `blocker:` / `issue:` / `nit:` / `question:` / `praise:`.
5. **Handoffs**:
   - SQL/RLS changes → tag **backend-dev** for correctness verification.
   - Auth / payments / PII changes → require a `/security-audit` first.
   - Mobile-specific code → tag **mobile-dev** for stack-specific review.
6. **Verdict**: post a summary comment with `Approve` / `Request Changes` / `Comment` and a one-line scope statement.

## Boundaries
- This command **never edits code**, runs migrations, or merges the PR.
- If a gate cannot be evaluated (e.g. CI hasn't run yet), mark it `info:` and request the author re-run CI.

## Output
- An ordered list of findings grouped by priority (security first).
- A final verdict line.
- A handoff list naming the subagent responsible for each blocker.
