# Code Quality — Why AI Needs This More Than Humans Do

Clean code isn't a luxury when working with AI. When Claude navigates your codebase, it reads files, greps for patterns, and builds mental models from what it finds. The cleaner your code, the faster and more accurately Claude works. Messy code wastes tokens and causes mistakes.

---

## SOLID Principles — Strictly Enforced

Document SOLID rules in CLAUDE.md as rules, not guidelines.

### Single Responsibility (SRP) — The Most Important One

Every class/module does ONE thing. This is critical for AI because:
- Claude finds the right file faster (grep for `UserCreator` vs digging through a 500-line `UserService`)
- Changes are isolated — Claude modifies one file without breaking ten others
- Tests are focused — one test file per module, easy to run in isolation

**Don't:**
```
class UserManager:
    def create(params): ...
    def update(params): ...
    def destroy(id): ...
    def send_notification(user): ...
    def generate_report(user): ...
```

**Do:**
```
src/services/user/creator.py      # 40 lines
src/services/user/updater.py      # 35 lines
src/services/user/destroyer.py    # 20 lines
src/services/user/notifier.py     # 50 lines
src/services/user/reporter.py     # 60 lines
```

**Why this matters for AI:** When you say "fix the user notification bug," Claude greps for `user/notifier`, reads one 50-line file, fixes it, runs its test. With a monolith `UserManager`, Claude reads 500 lines, understands all five responsibilities, and risks breaking create/update while fixing notify.

### Open/Closed Principle (OCP)

New behavior = new class, not new conditionals in existing code. Claude handles this naturally — creating a new file is safer than modifying existing complex logic.

### Dependency Inversion (DIP)

Inject dependencies in constructors. Makes testing trivial and Claude can swap implementations without hunting through method bodies:

```python
class OrderCreator:
    def __init__(self, payment_processor=None, notifier=None):
        self.payment = payment_processor or PaymentProcessor()
        self.notifier = notifier or OrderNotifier()
```

---

## Small Files — 500 LOC Max

**Rule: Keep all files under 500 lines of code.** Split into nested modules when needed.

This isn't arbitrary. When Claude reads a file, it uses context tokens. A 200-line file = efficient, understood in one pass. A 2000-line file = Claude reads in chunks, loses context, makes mistakes.

**When to split:**
- Multiple private methods doing distinct work
- Complex validations or formatting logic
- Test file covering many scenarios

**Pattern:**
```
# Before: one big file
src/services/email/sender.py  (400 lines)

# After: split by concern
src/services/email/sender.py          (80 lines — orchestrator)
src/services/email/formatter.py       (60 lines)
src/services/email/validator.py       (50 lines)
src/services/email/deliverer.py       (70 lines)

# Tests mirror the structure
tests/services/email/test_sender.py
tests/services/email/test_formatter.py
tests/services/email/test_validator.py
```

**Why this matters for AI:** Claude reads, understands, and modifies a 60-line formatter in one shot. No need to hold 400 lines in context to change how email subjects are formatted.

---

## Service-Oriented Architecture

All business logic in a services layer. No exceptions. This applies to any framework.

```
src/services/
├── payment/
│   ├── processor.py
│   ├── refunder.py
│   └── errors.py
├── invoice/
│   ├── generator.py
│   ├── renderer.py
│   └── sender.py
└── user/
    ├── creator.py
    ├── authenticator.py
    └── notifier.py
```

**Layers are thin:**
- **Controllers/Handlers** — HTTP only: parse request, call service, send response
- **Models/Entities** — Persistence only: validations, associations, queries
- **Jobs/Workers** — Thin wrappers: call a service
- **Services** — ALL business logic

**Why this matters for AI:** Predictable structure. Claude knows where to look. "Add invoice generation" → Claude creates `src/services/invoice/generator.py`. No guessing if it goes in the controller, model, utility, or helper.

---

## Linting — Automated, Non-Negotiable

Linting runs automatically via hooks (see [05-hooks-and-permissions.md](05-hooks-and-permissions.md)). Claude literally cannot commit unlinted code.

```bash
# Whatever your stack uses
npm run lint:fix        # ESLint/Prettier
ruff check --fix .      # Python
rubocop -A              # Ruby
gofmt -w .              # Go
cargo fmt               # Rust
```

**Why this matters for AI:**
- Claude doesn't waste time on formatting decisions
- Code style is 100% consistent — Claude's pattern matching works better on consistent code
- No "fix the style" back-and-forth
- Wrap it in a single command: `bin/lint` or `make lint`

---

## Testing — Always

Test-first or test-alongside, but always test. This is non-negotiable for AI-first development.

### Why testing matters for AI

1. **Confidence:** Claude refactors aggressively because tests catch regressions
2. **Specification:** Tests document expected behavior — Claude reads them to understand what code should do
3. **Verification:** After every change, Claude runs tests to confirm nothing broke
4. **TDD workflow:** Claude writes the test first, sees it fail, implements until it passes

### Test commands in CLAUDE.md

```bash
npm test                              # All unit tests
npm run test:e2e                      # E2E tests
npm test -- --testPathPattern=user    # Test one module
npm test -- --watch                   # Watch mode
```

Document these explicitly. Claude needs exact commands, not "run the tests."

---

## Custom Error Classes

Never throw generic exceptions. Always domain-specific:

```python
# src/services/payment/errors.py
class PaymentError(Exception): pass
class InsufficientFundsError(PaymentError): pass
class GatewayTimeoutError(PaymentError): pass
class FraudDetectedError(PaymentError): pass
```

**Why this matters for AI:** Claude writes precise error handling. `except InsufficientFundsError` is self-documenting. `except Exception` tells Claude nothing about what can go wrong.

---

## The Compound Effect

| Practice | AI benefit |
|----------|-----------|
| SRP / small files | Claude finds + reads the right code in one shot |
| Service objects | Predictable location for all business logic |
| Strict patterns | Claude follows architecture, doesn't reinvent it |
| Automated linting | Zero time on formatting |
| Comprehensive tests | Claude verifies its own work instantly |
| Custom errors | Self-documenting error flows |
| Documented commands | Claude knows exactly how to test, lint, deploy |

Each alone is good engineering. Together, they create an environment where AI produces code that looks like a senior engineer on your team wrote it — because it's following the same patterns, running the same tests, going through the same quality gates.
