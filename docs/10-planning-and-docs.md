# Planning, Docs, and Fast Feedback Loops

Architecture, planning, documentation, speed. Separates "AI helps me write code" from "AI builds my product."

---

## Architecture — The Foundation

Biggest single multiplier with AI. Not optional.

### Why it matters more with AI

Claude reads your codebase to understand it. Good architecture:
- **Predictable structure** → Claude finds things in one grep, not ten
- **Small focused files** → Claude reads and modifies in one pass, no chunking
- **Clear boundaries** → Claude knows where new code belongs without asking
- **Consistent patterns** → Claude follows conventions, doesn't invent new ones

With bad architecture:
- 2000-line God classes → Claude reads half, loses context, makes mistakes
- Logic scattered everywhere → Claude modifies the wrong layer
- No patterns → Claude asks you 5 questions before writing one line
- Inconsistent code → Claude generates inconsistent code

### Proven patterns

| Pattern | What it means | AI benefit |
|---------|--------------|-----------|
| **Service objects** | All business logic in `services/` | Claude knows exactly where to add/find logic |
| **SRP** | One class = one responsibility | 50-line files, not 500-line files |
| **500 LOC max** | Split into nested modules | Claude reads entire file in one shot |
| **Thin layers** | Controllers/models/jobs are thin wrappers | Claude only needs to touch one file per change |
| **Domain namespacing** | `Payment::Processor`, `User::Creator` | Grep finds exactly the right file |
| **Custom errors** | Domain-specific, never generic | Self-documenting error handling |
| **Dependency injection** | Constructor injection | Testable, swappable, clear dependencies |

---

## Planning — Before You Code

`/plan` exists for a reason. Planning before implementation is dramatically more effective with AI.

### The planning workflow

```
/plan add Stripe payment processing

Claude:
1. Explores the codebase thoroughly (grep, glob, read)
2. Identifies existing patterns to follow
3. Maps dependencies and affected files
4. Creates a structured plan:

.plans/stripe-payments/
├── plan.md          # Implementation steps, file list, testing strategy
└── context.md       # Relevant code snippets, patterns found
```

### Why planning matters

Without a plan: Claude codes immediately, hits problems halfway (wrong pattern, missing dep, conflict), backtracks, wastes tokens, produces inconsistent code.

With a plan:
- Explores first, codes second.
- Architecture decisions captured before implementation.
- Review plan in 2 min vs 20 files of code.
- Catch wrong approach before any code exists.

### Plan structure (concise!)

```markdown
# Stripe Payment Processing

## Files to create
- src/services/payment/stripe_processor.py
- src/services/payment/webhook_handler.py
- src/services/payment/errors.py
- tests/services/payment/test_stripe_processor.py

## Files to modify
- src/routes/api.py — add /payments endpoint
- src/config.py — add STRIPE_API_KEY

## Pattern: follows src/services/invoice/generator.py

## Steps
1. Create error classes
2. Create StripeProcessor service
3. Create webhook handler
4. Add routes
5. Write tests
6. Run full suite
```

30 lines. No fluff. All signal. Claude can execute this without ambiguity.

---

## Docs Directory — Organizational Knowledge

```
project/
├── docs/
│   ├── architecture.md      # System design decisions
│   ├── api.md               # API documentation
│   ├── onboarding.md        # New developer setup
│   └── decisions/           # ADRs (Architecture Decision Records)
│       ├── 001-use-postgres.md
│       └── 002-service-pattern.md
```

### What goes in docs/ vs CLAUDE.md

| `CLAUDE.md` | `docs/` |
|-------------|---------|
| Commands Claude runs | Architecture decisions and WHY |
| Patterns Claude follows | API documentation |
| Tech stack versions | Onboarding guides |
| Testing strategy | Design documents |
| Infrastructure overview | ADRs |

**CLAUDE.md is for execution.** What Claude needs to write code RIGHT NOW.
**docs/ is for context.** What Claude needs to understand WHY things are the way they are.

