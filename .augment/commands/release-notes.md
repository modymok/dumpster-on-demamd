---
description: "Generate release notes from Conventional Commits since the last tag."
argument-hint: "<version> [project-name]"
---

# /release-notes

Produce a structured release note using the **code-reviewer** subagent (read-only, good at summarization).

## Inputs
- `$1` (required): the new version tag, e.g. `v1.4.0` or `user-app@1.4.0`. If omitted, **stop and ask**.
- `$2` (optional): project name to scope the log. Defaults to repo-wide.

## Steps
1. **Resolve the diff range**:
   - Find the last matching tag: `git describe --tags --abbrev=0` (or scoped by project prefix if `$2` is given).
   - Range = `<last-tag>..HEAD`.
2. **Fetch commits**: `git log <range> --pretty='%H %s' --no-merges` (scoped to `$2` with `-- <project>/` when applicable).
3. **Parse Conventional Commits** into buckets:
   - 🚀 **Features** (`feat:`)
   - 🐛 **Fixes** (`fix:`)
   - ⚡ **Performance** (`perf:`)
   - ♻️ **Refactors** (`refactor:`)
   - 📚 **Docs** (`docs:`)
   - 🧪 **Tests** (`test:`)
   - 🔧 **Chores / Build / CI** (`chore:`, `build:`, `ci:`)
   - ⚠️ **Breaking changes** — commits with `!` or `BREAKING CHANGE:` footer.
4. **Deduplicate** and group by scope where provided (e.g. `feat(checkout):`).
5. **Write the release note** in this shape:
   ```
   # <version> — <YYYY-MM-DD>

   ## Highlights
   - 1–3 sentences on the headline changes.

   ## ⚠️ Breaking Changes
   - …

   ## 🚀 Features
   - …

   ## 🐛 Fixes
   - …

   (other buckets as needed)

   ## Contributors
   - @user1, @user2
   ```
6. **List contributors** from `git shortlog -sn <range>`.
7. **Output as a markdown file** (do not create it — print to chat for the user to paste into the GitHub release or CHANGELOG).

## Boundaries
- Read-only — never creates a tag, a release, or commits a CHANGELOG.
- If commits don't follow Conventional Commits, flag them at the bottom under `⚠️ Unstructured commits` with the SHA + subject.

## Output
Markdown release notes ready to paste into GitHub Releases or `CHANGELOG.md`.
