# The Claude Code Bible

> Proven patterns for structuring any project so Claude Code operates at maximum autonomy — writing production code, running tests, deploying, managing infrastructure, and learning from your feedback across sessions.

This isn't theory. These patterns run in production across a 48-repo fintech organization with 18 skills, 2 subagents, 7 commands, 12 infrastructure script suites, hooks, MCP integrations, and a monorepo workspace. They work for Rails, TypeScript, Rust, Python — any stack.

---

## The two pillars

**1. Good architecture and coding patterns** — SOLID, SRP, small files, service objects, automated testing. These aren't just "best practices" — they're what makes AI effective. Claude navigating 50 small focused files is 10× faster than Claude navigating 5 giant files.

**2. Good documentation and planning** — CLAUDE.md, skills, plans, docs directory. But concise. Every markdown file Claude reads costs tokens. A 50-line skill that's all signal beats a 500-line skill padded with fluff. Write docs like you're paying per word (because with AI, you are).

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
| 07 | [Memory and MCP](docs/07-memory-and-mcp.md) | Cross-conversation memory, MCP tool integrations |
| 08 | [Ecosystem](docs/08-ecosystem.md) | Claude Task Master and the broader AI dev ecosystem |
| 09 | [Checklist](docs/09-checklist.md) | Step-by-step setup checklist |
| 10 | [Planning and Docs](docs/10-planning-and-docs.md) | Planning workflow, docs directory, concise documentation |

---

## The core idea

Claude Code reads your project on every conversation. The better organized your project is, the better Claude performs. This isn't "AI-friendly code" as a buzzword — the same things that make code good for humans (small files, clear patterns, documented commands, automated quality) also make it trivially easy for an AI agent to navigate, understand, and modify your codebase autonomously.

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
2. **Allow-all safe commands** — remove permission friction for tests, lint, git, scripts
3. **Response rules** — "No preamble, lead with action, disagree when wrong" changes Claude's entire personality
4. **One lint command** — `bin/lint` that auto-fixes everything, wired as a pre-commit hook

New to this? Start with the [Checklist](docs/09-checklist.md).

---

## Stack-agnostic

These patterns are language-agnostic. Examples across chapters use different stacks (Rails, TypeScript, Rust, Python, Go) to show universality. The structure — CLAUDE.md, skills, scripts, hooks, permissions — works identically regardless of what you write.

---

## License

MIT
