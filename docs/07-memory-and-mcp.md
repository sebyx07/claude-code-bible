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

## Building MCP Servers — Few Parameterized Tools Beat Many

The single biggest mistake when building an MCP server: shipping one tool per action. `create_user`, `update_user`, `list_users`, `get_user`, `delete_user`, `archive_user` × every model = 50+ tools. Every one of those names, descriptions, and JSON schemas sits in the model's context **on every turn** after `tools/list`. 50 tools × ~200 tokens of schema = 10k tokens of overhead per request, forever — eating the budget you wanted for actual work.

**AIs are smart. They can dispatch on a parameter.** You don't need a separate tool per verb — one parameterized tool with a `resource` and `action` works. Let the model decide; that's what it's good at.

### The 3-tool meta pattern

Expose a flat, constant-size surface regardless of how many resources you have:

| Tool | Purpose |
|------|---------|
| `list_resources` | Catalog: `[{ name, description, actions: [...] }, ...]`. Cheap. |
| `describe_resource` | Schema for one resource (optionally one action). Lazy — only loads what's needed. |
| `manage_resource` | Single dispatcher: `{ resource, action, ...params }`. Routes to the registered handler. |

```
manage_resource({ resource: "posts", action: "list",
                  filters: { status_eq: "published" }, sort: "-created_at" })
manage_resource({ resource: "posts", action: "create",
                  attributes: { title: "...", author_id: 42 } })
manage_resource({ resource: "users", action: "get", id: 7 })
```

Add 50 more resources tomorrow — `tools/list` doesn't grow. Only the schemas the model actually asks about enter context, only when it needs them.

### Why this works

1. **Constant-size tool surface.** `tools/list` returns the same 3 entries whether you back 5 resources or 500.
2. **Schemas load lazily.** The model calls `describe_resource(resource: "deals")` right before it needs to act — that schema is in context for one turn, not every turn.
3. **The model handles dispatch.** Picking `action: "create"` vs `action: "update"` is trivial reasoning, not a routing problem that needs separate tools.
4. **Discovery is honest.** `list_resources` describes the whole API in ~1 sentence per resource. The model browses the catalog the same way a human reads a table of contents.

### Server-side shape

A registry maps bare resource names to handler classes. The dispatcher looks up the handler, validates visibility against the caller's auth context, and forwards the remaining args:

```
ResourceRegistry
  ├─ "posts"    → PostsHandler
  ├─ "users"    → UsersHandler
  └─ "comments" → CommentsHandler

manage_resource(resource:, action:, ...)
  → registry.handler_for(resource, auth_context)   # raises ToolNotFound if hidden
  → handler.new(args, auth_context).call           # handler enforces per-action auth
```

**Auth flows through unchanged.** The dispatcher doesn't know about your permission model — handlers do. Same `Ability` / scoping the rest of your app uses.

**Hidden resources return ToolNotFound, not Forbidden.** A user who shouldn't see `admin_users` gets the same error as a typo — they can't enumerate what's behind the curtain.

### Push composition into params, not tools

Same reasoning, one level deeper: don't ship `list_recent_posts`, `list_published_posts`, `top_authors_this_week`, `posts_by_tag`. Ship one `list` action with composable primitives and let the model compose the call:

| Param | Shape | What the model uses it for |
|-------|-------|----------------------------|
| `filters` | `{ field_op: value }` (Ransack-style: `_eq`, `_gt`, `_in`, `_cont`) | Arbitrary WHERE clauses |
| `query` | string | Full-text search |
| `sort` | `"field"` / `"-field"` / `"a,-b"` | Ordering, including multi-key |
| `scope` | string from a per-resource whitelist | Pre-baked named scopes for things filters can't express |
| `fields` | `["id", "title"]` | Sparse responses — model asks only for what it needs |
| `page` / `per_page` | int / int (capped) | Pagination |
| `include` | `["author", "tags"]` | Eager-load related records in one call |

AIs are very good at SQL and REST APIs precisely because those interfaces are heavily parameterized — one `SELECT` statement, one `GET /resource?...`, dozens of knobs. That's the same shape MCP tools should have. Give the model the primitives and let it assemble the call: `filters: { status_eq: "published", created_at_gteq: "2026-01-01" }, sort: "-views", per_page: 10` is exactly the kind of thing the model gets right on the first try. Pre-baking that as `top_published_posts_this_year` just moves the same intent into a name that has to be discovered, documented, and maintained — and falls over the moment the user wants 2025 instead.

The same logic applies to writes: one `update` action that takes `id` + `attributes` beats `rename_X`, `change_X_status`, `assign_X_to_Y`. The model knows which fields to pass.

**Whitelist, don't blacklist.** Filters, sort fields, and scopes should be opt-in per resource — otherwise you're shipping a SQL injection surface. The handler validates `filters.keys` against an allowed list and rejects the rest before touching the DB.

### When to break the pattern

Keep a tool separate when it genuinely doesn't fit the resource model:
- Utilities with no CRUD shape: `send_slack_message`, `reset_password`.
- Privileged backend tools you want hidden from OAuth users entirely.

Filter these out of `tools/list` based on the caller's role so they cost nothing for users who can't call them.

### Anti-patterns

- One tool per verb per model (`create_X`, `update_X`, `delete_X` × every model).
- Eagerly returning every field of every model in every list response — add a `fields` param so the model can ask for sparse responses.
- Putting auth logic in the dispatcher. Push it into the handler so the same code path works from tests, the dashboard, and MCP.
- Verbose schema descriptions repeated across every tool. The meta-tool's schema is loaded once; per-resource schemas come from `describe_resource` on demand.

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
