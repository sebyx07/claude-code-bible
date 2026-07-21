# Workflow Commands — `/planx` and `/feature`

Two commands carry most of the delivery weight in a mature repo: one that writes a plan, one that executes work end-to-end. [Ch. 2](02-skills-agents-commands.md) covers command mechanics — frontmatter, invocation, where files live. This chapter is about the pair that earns its keep.

## Why two, not one

| | `/planx` | `/feature` |
|---|---|---|
| Produces | files in `docs/plans/…` | merged PRs in prod |
| Touches source | never | always |
| Costs | one exploration pass | the whole build |
| Rerunnable | yes, cheaply | no |
| Who consumes it | a *different* agent or person, later | the repo |

Planning and building fail differently. A bad plan costs an exploration pass; a bad build costs a revert. Splitting them puts a durable, reviewable artifact between the two — one you can read, correct, hand to someone else, or execute six weeks later. Fusing them into a single "do the thing" command means the plan exists only inside one agent's context and dies with it.

The split also makes the expensive half restartable. An agent that dies mid-`/feature` picks up from the plan files, not from scratch.

## `/planx` — the plan is files, not a message

Rules that make plans executable by an agent that wasn't there when they were written:

- **Write to disk, always.** `docs/plans/<YYYY>/<MM>/<DD>/<1NN>-<slug>/`. A plan in chat is gone next session.
- **Multiple files, never one `plan.md`.** An `overview.md` index plus one `NN-<aspect>.md` per separable area — data model, API routes, UI, tests. Each slice stays short and independently executable.
- **Reference, don't restate.** Point at `file:line` and `Class#method`. Pasting code into a plan guarantees it goes stale.
- **Self-contained per slice.** The executor reads `overview.md`, its own slice, and the files those cite. Nothing else.
- **No checkboxes in the `.md` files.** They're reference maps. Tracking lives in exactly one place.

### `status.yml` — the one file that is a tracker

```yaml
plan: 101-internal-exchange-rates
status: in_progress        # not_started | in_progress | blocked | complete | superseded
created_by: sebi           # who wrote the plan
worked_by: ""              # who is executing; empty = unclaimed
percent: 40
current_focus: "03-api-routes.md — rate lookup endpoint"
slices:
  - file: 01-data-model.md
    status: complete
    percent: 100
evidence: ["#324", "abc1234"]
last_updated: 2026-07-21
```

Machine-readable so an orchestrator can query it. `created_by` separate from `worked_by` is what lets one person plan and another execute — the executor stamps their own name on pickup. `evidence` holds commits and PRs, so "80% done" is a claim you can check.

## `/feature` — one command, idea to deployed

The flow, compressed:

1. **Understand** — restate the goal in a line. Fetch any cited URLs, extract the *mechanism*, translate onto this stack.
2. **Explore in parallel** — fan out read-only agents to map every affected surface. Produce a worklist grouped into PR-sized batches.
3. **Track in the issue tracker** — one sub-issue per PR-sized slice; `Fixes #NNN` so merges auto-close.
4. **Build primitive-first** — for a sweep across N surfaces, land one reusable primitive with its first real caller, then adopt it everywhere. No abstractions before consumers.
5. **Verify** — typecheck, lint, test as the green gate. User-facing changes get driven in a real browser.
6. **PR and merge sequentially** — never merge in parallel; each merge rebases `main` under the others.
7. **Deploy and watch** — confirm the roll actually landed with a live probe, not a bundle grep.
8. **Close** — verify each auto-close fired; close the parent by hand.

### Read the intent from the prompt

The command should infer autonomy level from how the request is phrased rather than making the user configure it. "Just ship it" → run start to finish, decide everything, merge on green, surface decisions in the PR body instead of asking. A tentative ask → clarify what's genuinely ambiguous and stop before merge. Always stop for a true blocker: irreversible prod action, data-integrity or auth risk, a policy violation.

### Parallel agents share the checkout — no worktrees

Fan-out is for *reading and building*, not for cloning the repo N times.

Git worktrees look like the obvious way to parallelize agents, and they cost more than they save:

- Work lands in a directory you aren't looking at. `git status` in your checkout says clean while three branches of real work sit elsewhere.
- Every worktree needs its own dependency install, its own copied `.env*`, its own test database. For a Bun or Node monorepo that's minutes and gigabytes per agent.
- Commits get stranded. A worktree removed before its branch is pushed takes the work with it.
- Your own tooling — the language server, the test watcher, the dev stack on a fixed port — is pointed at one directory. Agents elsewhere run half-blind.

One checkout, one branch at a time. Agents working simultaneously coordinate over disjoint file sets, which is a scheduling problem you can see, rather than an isolation problem you can't. Where genuine isolation is required — a destructive migration rehearsal, a dependency bump you expect to explode — that's a container or a throwaway clone, not a worktree hanging off the repo you're using.

State it in the command file itself, not only in `CLAUDE.md`. The agent reading `/feature` is the one about to spawn the fan-out.

## Keep them short

Command budget from [ch. 11](11-compressed-config.md) is <30 lines for a simple command. These two are the justified exception — they encode a whole delivery flow — but the pressure still applies: point at skills and docs rather than inlining their content. A `/feature` that restates the deploy runbook rots the moment the runbook changes. Link to it.

Both files are per-repo. The flow is the same everywhere; the stack names, commands, and directory layout are not. Copying another project's `/feature` verbatim gives an agent confidently wrong build commands — adapt it, or it's worse than nothing.