Claude reads CLAUDE.md every conversation. It reads docs/ only when relevant (via skills or grep). This distinction keeps CLAUDE.md lean.

---

## Concise Documentation — The Golden Rule

**Every markdown file Claude reads costs tokens. Write like you're paying per word — because you are.** See [ch. 11](11-compressed-config.md) for the detailed rules and templates.

### Bad (fluff)

```markdown
# User Authentication Service

## Overview

The User Authentication Service is responsible for handling all authentication-related
operations within our application. This includes user login, registration, password
reset, and session management. The service follows our established patterns for
service objects and implements the Single Responsibility Principle by focusing
exclusively on authentication concerns.

## Getting Started

To use this service, you'll first want to understand how our dependency injection
system works. We use constructor injection to provide all dependencies...
```

### Good (signal)

```markdown
# User Auth

Login, register, password reset, session management.
Pattern: `services/auth/authenticator.py`, `services/auth/registrar.py`
Tests: `tests/services/auth/`
Deps: injected via constructor (see service base class)
```

**Same information. 80% fewer tokens.** Claude doesn't need prose. It needs structure, keywords, and file paths.

### Documentation rules

1. **Lead with the answer, not the question** — state what it is, then explain if needed
2. **File paths over descriptions** — "see `src/services/auth/`" beats a paragraph explaining the auth layer
3. **Tables over paragraphs** — structured data is easier to parse
4. **No filler words** — cut "In order to", "It's important to note that", "As mentioned above"
5. **Keep SKILL.md under 100 lines** — split reference material into supporting files
6. **Keep CLAUDE.md under 600 lines** — if it's longer, you're documenting too much

---

## Fast Feedback Loops — Speed Matters

Claude's effectiveness ∝ iteration speed. Slow tests = slow Claude. Slow builds = slow Claude.

### Local speed

| What | Target | Why |
|------|--------|-----|
| Unit tests | < 10 seconds | Claude runs these after every change |
| Lint | < 5 seconds | Runs on every commit via hook |
| Type check | < 15 seconds | Catches errors before tests |
| Dev server start | < 5 seconds | Claude needs to test UI changes |
| Single test file | < 3 seconds | Claude runs focused tests constantly |

**Optimize for the loop:** edit → test → edit → test. One test file: 30s × 20 iterations = 10 min waiting. 2s × 20 = 40s. Compounds hard.

### CI/CD speed

| What | Target | Why |
|------|--------|-----|
| CI pipeline | < 5 minutes | Claude waits for this before deploying |
| Docker build | < 3 minutes | Layered caching, minimal rebuilds |
| Deploy | < 2 minutes | Health check to production |
| Full cycle (push → live) | < 10 minutes | Iteration speed on production |

### How to get there

- **Parallel test execution** — split tests across workers
- **Docker layer caching** — don't rebuild what hasn't changed
- **Incremental builds** — esbuild, turbo, swc over webpack
- **Test database in memory** — SQLite for unit tests, real DB for integration
- **Hot reload** — dev server watches files, no manual restart
- **Pre-built base images** — don't install system deps on every build

### The fast loop in practice

```
Claude edits src/services/payment/processor.py
  → runs test (2 seconds) → passes
  → edits tests to add edge case
  → runs test (2 seconds) → passes
  → commits (lint hook runs, 3 seconds)
  → pushes
  → CI runs (4 minutes)
  → deploys (2 minutes)
  → verifies health check
Total: ~7 minutes from code to production
```

Slow setup: 30s tests, 15min CI, 10min deploy = 35+ min of mostly waiting. Claude idle, you idle. Everyone loses.

---

## Putting It All Together

```
Good architecture     → Claude writes correct code on first try
Good planning         → Claude doesn't backtrack or waste effort
Concise docs          → Claude reads fast, retains more
Fast feedback loops   → Claude iterates 10x more in the same time
```

The four multiply. Good architecture + slow tests = wasted. Fast tests + bad architecture = catches wrong things. Planning without concise docs = bloated plans. All four together = magic.
