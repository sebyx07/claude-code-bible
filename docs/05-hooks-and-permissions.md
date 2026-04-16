# Hooks, Permissions, and Allow-All Tools

Control what Claude can do automatically, what needs approval, and what quality gates it passes through.

---

## The Case for Allowing All Tools

The single biggest friction in Claude Code is the permission prompt. A typical feature implementation might need 50-100 tool calls (read files, grep, edit, run tests, lint, git add, commit, push, create PR). If Claude stops to ask permission on each one, you're babysitting instead of building.

**The production approach: allow everything safe, gate the dangerous stuff.**

```json
{
  "permissions": {
    "allow": [
      "Bash(bin/*:*)",
      "Bash(npm *:*)",
      "Bash(yarn *:*)",
      "Bash(make *:*)",
      "Bash(scripts/*:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git push:*)",
      "Bash(git fetch:*)",
      "Bash(git checkout:*)",
      "Bash(git rebase:*)",
      "Bash(gh pr create:*)",
      "Bash(gh pr view:*)",
      "Bash(gh api:*)",
      "Bash(docker compose:*)",
      "Bash(curl:*)",
      "Bash(chmod:*)",
      "WebSearch"
    ],
    "defaultMode": "acceptEdits"
  }
}
```

### Why this is safe

- **`scripts/*`** — your scripts have their own safety (read-only DB, no destructive ops)
- **`bin/*`** — your project's own commands (test, lint, build)
- **`git`** — add, commit, push, PR. You review the PR, not every git command
- **`defaultMode: "acceptEdits"`** — Claude edits files freely, you see diffs and approve

### What stays gated

Everything NOT in the allow list still requires approval:
- `rm -rf`, destructive file operations
- Direct `docker` commands (not compose)
- System-level changes
- Unknown bash commands

### The mindset shift

Stop thinking of Claude as untrusted software that needs to ask permission for every action. Think of it as a team member with a pre-approved set of safe operations. You wouldn't ask a senior dev to get approval before running `npm test`. Same energy.

**If you find yourself clicking "approve" 20 times in a row for the same type of command — add it to the allow list.**

---

## Hooks — Automated Quality Gates

Hooks run shell commands before/after Claude's tool calls. They enforce quality at the tool level, not by hoping Claude remembers.

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash(git commit:*)",
        "hooks": [
          {
            "type": "command",
            "command": ".claude/hooks/pre-commit.sh"
          }
        ]
      }
    ]
  }
}
```

### Pre-commit hook example

```bash
#!/bin/bash
set -e

echo "Running linter..."
npm run lint:fix   # or bin/lint, make lint, etc.

# If linter changed files, auto-stage them
if [ -n "$(git diff --name-only)" ]; then
  echo "Linter made changes. Adding to commit..."
  git add -u
fi
```

**Result:** Every commit Claude makes passes through the linter. If linting changes files, they're staged automatically. Claude physically cannot commit unlinted code.

### Hook ideas

| Trigger | Hook | Purpose |
|---------|------|---------|
| `Bash(git commit:*)` | Run linter, auto-stage fixes | Code style enforcement |
| `Bash(git push:*)` | Run full test suite | No broken pushes |
| `Bash(*migrate*)` | Backup schema files | Safety net for migrations |

### Pre vs Post hooks

- **PreToolUse** — runs before the tool. Can block the action if the hook fails
- **PostToolUse** — runs after. Good for notifications or cleanup

---

## The Trust Model

```
┌─────────────────────────────┐
│     Full autonomy           │  Tests, lint, read, grep, edit
│  (pre-allowed, no prompts)  │
├─────────────────────────────┤
│     Quality gates           │  Hooks on commit, push
│  (automated enforcement)    │
├─────────────────────────────┤
│     Approval required       │  Destructive ops, unknown commands
│  (human confirms)           │
└─────────────────────────────┘
```

1. **Tier 1: Full autonomy** — Read files, run tests, lint, grep, git ops. No prompts
2. **Tier 2: Auto with gates** — Commit triggers linter hook. Push could trigger tests
3. **Tier 3: Approval** — Delete files, system commands, production changes

This gives you speed (Claude flows through safe operations) with safety (destructive operations need human eyes).

---

## Settings file location

```
.claude/settings.local.json    # Project-level (committed or gitignored)
~/.claude/settings.json         # User-level (global defaults)
```

Project settings override user settings. Commit the project settings if the whole team uses Claude Code. Gitignore them if it's personal config.
