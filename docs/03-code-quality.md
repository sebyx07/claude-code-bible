# Code Quality — Why AI Needs This More Than Humans Do

AI reads code to work on it. Clean code → fast, accurate. Messy code → wasted tokens, mistakes.

**The meta-rule:** everything in this chapter earns its keep by making files smaller, patterns more predictable, and changes more isolated. Small focused units = Claude finds, reads, modifies in one shot.

---

## SOLID Principles — Strictly Enforced

Document SOLID rules in `CLAUDE.md` as rules, not guidelines.

### Single Responsibility (SRP) — the most important

One class/module = one thing. Smaller files, isolated changes, focused tests.

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

Fixing `user/notifier`: grep → read 50 lines → patch → test. With a monolith `UserManager`: read 500 lines → risk breaking create/update while fixing notify.

### Open/Closed (OCP)

New behavior = new class, not new conditionals in existing code.

### Dependency Inversion (DIP)

Constructor injection. Trivial to test, trivial to swap:

```python
class OrderCreator:
    def __init__(self, payment_processor=None, notifier=None):
        self.payment = payment_processor or PaymentProcessor()
        self.notifier = notifier or OrderNotifier()
```

---

## Small Files — 500 LOC Max

**Rule: all files ≤500 LOC.** Split into nested modules when exceeded.

200-line file = one read, full context. 2000-line file = chunked reads, lost context, mistakes.

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

---

## Service-Oriented Architecture

All business logic in a services layer. No exceptions. Applies to any framework.

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
- **Controllers/Handlers** — HTTP only: parse, call service, render.
- **Models/Entities** — Persistence only: validations, associations, queries.
- **Jobs/Workers** — Thin wrappers: call a service.
- **Services** — ALL business logic.

Predictable structure → Claude knows exactly where new code belongs. "Add invoice generation" → `src/services/invoice/generator.py`. No guessing.

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

Wrap in one command: `bin/lint` or `make lint`. No formatting decisions for Claude, no back-and-forth.

---

## Testing — Always

Non-negotiable for AI-first development.

- **Confidence:** tests catch Claude's regressions from aggressive refactors.
- **Specification:** tests document expected behavior — Claude reads them.
- **Verification:** run after every change.
- **TDD:** Claude writes test first, sees fail, implements until pass.

### Test commands in `CLAUDE.md`

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

`except InsufficientFundsError` is self-documenting. `except Exception` tells Claude nothing.

---

## The Compound Effect

| Practice | AI benefit |
|----------|-----------|
| SRP / small files | One-shot read, modify, test |
| Service objects | Predictable location for logic |
| Strict patterns | Follows, doesn't reinvent |
| Automated linting | Zero formatting decisions |
| Comprehensive tests | Self-verifies its own work |
| Custom errors | Self-documenting error flows |
| Documented commands | Exact CLI, no guessing |

Each alone = good engineering. Together = AI produces code indistinguishable from a senior engineer on your team — same patterns, same tests, same gates.
