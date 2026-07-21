---
description: Author or revise a chapter of The Claude Code Bible — research, draft in house style, fix TOC and cross-links, commit and PR.
argument-hint: <chapter to write or revision to make>
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Task, WebFetch
---

# /feature

Author on **The Claude Code Bible** — a prose book shipped as Markdown. No code, no build, no tests. Edits ship by committing `.md` files.

## Request
$ARGUMENTS

## Flow

1. **Scope.** New chapter, retitle, prose fix, or rules update? Read the adjacent chapters first — never write about behavior you haven't confirmed.
2. **Draft — no worktrees.** **Never use git worktrees and never `isolation: worktree`.** Parallel `Task` agents share this one checkout: one chapter file per agent, one branch, no branch switching under each other, and leave other agents' files alone (another workstream is often mid-chapter).
3. **New chapter** → `docs/NN-<slug>.md` with the next number, a TOC row in `README.md` (columns aligned: `#`, Chapter link, "What it covers"), and cross-links (`[ch. 11](11-compressed-config.md)` from inside `docs/`, `docs/11-…` from the README). Avoid renumbering — it rewrites every inter-chapter link.
4. **Retitle** → heading + README TOC row + every reference (`grep -rn "NN-old-slug"`).
5. **Style.** Rules from `docs/11-compressed-config.md` govern this repo's own prose: lead with the rule not the reason; fragments over sentences; tables for any ≥3-row structure; no meta-framing, no rhetoric, no trailing summary. Code, paths, URLs, version strings stay verbatim. Match tone with ch. 01, 03, or 11.
6. **Validate.** No linter here: preview the Markdown (tables and fences render), click every link you touched, re-check the self-check list at the end of ch. 11.
7. **Ship.** Branch → commit (short imperative subject, matching `git log --oneline`) → PR to `main`.

## Output

```
Chapter:  docs/NN-<slug>.md (new|revised)
TOC:      updated | n/a    Cross-links: <n> checked
PR:       #<n> → main
```
