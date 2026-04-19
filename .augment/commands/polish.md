---
description: "Run an Impeccable-style design audit on the current screen or component. Flags UI slop, spacing/typography issues, and RTL correctness."
argument-hint: "[file-or-component-path]"
---

# /polish

Design-fluency audit using the **frontend-dev** or **mobile-dev** subagent (whichever owns the target), backed by the Impeccable skill (installed in step 4).

## Inputs
- `$1` (optional): path to a component, screen, or route. Defaults to the file currently open in the IDE.

## Steps
1. **Identify the target**:
   - If `$1` looks like `dumpster-user-app/...` or `dumpster-driver-app/...` → invoke **mobile-dev**.
   - Otherwise (web portals) → invoke **frontend-dev**.
   - If ambiguous, **stop and ask** the user.
2. **Load rules**: `.augment/rules/main.md` + the role file for the picked subagent.
3. **Render the screen** when possible:
   - Web: use Playwright MCP to capture a screenshot at standard breakpoints (`375`, `768`, `1440`).
   - Mobile: use the iOS simulator MCP to capture the screen in both LTR and RTL.
4. **Run the Impeccable audit** on the screenshot(s) and the source. Check:
   - **Spacing scale** consistency — no magic pixel values.
   - **Type ramp** — no off-scale font sizes / weights / line heights.
   - **Color tokens** — no hardcoded hex values in JSX.
   - **Alignment** — grid and baseline adherence.
   - **Density** — balanced whitespace; no cramped clusters.
   - **Direction (RTL / LTR)** — logical utilities only, icons flipped, no `ml-*`/`mr-*` or `marginLeft`/`marginRight` on the horizontal axis.
   - **Accessibility** — contrast ratios, focus states, accessible names.
   - **Consistency** — matches the existing component library of the target app.
5. **Produce a findings report** with severity tags and a screenshot-annotated diff per finding.
6. **Optionally** (if the user approves) apply low-risk fixes via the appropriate subagent.

## Boundaries
- Fixes are **opt-in** — always show findings first, ask before editing.
- Do not refactor anything unrelated to design fidelity.

## Output
- Annotated screenshots (LTR + RTL where applicable).
- Findings list: `critical:` visual bug / `issue:` polish needed / `nit:` optional / `praise:` what's done well.
- Optional follow-up patch (user-confirmed).
