# Skills, Subagents, and Commands

Three layers of reusable AI capability that compound over time.

---

## Skills — Domain Expertise Packages

Skills are `.claude/skills/*/SKILL.md` files. Think of them as "expert hats" Claude puts on — activated automatically or via `/skill-name`.

### Real production examples (18 skills)

| Category | Skills |
|----------|--------|
| **Development** | rails-development, database-design, performance-optimization, writing-react-components, ui-styling |
| **Infrastructure** | deploy, coolify-api, jenkins, cloudflare-dns, ovh, mail |
| **Quality** | debug-coderabbit, debug-prod, migration-management |
| **Tooling** | use-browser (Playwright), using-linear, rails-secrets |
| **Meta** | creating-skills (Claude creates new skills!) |

### Skill structure

**Single-file** (simple instructions):
```
deploy/
└── SKILL.md
```

**Multi-file** (instructions + reference material):
```
development/
├── SKILL.md          # Main instructions
├── patterns.md       # Architecture patterns
├── testing.md        # Testing conventions
└── examples.md       # Code examples
```

### SKILL.md anatomy

```yaml
---
name: deploy
description: End-to-end deploy — test, build, deploy, verify
allowed-tools:
  - Bash
  - Read
---

# Deploy

1. Pre-deploy: run tests + lint + security scan
2. Trigger CI build
3. Monitor build log
4. Verify deployment health
5. Rollback if needed

## Commands
scripts/ci/trigger-build JOB
scripts/hosting/app-status APP
scripts/hosting/rollback APP
```

**Key fields:**
- **`description`** — triggers automatic activation. Make it keyword-rich so Claude picks the right skill for the task
- **`allowed-tools`** — restricts what Claude can do in that context. A `debug-prod` skill gets only `Read` + `Bash` (read-only). A `deploy` skill gets `Bash` + `Read`
- **Supporting `.md` files** — reference material loaded on activation. Split large skills into focused docs

### Skill design tips

- One skill per domain, not one skill per task
- `description` is the trigger — pack it with keywords Claude would match
- Use `allowed-tools` to enforce safety (read-only debugging, no writes during review)
- Supporting files let you add depth without bloating SKILL.md

---

## Subagents — Specialized Workers

Subagents are pre-configured Claude instances with specific roles. They run in parallel for complex tasks.

```
.claude/agents/
├── backend-expert.md     # Your stack's specialist
└── infra-expert.md       # Docker, CI/CD, hosting, DNS
```

### Agent definition

```yaml
---
name: backend-expert
description: "Backend expert. Models, services, controllers, tests, database."
model: opus
skills:
  - development
  - database-design
  - performance-optimization
---

You are the backend specialist for this project.

## Principles
1. SOLID enforced — SRP, OCP, DIP
2. Custom errors only — never generic exceptions
3. Testing required — write tests for everything
4. Performance — memoize, optimize queries, use background jobs
```

### Why subagents matter

- **Parallelism:** Main Claude delegates to `backend-expert` for code while `infra-expert` handles deploy — simultaneously
- **Isolation:** Each agent has its own tool restrictions and skill context
- **Model control:** Run subagents on cheaper/faster models for simple tasks, opus for complex ones
- **Worktrees:** Agents work in isolated git worktrees — no conflicts

---

## Commands — Slash-Invocable Workflows

Commands are workflows triggered by `/command-name`. They orchestrate multi-step processes.

### Production command set

```
.claude/commands/
├── plan.md              # /plan — explore codebase, create implementation plan
├── implement.md         # /implement — execute a plan step by step
├── qa.md                # /qa — review changes, check for regressions
├── fix-ci.md            # /fix-ci — extract CI errors, fix them
├── fix-coderabbit.md    # /fix-coderabbit — address PR review comments
├── update-claude.md     # /update-claude — update CLAUDE.md
└── initial-idea.md      # /initial-idea — generate spec from idea
```

### The workflow

```
/plan add Stripe payment processing
  → Claude explores codebase thoroughly
  → Creates .plans/stripe-payments/plan.md
  → Lists files to create/modify, testing strategy, edge cases

/implement
  → Reads the plan, executes step by step
  → Writes code + tests, runs test suite

/qa
  → Reviews all changes for regressions
  → Runs full test suite + linter + security scan

/fix-ci  (if CI fails)
  → Pulls CI logs, identifies failures, fixes them

/deploy
  → Triggers build, monitors, verifies health
```

### Command frontmatter

```yaml
---
description: Create a structured implementation plan
argument-hint: [what you want to do]
allowed-tools: Write, Read, Glob, Grep, Bash, Skill
---
```

Commands can invoke skills and spawn subagents internally. A `/plan` command can activate the `development` skill for pattern discovery and spawn an `Explore` agent for thorough codebase analysis.
