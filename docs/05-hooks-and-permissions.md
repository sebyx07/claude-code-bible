# Hooks, Permissions, and Allow-All Tools

Control what Claude can do automatically, what needs approval, and what quality gates it passes through.

---

## The Case for Allowing All Tools

Biggest friction in Claude Code = permission prompts. A feature implementation needs 50-100 tool calls. Stopping for each = babysitting, not building.

**Recommended: just bypass everything.**

```bash
claude --dangerously-skip-permissions
# or
claude --permission-mode bypassPermissions
```

Zero prompts. This is how most daily Claude users actually run it — the ignore-sandbox-only warning in `--help` is overcautious. After a year of daily use across production work, it's fine. Claude respects your CLAUDE.md, tests run, hooks still fire. You still review commits and PRs.

**Alternative: curated allow list** (below). Use if you want granular gating or share the config with teammates who prefer it.

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

### Mindset

Claude = team member with pre-approved safe ops. Not untrusted software pinging for every action. You wouldn't gate a senior dev on `npm test`.

**Clicking approve 20 times for the same command type → stop. Use `--dangerously-skip-permissions`. The prompts aren't protecting you from anything a code review wouldn't catch.**

---

## Hooks — Automated Quality Gates

Shell commands that run before/after Claude's tool calls. Enforce quality at the tool level — not by hoping Claude remembers.

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

1. **Tier 1 — autonomy:** read, test, lint, grep, git. No prompts.
2. **Tier 2 — auto + gates:** commit fires linter hook, push fires tests.
3. **Tier 3 — approval:** destructive ops, system commands, production changes.

Speed on safe ops. Human eyes on destructive ones.

---

## Settings file location

```
.claude/settings.local.json    # Project-level (committed or gitignored)
~/.claude/settings.json         # User-level (global defaults)
```

Project settings override user settings. Commit the project settings if the whole team uses Claude Code. Gitignore them if it's personal config.
