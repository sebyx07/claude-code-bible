# Compressed Config — Writing `CLAUDE.md` and `.claude/*`

Config files Claude reads every session cost tokens forever. Write them compressed.

Related: [JuliusBrussee/caveman](https://github.com/JuliusBrussee/caveman).

---

## The cost

| File | Read when | Cost |
|------|-----------|------|
| `CLAUDE.md` | every session start | every turn forever |
| `SKILL.md` | skill activates | per activation |
| `agents/*.md` | subagent spawns | per spawn |
| `commands/*.md` | slash command runs | per invocation |

Every filler word = token tax on every session. A 600-line fluffy `CLAUDE.md` costs ~4× a 150-line terse one. Across 1000 sessions that's real money, real latency, real context eviction.

---

## Rules

1. **Lead with the rule, not the reason.** `Business logic in src/services/.` not `It's important that business logic goes in...`
2. **Fragments over sentences.** `Tests: npm test. Lint: bin/lint.` not `To run tests, use npm test. For linting, run bin/lint.`
3. **Tables > paragraphs** for any structured data.
4. **File paths > descriptions.** `see src/services/auth/` beats a paragraph.
5. **Drop filler:** articles, `just`, `really`, `basically`, `in order to`, `it's important to note`, `as mentioned`, pleasantries, hedges.
6. **No meta-framing.** Skip "This section covers...". Get to it.
7. **No rhetoric.** Drop "this is critical", "the single most important", "isn't optional" — show it in the rule.
8. **Code/commands/paths unchanged.** Compress prose only. Exact CLI stays exact.
9. **One rule per line** in rule lists. Easier to scan, grep, cite.
10. **Why only when non-obvious.** If removing the "why" wouldn't confuse a reader, drop it.

---

## Value, not explanation

Claude is your reader. Not your student, not your grader. Reads your `CLAUDE.md` to decide the next action — not to verify you know your own codebase. Lines that don't change a decision are filler. Cut them.

Same logic as humans. Expert readers — human or AI — read selfishly. They scan for what's useful to them *right now*, skip the rest, and abandon prose that doesn't earn its tokens. Writing for Claude is not a special skill; it's writing the way busy experts already read. Adapted from Larry McEnerney's *Craft of Writing Effectively* (UChicago) — readers stop, misunderstand, or get aggravated when prose signals continuity instead of value.

### Value-words (the community code)

Words that tell Claude this matters and what to do:

| Function | Words |
|----------|-------|
| Hard rule | `MUST`, `NEVER`, `ALWAYS`, `required`, `forbidden` |
| Cause/cost | `breaks when`, `fails if`, `causes`, `without this` |
| Boundary | `before X`, `after Y`, `only in`, `not in` |
| Trigger | `when Z`, `if user asks`, `on commit` |

A `CLAUDE.md` section with zero value-words is probably narration. Rewrite as instruction.

### Kill flow-words

Drop: `however`, `moreover`, `furthermore`, `in addition`, `additionally`, `thus`, `therefore`, `as such`, `it should be noted`. They signal essay-mode — Claude is looking for rules, not transitions.

### Open with a constraint, not a tour

| ❌ Tour | ✅ Constraint |
|---|---|
| "This is a Rails app using PostgreSQL. We follow service-oriented patterns…" | "Business logic → `app/services/<domain>/`. Controllers thin. Custom errors only — no `Exception`." |

The tour is you talking to yourself. The constraint changes what Claude writes next.

### Gap ≠ error

| Weak (gap) | Strong (error) |
|------|--------|
| "We haven't documented the webhook flow." | "Webhooks: always call `Webhook.verify` first — signature verification was missed twice." |
| "Auth is complex." | "Auth: never read `current_user` in models. Pass it explicitly." |

"Gap" assumes Claude will fill it correctly. "Error" tells Claude what went wrong before and prevents the repeat.

---

## `CLAUDE.md` — before / after

### Before (fluff)

```markdown
# My App

## Overview

My App is a SaaS platform that helps businesses manage their customer
relationships. It's built on Rails 7.1 with PostgreSQL and Redis.

## Architecture

We follow a service-oriented architecture pattern. All business logic
should be placed in service objects under app/services/, organized by
domain. Controllers should be kept thin — they parse the request,
delegate to a service, and render the response. Models handle
persistence only — no business logic.

## Testing

To run tests, we use RSpec. You can run the full test suite by executing
`bundle exec rspec`. For faster feedback during development, you can
run a specific file with `bundle exec rspec path/to/spec.rb`.
```

~90 words of prose.

### After (compressed)

```markdown
# My App

Rails 7.1 · PostgreSQL · Redis · SaaS CRM.

## Architecture
- Business logic → `app/services/<domain>/`
- Controllers: thin. Parse → service → render.
- Models: persistence only. No business logic.

## Testing (RSpec)
bundle exec rspec                    # all
bundle exec rspec path/to/spec.rb    # one file
```

~30 words. Same info. 3× less to read every session.

---

## `SKILL.md` template

```markdown
---
name: deploy
description: Deploy app — test, build, push, verify. Triggers: deploy, ship, release.
allowed-tools: [Bash, Read]
---

# Deploy

1. `bin/test` · fail → stop
2. `bin/lint` · fail → stop
3. `scripts/ci/trigger-build production`
4. `scripts/ci/build-log` · tail until done
5. `scripts/hosting/app-status production` · verify healthy
6. Fail → `scripts/hosting/rollback production`
```

Rules:
- **`description`** — keyword-packed, ends with triggers. Claude matches on this.
- **Numbered steps** for sequential workflows. One action per step.
- **Exact commands.** No prose wrappers like "first we'll run tests".
- **Failure branch inline.** `fail → stop` beats a "Error handling" section.
- **≤80 lines.** Split into supporting `.md` files if longer.

---

## `agents/<name>.md` template

```markdown
---
name: backend-expert
description: Backend specialist. Models, services, controllers, migrations, tests.
model: opus
skills: [development, database-design, performance-optimization]
---

Backend specialist. This project.

## Rules
- SOLID. SRP hard.
- Services under `app/services/<domain>/`.
- Custom errors only. No generic `Exception`.
- Tests required. RSpec. TDD where possible.
- Files ≤500 LOC. Split → nested modules.
- Performance: memoize, batch queries, background jobs for >100ms work.
```

Rules:
- Persona = one sentence, not a paragraph.
- Rule list = one rule per line, imperative.
- Skip `## Overview`, `## Responsibilities`, `## Approach`. Just rules.

---

## `commands/<name>.md` template

```markdown
---
description: Review PR — regressions, security, style, test coverage
argument-hint: [pr number]
allowed-tools: [Bash, Read, Grep, Glob]
---

# /review

1. `gh pr diff $1`
2. Check: regressions · security · style · test coverage
3. Output one-line comments per finding: `L42: 🔴 bug: X. Fix: Y.`
4. No throat-clearing. No summary.
```

Rules:
- Number the steps.
- Specify output format literally — show the pattern.
- Forbid fluff explicitly (`No throat-clearing.`).

---

## Response rules (in `CLAUDE.md`)

Shape Claude's output style across the session. Paste near the top:

```markdown
## Response Rules
- Execute. No preamble. No "I'll start by...". No restating the task.
- Lead with action or answer. Reasoning after, only if non-obvious.
- Parallel tool calls when independent.
- Read before speculating.
- Disagree when user is wrong. State the correction.
- Terse. Fragments OK. Drop articles, filler, hedging.
- Code, commits, commands: normal prose/syntax. Only responses get compressed.
- Summary at end: 1-2 sentences max. Nothing else.
```

This alone cuts ~30% of output tokens for free. Compounds with compressed configs.

---

## What NOT to compress

Leave these verbatim — compression loses information:

- **Code blocks** — exact syntax matters.
- **Shell commands** — CLI flags are load-bearing.
- **File paths** — must resolve.
- **URLs, versions, env var names** — literal strings.
- **Regex, queries, config values** — exact characters.
- **Error messages** to match against — must be exact.

Compress the prose around them, not the tokens themselves.

---

## Layered `CLAUDE.md` (monorepo)

```
workspace/
├── CLAUDE.md              # Org: stack, categories, cross-repo rules
└── repos/
    ├── backend/CLAUDE.md  # Project: architecture, commands
    └── frontend/CLAUDE.md # Project: same pattern, different stack
```

Claude loads both. Keep each layer to its scope — no duplication. Org file: cross-cutting. Project file: local only.

---

## Self-check before committing a config file

- [ ] Opening paragraph removed or cut to one line
- [ ] All "Why this matters" sidebars either merged to one top statement or deleted
- [ ] Every list item ≤15 words
- [ ] Prose transitions (`Next, we'll...`, `Now let's...`) removed
- [ ] Hedges (`might`, `generally`, `in most cases`) removed unless load-bearing
- [ ] Tables used for any ≥3-row structured data
- [ ] Commands quoted exactly, not described
- [ ] File ≤ target (CLAUDE.md <600 · SKILL.md <80 · agent <50 · command <30)
- [ ] Every section has at least one value-word (`MUST`/`NEVER`/`fails when`/`breaks if`/`when X`)
- [ ] Flow-words (`however`/`moreover`/`thus`/`furthermore`) removed
- [ ] Opens with a constraint or rule, not project history

---

## Compression is not cryptic

Terse ≠ unreadable. A reader should understand more per second, not less. If a cut makes the file harder to parse, restore it. Target: **high info density**, not **low word count for its own sake**.

The breath test: can a new engineer read a section in one breath and know what to do? If yes, compressed enough. If no, too dense — add structure back (bullets, headers, one more sentence).
