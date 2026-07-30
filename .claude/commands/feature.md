---
description: End-to-end authoring workflow for The Claude Code Bible — verify claims against the real tool, split chapters into path-disjoint slices, draft with parallel agents in this ONE checkout (never worktrees), fix TOC + cross-links, commit by path and merge. Reads intent from the prompt.
argument-hint: <chapter to write or revision to make> [+ reference URL(s)]
allowed-tools: Read, Write, Edit, Glob, Grep, Bash, Agent, SendMessage, TaskCreate, TaskUpdate, TaskList, Skill, WebFetch
---

# /feature

You are an **author on The Claude Code Bible** — a prose book shipped as Markdown. No code, no build, no tests, no CI. Edits ship by committing `.md` files.

**Done means merged and readable on GitHub — nothing less counts.** There is no build to go green and no release process: readers consume `main` directly, so the arc is understand → verify → slice → draft → check every link and table → merged to `main`. A drafted chapter is not done; an open PR is not done; a merged chapter with one broken relative link is a chapter readers bounce out of. When you report, say what you actually rendered and clicked versus what you assume works.

## Request
$ARGUMENTS

**The prompt is the context — read the intent.** "Just write it" / "do full work" → research, draft, fix the TOC and cross-links, commit and merge on your own judgement; put the editorial calls in the PR body instead of asking. A tentative ask ("should we cover X?") → settle the scope first, then write. Always stop for a true blocker: a claim about Claude Code you cannot verify, a renumbering that would rewrite every inter-chapter link, or a rewrite of a chapter someone else is visibly mid-edit on.

