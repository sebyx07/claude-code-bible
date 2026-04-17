# Checklist — Set Up Your Project

Start small, expand as you find yourself repeating instructions.

---

## Phase 1: Foundation (Day 1)

```
[ ] CLAUDE.md at project root
    - Project structure (directory tree with purpose of each folder)
    - Tech stack with versions
    - All runnable commands (test, lint, build, deploy — exact CLI)
    - Architecture patterns (where business logic goes, file size limits)
    - Response rules (no preamble, lead with action, disagree when wrong)

[ ] Permissions — recommended: `claude --dangerously-skip-permissions`
    - Zero prompts. Maximum autonomy. What most daily users actually run.
    - Alternative: .claude/settings.local.json with allow-list + defaultMode: "acceptEdits"
    - Goal: Claude never pauses on safe operations

[ ] .env.example (committed)
    - Document every env var with descriptions
    - Group by purpose (infra, app, external services)
    - .env itself is gitignored — only on the machine
```

## Phase 2: Quality Gates (Week 1)

```
[ ] Single lint command that auto-fixes everything
    - bin/lint, make lint, npm run lint:fix — one command
    - Claude runs this, not individual formatters

[ ] Pre-commit hook
    - .claude/hooks/pre-commit.sh → runs linter before every commit
    - Wire in settings.local.json hooks.PreToolUse
    - Claude physically cannot commit unlinted code

[ ] Enforce small files
    - Set a LOC limit in CLAUDE.md (recommended: 500)
    - Document the split pattern (nested modules/namespaces)
    - "When a file exceeds 500 LOC, split by concern"

[ ] Testing commands documented
    - Exact commands in CLAUDE.md for every test type
    - Claude runs tests after every change — make this frictionless
```

## Phase 3: Skills (Week 2-3)

```
[ ] 2-3 core skills
    - Development skill (your stack's patterns and conventions)
    - Deploy skill (how to deploy, verify, rollback)
    - One domain skill (whatever your app's core domain is)

[ ] Skill structure
    - .claude/skills/[name]/SKILL.md with frontmatter
    - allowed-tools to restrict scope where needed
    - Supporting .md files for reference material

[ ] /plan and /implement commands
    - /plan: explore codebase → create implementation plan
    - /implement: execute plan step by step
    - Store plans in .plans/ or .debugging/plans/
```

## Phase 4: Infrastructure Scripts (Week 3-4)

```
[ ] scripts/ directory with shared base class
    - Common auth, retries, error handling, .env loading
    - All scripts executable (chmod +x, shebangs)

[ ] Wrap your infrastructure APIs
    - Hosting (Coolify, Heroku, AWS, Vercel, etc.)
    - CI/CD (Jenkins, GitHub Actions, etc.)
    - DNS (Cloudflare, Route53, etc.)
    - Database (read-only production access)
    - Monitoring (queues, errors, health)

[ ] Add scripts to permission allow list
    - "Bash(scripts/*:*)" in settings.local.json
    - Claude calls them directly, no approval needed
```

## Phase 5: Subagents and MCP (Month 2)

```
[ ] Specialized subagents
    - .claude/agents/backend-expert.md (your stack)
    - .claude/agents/infra-expert.md (DevOps)
    - Assign skills and model preferences

[ ] MCP servers
    - Issue tracking (Linear, Jira)
    - Browser testing (Playwright)
    - Error monitoring (Sentry, Bugsnag)

[ ] Seed memory system
    - Save your role and preferences (user memories)
    - Save confirmed approaches (feedback memories)
    - Save external system references (reference memories)
```

## Phase 6: Full Workflow (Month 2-3)

```
[ ] Complete command set
    - /qa — review changes, check regressions
    - /fix-ci — extract and fix CI failures
    - /update-claude — keep CLAUDE.md current

[ ] Iterate on everything
    - Explain something twice? → make it a skill
    - Claude asks the same question? → add to CLAUDE.md
    - Approve the same command type 5 times? → add to allow list
    - Claude makes the same mistake? → save a feedback memory

[ ] Monorepo workspace (if multi-repo)
    - Workspace CLAUDE.md with org context
    - repos.json manifest
    - setup.sh for bootstrapping new servers
```

---

## Quick Wins (Do These First)

The 4 things that give you 80% of the value in under an hour:

1. **`CLAUDE.md` with commands** — document test, lint, deploy. 3× more useful immediately.
2. **Skip the prompts** — `claude --dangerously-skip-permissions`. Zero friction. What most daily users run.
3. **Response rules** — "No preamble, lead with action, disagree when wrong." Changes Claude's personality.
4. **One lint command** — `bin/lint` auto-fixes everything. Wire as pre-commit hook.

---

## Anti-Patterns

- **CLAUDE.md over 600 lines** — Claude reads it every turn. Trim ruthlessly
- **Duplicating code in CLAUDE.md** — reference patterns, don't paste entire files
- **Skills for everything** — only for repeated, complex workflows
- **No tests** — Claude's biggest advantage is verifying its own work. No tests = flying blind
- **Secrets in code** — always .env + .env.example. Claude should never see credentials in source
- **Ignoring memory** — correct Claude when it's wrong, confirm when it's right. Memory makes every future session better
- **Too many approval gates** — if you're clicking approve 20 times in a row, you're doing it wrong

---

## The ROI

| Task | Without structure | With structure |
|------|------------------|----------------|
| Add a feature | Claude asks 10 questions about patterns | Follows conventions from CLAUDE.md + skills |
| Fix a bug | Reads random files hoping to find it | Greps small, focused service files |
| Deploy | "I can't deploy, please do it manually" | Runs scripts, monitors, verifies |
| Code review | Generic feedback | Checks against your documented patterns |
| After correction | Forgets next session | Saves to memory, applies permanently |

The upfront investment is real — CLAUDE.md takes a few hours, first skills take a day. But the compounding returns are massive. Every skill, every script, every memory makes every future session faster and more autonomous. After a month, Claude feels less like a tool and more like a team member who knows your codebase.
