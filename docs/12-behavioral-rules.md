# Behavioral Rules — Steering How Claude Codes

Response rules ([ch. 01](01-claude-md.md#response-rules)) shape how Claude *talks*. Behavioral rules shape how Claude *codes* — what it assumes, how much it builds, what it touches, how it knows it's done.

Derived from [Karpathy's observations](https://x.com/karpathy/status/2015883857489522876) on LLM coding pitfalls, packaged by [multica-ai/andrej-karpathy-skills](https://github.com/multica-ai/andrej-karpathy-skills).

---

## The four failure modes

| Failure | Looks like | Counter-rule |
|---------|-----------|--------------|
| Silent assumptions | Picks one interpretation of an ambiguous request and runs | Think before coding |
| Overengineering | 1000 lines, strategy patterns, config flags — where 100 lines would do | Simplicity first |
| Drive-by edits | Reformats quotes, adds type hints, rewrites comments orthogonal to the task | Surgical changes |
| Vague execution | "I'll review and improve the code" — no done-condition | Goal-driven execution |

Untreated, every one of these costs a rewrite. The rules below prevent them per-session, for the cost of ~30 lines in `CLAUDE.md`.

---

## Paste-ready block

Drop into `CLAUDE.md` next to your response rules:

```markdown
## Coding Rules

### Think before coding
- State assumptions explicitly. Uncertain → ask, don't guess.
- Multiple interpretations → present them, don't pick silently.
- Simpler approach exists → say so. Push back when warranted.
- Confused → stop. Name what's unclear. Ask.

### Simplicity first
- Minimum code that solves the stated problem. Nothing speculative.
- No abstractions for single-use code. No unrequested flexibility/config.
- No error handling for impossible cases.
- 200 lines that could be 50 → rewrite as 50.

### Surgical changes
- Touch only what the task requires. No drive-by refactors, reformatting, comment edits.
- Match existing style, even when you'd do it differently.
- Remove only orphans YOUR change created. Pre-existing dead code: flag, don't delete.
- Every changed line must trace to the request.

### Goal-driven execution
- Convert tasks to verifiable goals before coding:
  - "Fix the bug" → write reproducing test → make it pass
  - "Add validation" → write tests for invalid inputs → make them pass
  - "Refactor X" → tests green before AND after
- Multi-step work: plan as `step → verify: check`, one verification per step.
```

---

## Success criteria beat instructions

Karpathy: *"LLMs are exceptionally good at looping until they meet specific goals... Don't tell it what to do, give it success criteria and watch it go."*

| Imperative (weak) | Verifiable (strong) |
|-------------------|---------------------|
| "Make the search faster" | "Search p95 < 100ms, measured by `bin/bench search`" |
| "Fix the auth bug" | "Test: password change invalidates old sessions. Make it pass." |
| "Clean up the user model" | "User model split to ≤500 LOC each. `bin/test` green before and after." |
| "Add rate limiting" | "11th request in 60s returns 429. Test proves it." |

Strong criteria let Claude loop independently — edit, run, check, repeat — without you in the loop. Weak criteria pull you back in for every judgment call. This is the same `step → verify` shape as plan files in [ch. 10](10-planning-and-docs.md#planning--before-you-code); behavioral rules make Claude apply it to *every* task, not just planned ones.

---

## The self-checks

One-line tests Claude (and you, reviewing) can apply:

- **Overengineering**: would a senior engineer call this overcomplicated? Yes → simplify.
- **Surgical**: does every changed line trace to the request? No → revert the strays.
- **Done**: is there a command that proves it works? No → write the test first.

The overengineered version is rarely *obviously* wrong — it follows design patterns, handles edge cases, looks professional. The problem is timing: complexity added before it's needed. Simple now, refactor when the requirement actually arrives.

---

## Scale rigor to the task

These rules bias caution over speed. A typo fix doesn't need an assumptions audit; a schema migration does.

| Task size | Rigor |
|-----------|-------|
| Typo, one-liner, obvious fix | Just do it |
| New function, bug fix | Success criteria + surgical diff |
| New feature, refactor, migration | Full: assumptions surfaced → plan with verification → tests first |

---

## How to know it's working

- Diffs contain only requested changes — no drive-by "improvements".
- Clarifying questions arrive *before* implementation, not after mistakes.
- Fewer rewrites caused by overcomplication.
- PRs reviewable in one pass.

---

## Where to put them

| Location | Tradeoff |
|----------|----------|
| `CLAUDE.md` (recommended) | Always on. ~30 lines ≈ cheap insurance every session. |
| Skill (`.claude/skills/`) | Loads per-activation only — but behavioral drift happens on tasks that *don't* trigger skills. |
| Plugin ([install](https://github.com/multica-ai/andrej-karpathy-skills#install)) | Global across projects, zero per-repo setup. |

Compress before pasting — the block above already follows [ch. 11](11-compressed-config.md) rules. Resist expanding it with explanations; the rules are the explanation.
