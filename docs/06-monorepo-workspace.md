# Monorepo Workspace — Scaling Across Repos and Servers

Dozens of repos → need structure above the project level. The workspace ties everything together and enables multi-agent, multi-server development.

---

## The Workspace

```
dev-workspace/
├── CLAUDE.md          # Org-level context (all repos, tech stack, conventions)
├── repos.json         # Manifest of all repos with metadata
├── repos/             # All git repositories
│   ├── backend/
│   ├── frontend/
│   ├── infrastructure/
│   ├── mobile/
│   └── ... (as many as needed)
├── projects/          # Agent workspaces
├── docs/              # Org-wide documentation
└── scripts/
    └── setup.sh       # Bootstrap a new server
```

### repos.json — The Repository Manifest

Catalogue every repo with metadata:

```json
{
  "repos": [
    {
      "name": "backend",
      "category": "core",
      "url": "git@github.com:org/backend.git",
      "clone": true,
      "description": "Main API server"
    },
    {
      "name": "ml-pipeline",
      "category": "ai",
      "url": "git@github.com:org/ml-pipeline.git",
      "clone": false,
      "description": "ML training pipeline"
    }
  ]
}
```

Categories give Claude instant understanding of your org:
- **core** — main products
- **infra** — infrastructure, DevOps
- **libraries** — shared packages, SDKs
- **ai** — ML, agents, AI tooling
- **tools** — internal utilities
- **docs** — documentation

### Workspace CLAUDE.md

Top-level context:
- All repository categories and what they do
- Key tech stack across the org
- Setup commands for new servers
- Relationships between repos ("backend uses data from scraper, frontend calls backend API")

When Claude works inside `repos/backend/`, it reads BOTH:
1. `dev-workspace/CLAUDE.md` — org context
2. `repos/backend/CLAUDE.md` — project specifics

Claude always has the full picture.

---

## Multi-Agent, Multi-Server

Multiple Claude instances across different servers simultaneously.

### Setup a new server

```bash
./scripts/setup.sh              # Clone all repos marked clone:true
./scripts/setup.sh --category core  # Only core repos
```

### Parallel agent work

```
Server 1 (Claude): repos/backend/    — adding payment processing
Server 2 (Claude): repos/frontend/   — redesigning dashboard
Server 3 (Claude): repos/infra/      — updating Kubernetes config
```

Each instance has:
- Its own workspace clone
- Its own `.env` with credentials
- Its own CLAUDE.md context (workspace + project)
- Its own skills and subagents

They work on different repos simultaneously without conflicts. Same org context, different project focus.

---

## The Inner Monorepo

Individual projects can also be monorepos with multiple services (desktop + backend, driver + CLI + UI, mobile + API, etc.):

```
sales-app/
├── api/            # Backend (Python/Node/Ruby/Go)
├── web/            # Frontend (React/Vue/etc.)
├── worker/         # Background jobs
├── agent/          # AI agent server
├── scripts/        # Infrastructure automation
├── infra/          # Dockerfiles, CI pipelines
└── .claude/        # AI configuration (shared across all services)
```

Multiple runtimes, one repo. Claude knows all of them via `CLAUDE.md`. One `/deploy` can build, test, and ship every service.

---

## Why This Structure

**Humans:** one clone, run setup, everything ready. Consistent tooling. Shared automation.

**AI agents:**
- **Cross-repo context** — backend work can reference frontend patterns.
- **Infrastructure awareness** — full deploy pipeline across services.
- **Consistent patterns** — same `CLAUDE.md` structure, same skill format.
- **Scalability** — new server, run setup, immediately productive.
- **Knowledge transfer** — skills from one project inform others.

### Every repo gets the same .claude/ structure

```
any-repo/
├── CLAUDE.md
└── .claude/
    ├── settings.local.json
    ├── skills/
    ├── agents/
    ├── commands/
    └── hooks/
```

Consistency means a developer (human or AI) who knows one repo can navigate all of them.
