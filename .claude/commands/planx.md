---
description: Write a self-contained chapter plan to docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/ for another AI to write the prose.
argument-hint: [chapter or revision to plan]
allowed-tools: Write, Read, Glob, Grep, Task, Bash
---

# /planx

Plan only — outline the writing, never write the chapter. No edits outside the plan dir; never touch `docs/NN-*.md` from here.

## Goal
$ARGUMENTS

## Steps

1. **Path.** `date +%Y`/`%m`/`%d`. `Glob docs/plans/<YYYY>/<MM>/<DD>/1*` → next `1NN` (else `101`). Slug kebab-case, ≤5 words. Dir: `docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/`.
2. **Explore.** `Task` (Explore, "very thorough"): the adjacent chapters, the README TOC row that will change, every cross-link that must be added or updated (`grep -rn "NN-"`), and the ch. 11 rules the prose must obey.
3. **Write `overview.md`** — Goal (what the chapter teaches, why it earns a slot) · Placement (chapter number, neighbours, TOC row text) · Section outline (one line per `##`) · Plan files (numbered) · Done when · Risks.
4. **One `<NN>-<aspect>.md` per section or per file touched** — `> Part of overview.md. Depends on: <NN or none>.` · What to write (bullets, the claims to make) · Sources to verify against · Links to add/update (`path` → `path`) · Done when. Never draft the prose here.
5. **`status.yml`** alongside `overview.md` — `plan`, `title`, `status` (not_started|in_progress|blocked|complete|superseded), `created_by`/`owner` from `git config user.name`, `worked_by: ""`, `percent: 0`, `current_focus: ""`, `slices:` (one row per `<NN>` file with `status`/`percent`), `evidence: []`, `notes: ""`, `last_updated`.

## Rules

- Compact English, fragments over sentences, tables for structured data — the plan obeys ch. 11 too.
- Reference-only: point at chapters and line numbers, don't paste prose.
- No checkboxes. `status.yml` is the only tracker.
- **Never plan around git worktrees.** The writer works directly in this checkout; parallel agents share it, one chapter file each. Never `isolation: worktree`.
- Never plan a renumber unless the ask demands it — it rewrites every inter-chapter link.
- Every plan that adds or retitles a chapter includes the README TOC row and cross-link sweep as an explicit slice.

## Output
```
✓ docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/overview.md + 01-….md, … + status.yml
Next: run a writer on overview.md.
```
