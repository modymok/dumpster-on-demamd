---
description: "Audit EN/AR parity and find hardcoded strings in JSX/TSX across a project."
argument-hint: "[project-name]"
---

# /i18n-audit

Find hardcoded user-facing strings and verify `en.json` / `ar.json` parity using the **tester** subagent.

## Inputs
- `$1` (optional): project name. Defaults to CWD.

## Steps
1. **Confirm scope**: target project directory.
2. **Load rules**: `.augment/rules/main.md` + `.augment/rules/role-tester.md`.
3. **Locate i18n resources**:
   - `locales/en.json` + `locales/ar.json` (or the project's equivalent path).
4. **Parity check**:
   - Every key in `en.json` exists in `ar.json` and vice versa.
   - No empty values in either file.
   - No placeholder-count mismatch (e.g. `{{count}}` appears in both or neither).
5. **Hardcoded-string scan**:
   - Grep `.tsx` / `.jsx` / `.ts` / `.js` under `src/` and `app/` for JSX text nodes and string literals passed to user-facing components (`<Text>`, `<Button label={...}>`, `placeholder=`, `title=`, `label=`, `aria-label=`).
   - Ignore test files, fixtures, and anything under `node_modules/`, `dist/`, `build/`.
6. **Flag**:
   - Missing key pairs.
   - Keys used in code but not present in both locale files.
   - Keys defined in locale files but never referenced in code (dead keys).
   - Strings with suspected typos (optional, low confidence — tag as `info:`).
7. **Suggest fixes**:
   - Generate the missing `ar.json` stubs with `TODO: translate` markers.
   - Propose key names for hardcoded strings following the flat kebab-case convention (`namespace.key`).

## Boundaries
- This is a **read-only** audit. Do not automatically add keys or edit locale files.
- Translation quality is out of scope — flag missing AR values but don't translate them.

## Output
- Parity summary: `✅ N keys match` / `❌ M keys missing in ar.json` / `⚠️ K dead keys`.
- Hardcoded-strings table: file / line / suggested key.
- A ready-to-paste patch for missing keys (EN + AR stubs).