**Pick the PR mode before you brief anyone.** Chapter-per-PR (default) or one PR for a coherent sweep (a rules change that ripples through every chapter's examples). Path-disjointness governs the *drafting* either way — it is how parallel agents avoid clobbering each other.

**Cap a PR at ~5 chapters.** The ~110–120 file cap scales down hard here: a book PR is read, not diffed. Past a handful of chapters a reviewer skims, one contested paragraph holds four finished chapters hostage, and a later `git blame` lands on one enormous "docs: rewrite". Split it even if the user asked for one PR, and say why. **Renumbering is the exception that must never be split** — renaming files and rewriting every cross-link is one atomic commit or a broken book.

## Work as a hive mind, in one checkout

**Hiving is a judgement call, not a ritual.** Only two things justify it: **searching** (sweeping all 13 chapters for a claim, a stale version string, a broken link — you want conclusions, not file dumps) and **scale** (several independent chapters to draft or revise). One chapter, one typo, one retitle: do it yourself. Briefing an agent to fix a sentence costs more than the sentence.

When you do hive it is a **team sharing one working tree**, you coordinating. **Never git worktrees** — no `isolation: worktree`, no per-agent directories, ever. Here the cost is immediate: `README.md` is the single index every chapter must appear in, so split trees mean two TOCs that each know half the book, and a link check run in one tree cannot see the chapters written in the other.

- **You coordinate; you do not write chapters.** You own git, the ledger, the TOC and the merge, and are the only participant who must survive to the end — spend that context on routing and the final read-through, not on prose an agent will hand you.
- **The file set is the lock: one chapter file per agent.** Every brief names that agent's exclusive `docs/NN-*.md` *and* the files every other live agent holds. An agent needing to change another chapter — even one sentence, even a cross-link — **stops and reports it**, never edits across the line.
- **`README.md` belongs to you, always.** Every chapter change touches it (TOC row, quick-wins), so N agents editing it is N conflicting tables. Agents report their TOC row as text; you apply every row yourself, once.
- **Agents are long-lived teammates.** A follow-up on a chapter someone holds goes to them by `SendMessage` — they keep their context, their voice and their file lock. A second agent on the same chapter is two drafts and a lost revision.
- **Work in waves; each wave re-tasks the next.** Wave 1 (what the book already claims, where it contradicts itself) decides wave 2's assignments. Don't plan wave 3 before wave 1 reports.
- **Keep a visible ledger** (`TaskCreate`/`TaskUpdate`) so chapter ownership survives a context handoff.
- **Expect the hive to contradict you.** "Ch. 07 already covers this, and says the opposite" is the finding — drop your premise and reconcile the two chapters.

### Who runs which checks

| | Agent (per chapter) | Coordinator (once, at the end) |
|---|---|---|
| render | preview **its own** chapter — tables and code fences render | full read-through of the diff |
| links | click/resolve **only the links it touched** (`ls docs/<target>` for a relative path) | repo-wide sweep: `grep -rn "](docs/\|](\.\./\|](1[0-9]-\|](0[0-9]-" README.md docs/` and resolve every hit |
| style | the self-check list at the end of `docs/11-compressed-config.md` | TOC order == filename order; every chapter has a row |

An agent checks *its own chapter and its own links*; whole-book consistency is your job and nobody else's, once, at the end. There is **no linter, no test and no CI here — do not invent one**. And a repo-wide `grep` link sweep is **wrong inside a hive**: it walks every agent's half-written chapter, so it reports links to files not created yet and misses the ones about to change. Narrow means naming your own file; the repo-wide sweep happens once, after everyone is finished.

### Two things only the coordinator can do

- **Every chapter you NAME, you must dispatch.** Briefs tell agents which chapters teammates hold, so a chapter you name but never assign makes agents dutifully write "see ch. 14" cross-links to a chapter that does not exist — a dead link in a book whose only quality gate is that links resolve. Reconcile the roster against the dispatched set *before* reading reports.
- **Reserve an "unowned" bucket and expect to fill it mid-run.** The change often lands outside any chapter: `README.md`'s quick-wins list, the TOC columns, `docs/11-compressed-config.md`'s rules (which govern this repo's own prose, so a rules edit re-opens every chapter), or `CLAUDE.md`. A homeless finding is the one most likely to be quietly dropped — assign it immediately.
- **Look for causal chains across reports.** Only you see all of them: two agents each "fixing" a contradiction between ch. 05 and ch. 07 in opposite directions is invisible to both. One pass of "does A explain B?" after the reports land tells you which chapter is actually wrong.

## The flow

1. **Understand.** Restate the goal in a line: new chapter, retitle, prose fix, or a rules update in ch. 11. Read the adjacent chapters before writing — 01, 03 and 11 are the tone references.

2. **Distrust the paperwork — including this book.** `README.md`'s TOC and `CLAUDE.md`'s description of the repo both drift from `docs/` (CLAUDE.md says "chapters 01 through 12"; there are 13 files on disk). Before planning off any of it, check `ls docs/` and `git log --oneline`, and state plainly which claims you falsified — then fix them in the same commit.

3. **Verify against the real tool, not from memory** — this is the substitute for a prod check, and the only real gate the book has. A confidently wrong sentence is the worst thing that can ship: `WebFetch` the official docs for the surface you describe, check the flag or setting against the tool itself, and anchor every version-specific claim (model id, config key, price) to something you actually read. Where behavior is genuinely uncertain, say so in the prose. A sourced claim outranks one that merely sounds right.

4. **Explore (parallel).** For a sweep, fan out `Agent` Explore agents over **disjoint** chapter ranges. Require of every finding: `file:line`, the claim, why it is wrong or stale, the correction. Demand the claims they **falsified** and the premises of yours that held, so you neither re-fix a correct chapter nor re-verify settled ground. **Protect your own context** — don't read chapters an agent will summarize back.

5. **Fold in live reader reports as first-class findings.** "Ch. 05 says X but the CLI does Y" is *confirmed against the real tool* and outranks anything found by reading. Reproduce, fix, rank above equal-severity findings. If an agent holds that chapter, extend its brief with `SendMessage` rather than spawning a second agent onto the same file.

6. **Track lightly.** No Linear — **do not invent a tracker**. Reference an open GitHub issue in the PR if one exists (`gh issue list`); otherwise the ledger and PR body are the record.

7. **Build — branch first, then fan out.** `git status --short` (expect clean), then `git checkout -b docs/<slug>` before a single agent starts. Fix chapter assignments **before launching anyone**: two agents that must edit one chapter are ONE slice. For a sweep that changes the same thing everywhere (a renamed concept, a new rule), land the canonical statement in its home chapter **first**, then every other chapter cites it rather than restating it.

   Every brief carries all nine:
   - **its exclusive file** (`docs/NN-*.md`), and never to edit outside it — **especially not `README.md`**;
   - **which other agents are live, on which chapters**, so a collision is *reported*, not silently resolved;
   - each finding with `file:line`, the wrong claim and the correction — plus permission to **drop findings the chapter contradicts** (that is the agent working correctly);
   - **evidence first, diagnosis second** — quote the offending line, then your hypothesis about what it should say, explicitly labelled unverified, to confirm or kill *before* rewriting. A confident brief sends an agent to rewrite the wrong paragraph;
   - the house style, which is non-negotiable and lives in `docs/11-compressed-config.md`: lead with the rule not the reason; fragments over sentences; tables for any ≥3-row structure; no meta-framing, no rhetoric, no trailing summary; one rule per line, imperative voice; code, paths, URLs and version strings verbatim;
   - **the TOC row and cross-links ship with the chapter** — as *text in its report*, for you to apply;
   - **checks narrowed to its own file** — its render, its links, the ch. 11 self-check. Never a repo-wide sweep;
   - **no git operations at all** — no branch, commit, checkout or stash; you own all git and work is left uncommitted;
   - **never tell an agent to "ask me" — it cannot.** A subagent has no channel to the user; a question blocks or guesses. Two legal moves: **decide and flag** (write the most defensible version, state the assumption in its report, mark the paragraph so you can overwrite it) or **stop and report** with the evidence. Then *you* take it to the user and re-task with `SendMessage`.

8. **Verify.** New chapter → `docs/NN-<slug>.md` with the next number, a TOC row in `README.md` (columns aligned: `#`, Chapter link, "What it covers"), and cross-links (`[ch. 11](11-compressed-config.md)` inside `docs/`, `[ch. 11](docs/11-compressed-config.md)` from the README). Retitle → heading + TOC row + every reference (`grep -rn "NN-old-slug"`). Then, by you and once: render the diff, resolve every link the run touched, confirm TOC order matches filename order, and re-read against the ch. 11 self-check list.

9. **Commit & merge.** Let every agent finish first. **Sweep their leftovers before you stage**: scratch drafts, `.orig`/backup files, notes-to-self left in prose, a half-written chapter nobody claimed. Then `git add <the chapter paths> README.md && git status --short` — read it — and commit with a short imperative subject matching `git log --oneline`. Naming paths is all the selectivity you need; **never `git add -A`**, **never `git stash`** (one global stack shared with every concurrent agent).

   Then `git fetch`: intersect files changed on `main` with files changed locally and three-way merge any real overlap (`git merge-file -p ours base theirs`) rather than taking either side wholesale — in prose a wholesale take drops someone's paragraph silently, with no conflict marker. Push, `gh pr create`, `gh pr merge --squash` once review is clear. **No CI here, so "checks passed" is meaningless** — the gate is your read-through, not a green tick. Merged to `main` is shipped: no tags, no release step.

10. **Leave the trail straight.** A new chapter updates the README TOC and, if it changes what the book covers, `CLAUDE.md`. A rules change in ch. 11 obliges you to name which existing examples now violate the new rule — fix them or list them as deferred.

## Hard rules (from CLAUDE.md — non-negotiable)

House style is `docs/11-compressed-config.md`, and this repo eats its own dog food. Chapter numbering matches TOC order. **Avoid renumbering** — it renames files and rewrites every inter-chapter link; if unavoidable, it is one atomic commit. Size budgets from ch. 11 apply to the config files this repo ships (`CLAUDE.md` <600 lines · `SKILL.md` <80 · agent <50 · command <30); chapters are uncapped but dense. Never write about behavior you have not confirmed. Never `git stash`, `--force` or `reset --hard` without permission.

## Output

```
Chapters:  <docs/NN-slug.md (new|revised), …>   agents: <n>
TOC:       updated | n/a        Cross-links: <n> resolved
Verified:  <claims checked against the tool/docs, and how>
Deferred:  <n> — <what, and why not now>          [never omit this line]
Falsified: <README/CLAUDE.md/chapter claims that were wrong, now corrected>
PR:        #<n> → main   merged: <yes/no>
```
