# The Claude Code Bible

> Proven patterns for structuring any project so Claude Code operates at maximum autonomy — writing production code, running tests, deploying, managing infrastructure, and learning from your feedback across sessions.

Battle-tested across 50+ projects — the whole spectrum of software. Rails, TypeScript, Python, Rust, Go, React, desktop apps, Android, iOS, Linux drivers, embedded, ML pipelines, CLIs. 18 skills, subagents, slash commands, infrastructure scripts, hooks, MCP integrations, monorepo workspaces. Stack-agnostic.

---

## The two pillars

**1. Architecture and coding patterns** — SOLID, SRP, small files, service objects, automated testing. What makes code good for humans also makes it trivially navigable for AI. Claude across 50 small focused files is 10× faster than across 5 giant ones.

**2. Compressed docs and planning** — `CLAUDE.md`, skills, plans. Every markdown file Claude reads costs tokens on every session. A 50-line skill all signal beats a 500-line one padded with fluff. Write configs like you're paying per word — with AI, you are. See [ch. 11](docs/11-compressed-config.md).

---

## Table of contents

| # | Chapter | What it covers |
|---|---------|----------------|
| 01 | [CLAUDE.md](docs/01-claude-md.md) | The project brain Claude reads every session |
| 02 | [Skills, Agents, Commands](docs/02-skills-agents-commands.md) | Skills, subagents, and slash commands |
| 03 | [Code Quality](docs/03-code-quality.md) | SOLID, SRP, small files, linting, testing — why AI needs clean code |
| 04 | [Scripts and Env](docs/04-scripts-and-env.md) | Infrastructure scripts, `.env` files, giving Claude hands |
| 05 | [Hooks and Permissions](docs/05-hooks-and-permissions.md) | Quality gates, allow-all tools, trust boundaries |
| 06 | [Monorepo Workspace](docs/06-monorepo-workspace.md) | Multi-repo workspace, multi-agent, scaling across servers |
| 07 | [Memory and MCP](docs/07-memory-and-mcp.md) | Cross-conversation memory, MCP tool integrations, building efficient MCP servers |
| 08 | [Ecosystem](docs/08-ecosystem.md) | Claude Task Master and the broader AI dev ecosystem |
| 09 | [Checklist](docs/09-checklist.md) | Step-by-step setup checklist |
| 10 | [Planning and Docs](docs/10-planning-and-docs.md) | Planning workflow, docs directory, concise documentation |
| 11 | [Compressed Config](docs/11-compressed-config.md) | Writing `CLAUDE.md` and `.claude/*` for minimum tokens |

---

## The core idea

Claude reads your project every conversation. Good organization → better performance. Small files, clear patterns, documented commands, automated quality — same things that make code good for humans make it trivially navigable for AI.

## The workflow this enables

```
/plan add feature X     → Claude explores codebase, creates implementation plan
/implement              → Claude executes the plan, writes code + tests
/qa                     → Claude reviews for regressions, runs full test suite
/deploy                 → Claude triggers CI, builds, deploys to prod
```

You approve at checkpoints. Claude does the rest.

---

## Quick wins (do these first)

The four things that give you 80% of the value in under an hour:

1. **[CLAUDE.md](docs/01-claude-md.md) with commands** — document your test, lint, and deploy commands
2. **Skip the prompts** — `claude --dangerously-skip-permissions`. Zero friction. See [ch. 5](docs/05-hooks-and-permissions.md)
3. **Response rules** — "No preamble, lead with action, disagree when wrong" changes Claude's entire personality
4. **One lint command** — `bin/lint` that auto-fixes everything, wired as a pre-commit hook

New to this? Start with the [Checklist](docs/09-checklist.md).

---

## Stack-agnostic

Language-agnostic. Examples across chapters span Rails, TypeScript, Python, Rust, Go, C, Swift, Kotlin. The structure — `CLAUDE.md`, skills, scripts, hooks, permissions — works identically across backend services, frontend apps, desktop, mobile, drivers, embedded, ML.

---

## License

MIT
