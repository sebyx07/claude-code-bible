# Memory and MCP — Persistence and External Tools

How Claude remembers across conversations and connects to external systems.

---

## Memory — Cross-Conversation Learning

File-based memory at `~/.claude/projects/<project>/memory/`. Persists between conversations — Claude learns from corrections and preferences over time.

### Memory types

| Type | What it stores | Example |
|------|---------------|---------|
| **user** | Who you are, your preferences | "Senior backend dev, prefers terse responses, deep knowledge of distributed systems" |
| **feedback** | Corrections AND confirmed approaches | "Use integration tests for database operations — mocks burned us last quarter" |
| **project** | Ongoing work, decisions, deadlines | "Auth rewrite is driven by compliance, not tech debt — scope favors compliance" |
| **reference** | Pointers to external systems | "Pipeline bugs tracked in Linear project INGEST" |

### How memories are stored

Each memory is a markdown file with frontmatter:

```markdown
---
name: testing-approach
description: Prefer integration tests over mocks for database operations
type: feedback
---

Use real database connections in tests, not mocks.
**Why:** Last quarter, mocked tests passed but the production migration failed.
**How to apply:** All service tests that touch the database should use the test database.
```

An index file (`MEMORY.md`) tracks all memories for quick lookup.

### Why it matters

Without memory, every session starts from zero. With memory:
- No repeating mistakes you've corrected.
- Confirmed preferences applied without reminders.
- Ongoing context: deadlines, decisions, ownership.
- Knows where to look in external systems.

### Memory saves automatically when:

- You correct Claude ("no, don't mock the database")
- You confirm a non-obvious choice ("yes, the bundled PR was right")
- You share project context ("merge freeze starts Thursday")
- You explicitly say "remember this"

**Record from failure AND success.** If you only save corrections, Claude avoids mistakes but drifts from validated approaches.

---

## MCP Servers — External Tool Integration

MCP (Model Context Protocol) = native external tools for Claude. No tab switching.

### Common MCP integrations

| Server | What Claude can do |
|--------|-------------------|
| **Linear / Jira** | Create/update issues, manage projects, search, comment |
| **Playwright** | Open browsers, click elements, fill forms, take screenshots |
| **Sentry / Bugsnag** | Search errors, view stack traces, check production health |
| **Slack** | Read/send messages in channels |
| **Gmail** | Read/send emails |
| **Google Calendar** | View/create events |
| **Custom MCP** | Anything you build — task management, knowledge bases, CI tools |

### How MCP works in practice

**Issue tracking:**
```
User: "Create a ticket for the rate limit bug"
Claude: *calls linear__save_issue with title, description, labels*
Claude: "Created ENG-482: Rate limit handling. Assigned to you."
```

**Browser testing:**
```
User: "Test the new settings page"
Claude: *opens browser, navigates, fills forms, takes screenshot*
Claude: "Settings page loads. Form saves correctly. But the timezone
         dropdown overlaps the save button on mobile — fixing now."
```

**Error monitoring:**
```
User: "Any new errors since the deploy?"
Claude: *calls sentry__search_issues with date filter*
Claude: "3 new issues. Most common: NullPointerException in SyncJob —
         47 occurrences. Here's the fix..."
```

### Setting up MCP

```bash
claude mcp add linear-server -- npx @anthropic/mcp-linear
claude mcp add playwright -- npx @anthropic/mcp-playwright
claude mcp add sentry -- npx @anthropic/mcp-sentry
```

Tools become available in every conversation automatically. Claude discovers what's available and uses them when relevant.

---

## The Full Picture

```
┌──────────────────────────────────────────────┐
│              Claude Code Session              │
├──────────────────────────────────────────────┤
│  CLAUDE.md ──── Project context (every turn) │
│  Skills ─────── Domain expertise (on demand) │
│  Memory ─────── Cross-session learning       │
│  Scripts ────── Infrastructure automation    │
│  MCP ────────── External tools (Linear,      │
│                 Sentry, Browser, etc.)        │
│  Hooks ──────── Quality gates (automatic)    │
│  Permissions ── Trust boundaries             │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Backend  │  │  Infra   │  │ Explore  │  │
│  │  Expert  │  │  Expert  │  │  Agent   │  │
│  │(subagent)│  │(subagent)│  │(built-in)│  │
│  └──────────┘  └──────────┘  └──────────┘  │
└──────────────────────────────────────────────┘
```

Everything feeds into the same conversation. The more you invest in each layer, the more autonomous Claude becomes.
