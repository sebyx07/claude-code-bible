# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A prose book — "The Claude Code Bible" — distributed as Markdown. No source code, no build, no tests, no CI. Edits ship by committing `.md` files.

- `README.md` — landing page and chapter index (TOC table, quick wins, license).
- `docs/NN-<slug>.md` — numbered chapters, 01 through 11. Read top-to-bottom as a book.
- `LICENSE` — MIT.
- `.gitignore` ignores `.claude/`, so local agent config here is never committed.

Default branch: `main`. Remote: `origin`. No tags, no release process — readers consume from GitHub directly.

## Chapter layout

Chapters are numbered so the TOC order in `README.md` matches filename order. Adding a new chapter means:

1. Create `docs/NN-<slug>.md` with the next number.
2. Add a row to the TOC table in `README.md` (keep columns aligned: `#`, Chapter link, "What it covers").
3. Cross-links between chapters use relative paths like `[ch. 11](docs/11-compressed-config.md)` from the README, or `[ch. 11](11-compressed-config.md)` from inside `docs/`.

Renumbering = renaming files and rewriting every inter-chapter link. Avoid unless necessary.

## Writing conventions (this repo eats its own dog food)

The rules in `docs/11-compressed-config.md` govern this repo's own prose. When editing any `.md` here, follow them:

- Lead with the rule, not the reason. Fragments over sentences where natural.
- Tables for any ≥3-row structured data (see README TOC, ch. 11 cost table).
- Code blocks, shell commands, file paths, URLs, and version strings stay verbatim — compress the prose around them, not these.
- No meta-framing ("This section covers…"), no rhetoric ("this is critical"), no trailing summary paragraphs.
- One rule per line in rule lists; imperative voice.
- Size budgets per file type (from ch. 11): `CLAUDE.md` <600 lines · `SKILL.md` <80 · agent <50 · command <30. Chapters themselves aren't capped but aim for high info density — current chapters run ~100–400 lines.

When in doubt about tone, open an adjacent chapter and match it. Chapters 01, 03, and 11 are representative.

## Common edits

- **Fix a typo / tighten prose in a chapter**: edit the single `docs/NN-*.md` file. No TOC change needed unless the chapter title changes.
- **Chapter title change**: update the heading in the chapter file *and* the matching row in the README TOC table *and* any cross-references from other chapters (`grep -rn "NN-old-slug"`).
- **New chapter**: as above — new file, TOC row, cross-links.
- **Rules update (e.g. new compression rule)**: edit `docs/11-compressed-config.md`. If it's load-bearing for other chapters, consider whether any existing example needs rewriting.

## Validation

There is no linter or test. Before committing:

- Preview the Markdown (any viewer) to confirm tables and code fences render.
- Click every link you added or touched — relative paths break silently.
- Re-check against the self-check list at the bottom of `docs/11-compressed-config.md`.

## Git workflow

Normal flow: branch, edit, commit, PR to `main`. Recent history (`git log --oneline`) shows the commit-message style — short imperative subject, optionally one blank line and a body. Match it.
