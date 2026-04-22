# Ecosystem — Claude Task Master and Beyond

Claude Code = foundation. A growing ecosystem of tools extend it.

---

## Claude Task Master

**[github.com/developerz-ai/claude-task-master](https://github.com/developerz-ai/claude-task-master)**

An autonomous task orchestration system that wraps the Claude Agent SDK. It keeps Claude working on a goal until fully complete — without human babysitting between steps.

### What it does

Given a goal like "Add user authentication with tests," it autonomously:

1. **Plans** — analyzes the codebase, breaks the goal into PR-sized tasks
2. **Implements** — writes code + tests for each task
3. **Commits & pushes** — creates branches, commits, pushes
4. **Creates PRs** — one PR per logical task
5. **Waits for CI** — monitors CI status
6. **Fixes failures** — auto-detects and fixes failing checks
7. **Addresses reviews** — responds to PR review comments
8. **Merges** — merges when approved

It persists state between sessions, so it survives interruptions.

### Key features

- **PR-based workflow:** All work must be in a PR before a task is considered done
- **CI integration:** Auto-detects and fixes failing checks
- **Mailbox system:** Inject dynamic plan updates while it's running
- **Multi-instance:** Multiple agents working in parallel on different tasks
- **State persistence:** Resume after crashes or interruptions

### How it relates to this guide

Claude Task Master is a **supervisor loop** — it sits above Claude Code and drives it end-to-end. The patterns in this guide (CLAUDE.md, skills, scripts, hooks) make Claude Task Master more effective because the agent it's driving has better context and tooling.

Think of it this way:
- **This guide** = making each Claude session maximally effective
- **Claude Task Master** = chaining sessions together into autonomous workflows

---

## Other Tools Worth Knowing

### Aider
Git-aware AI coding assistant. Good for targeted code changes. Different philosophy from Claude Code — more focused on diffs, less on full-session autonomy.

### Cursor / Windsurf
IDE-integrated AI coding. Good for interactive development. Less autonomous than Claude Code — more copilot, less agent.

### Claude Code vs. the ecosystem

| Tool | Strength | Mode |
|------|----------|------|
| **Claude Code** | Full autonomy, skills, scripts, deploy | Agent — works independently |
| **Claude Task Master** | Multi-PR orchestration, CI loops | Supervisor — chains agent sessions |
| **Cursor/Windsurf** | Real-time IDE assist, quick edits | Copilot — works alongside you |
| **Aider** | Git-aware diffs, targeted changes | Tool — focused on specific changes |

They're not competitors — they serve different needs. Claude Code with good project structure (this guide) is the foundation. Layer on Task Master for full autonomy, use Cursor for quick interactive edits.

---

## Building Your Own MCP Tools

MCP (Model Context Protocol) lets you create custom tool servers that Claude can use natively. This is how you extend Claude's capabilities to your specific domain.

### Example: custom deployment MCP

Instead of scripts, you could build an MCP server that gives Claude deploy tools directly. Keep the surface small and parameterized — one `deploy` tool with verbs beats five narrow tools (see [ch. 7](07-memory-and-mcp.md#building-mcp-servers--few-parameterized-tools-beat-many) for why):

```
deploy({ action: "promote", service: "api", env: "staging", branch: "main" })
deploy({ action: "promote", service: "api", env: "production" })
deploy({ action: "rollback", service: "api", version: "v1.42.0" })
deploy({ action: "status",   service: "api" })
deploy({ action: "logs",     service: "api", lines: 200 })
```

Claude calls these as native tools — no shell commands needed. The MCP server handles auth, validation, and safety internally.

### When to use MCP vs scripts

| Use scripts when... | Use MCP when... |
|---------------------|-----------------|
| Simple API wrapper | Complex multi-step tool |
| One-off commands | Tool needs state/context |
| Team doesn't use Claude | Team standardizes on Claude |
| Quick to build | Worth the investment |

Both work. Scripts are simpler to build and maintain. MCP gives tighter integration and richer tool descriptions that help Claude use them more effectively.
