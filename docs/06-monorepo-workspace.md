# Monorepo Workspace — Scaling Across Repos and Servers

When you have dozens of repositories, you need structure above the project level. This is about the workspace that ties everything together and enables multi-agent, multi-server development.

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

The workspace supports multiple Claude instances running on different servers simultaneously.

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

Individual projects can also be monorepos with multiple services:

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

Multiple runtimes, one repo. Claude knows all of them because CLAUDE.md documents everything. One `/deploy` command can build, test, and deploy all services.

---

## Why This Structure

### For humans
- One clone, run setup, everything is ready
- Consistent tooling across all projects
- Shared scripts and automation

### For AI agents
- **Cross-repo context:** Claude working on the backend can reference patterns from the frontend
- **Infrastructure awareness:** Claude knows the full deploy pipeline across all services
- **Consistent patterns:** Same CLAUDE.md structure, same skill format, same conventions
- **Scalability:** Spin up a new server, run setup, Claude is immediately productive
- **Knowledge transfer:** Skills written for one project inform skill creation in others

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
