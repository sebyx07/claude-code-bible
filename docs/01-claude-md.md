# CLAUDE.md — The Project Brain

The single most important file. Claude reads it on every conversation start. Think of it as onboarding docs for a brilliant engineer who has amnesia every morning.

---

## What goes in it

1. **Project structure** — directory tree with the purpose of each folder
2. **Tech stack** — with versions
3. **Architecture decisions** — "business logic goes in services, never controllers"
4. **All runnable commands** — exact CLI, not descriptions
5. **Auth model & roles** — who can access what
6. **Database conventions** — ID types, naming, patterns
7. **Testing strategy** — frameworks, commands, what to test
8. **Infrastructure** — hosting, CI/CD, monitoring, deploy flow

## What makes it good

### Actionable, not aspirational

Don't write "we value clean code." Write:
```
Business logic belongs in src/services/ following Domain::Action pattern.
Controllers are thin — delegate to services. Models handle persistence only.
Keep all files under 500 LOC. Split into nested modules when needed.
```

### Exact commands

Claude can't guess your CLI:
```bash
# Testing
npm test                          # Unit tests
npm run test:e2e                  # End-to-end tests
npm run test -- --watch           # Watch mode

# Quality
npm run lint                      # Auto-fix linting
npm run typecheck                 # TypeScript check

# Infrastructure
scripts/deploy.sh production      # Deploy to prod
scripts/db/query "SELECT ..."     # Read-only prod SQL

# Development
docker compose up -d              # Start services
make migrate                      # Run migrations
```

### Document the WHY, not just the WHAT

```
## N+1 Prevention
NEVER write manual eager loading. The ORM plugin handles it automatically.
Just write User.findAll() — the plugin detects association access at runtime.
```

```
## Error Handling
Always use domain-specific error classes, never generic Error/Exception.
Custom errors give better debugging, logging, and catch specificity.
```

### Keep it under 600 lines

Claude reads it every conversation. Bloat = wasted context tokens. A well-structured 500-line CLAUDE.md covers everything for a complex production app.

---

## Layered CLAUDE.md

You can have CLAUDE.md at multiple levels:

```
workspace/
├── CLAUDE.md              # Org-level (tech stack, repo categories, conventions)
└── repos/
    ├── backend/
    │   └── CLAUDE.md      # Project-level (architecture, commands, patterns)
    └── frontend/
        └── CLAUDE.md      # Different project, different context
```

Claude loads all of them, root to project. The workspace-level gives org context; the project one gives specifics. Claude always has the full picture.

---

## Response rules

Put behavioral instructions at the top of CLAUDE.md. This shapes how Claude interacts with you across the entire session:

```markdown
## Response Rules
- Execute immediately. No preamble, no "I'll start by...", no restating the task.
- Lead with action or answer. Reasoning after, only if non-obvious.
- Be proactive: if you see bugs or issues in files you're modifying, fix them.
- Batch independent tool calls in parallel. Read multiple files at once.
- Never speculate about code you haven't read. Read first, then answer.
- Disagree when the user is wrong. State the correction directly.
```

This alone transforms Claude from a chatbot into an executor. No fluff, no narration, just code.

---

## Self-updating

Create a `/update-claude` command so Claude keeps CLAUDE.md current when features are added. Claude updates its own documentation — the brain maintains itself.
