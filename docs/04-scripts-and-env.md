# Scripts and .env — Giving Claude Hands

Scripts = how Claude touches infrastructure. Not "go to Jenkins and click build" — `scripts/ci/trigger-build JOB` and done.

---

## The Scripts Directory

Wrap every infrastructure API in a callable script:

```
scripts/
├── lib/
│   └── api_client.rb          # Base class — auth, retries, JSON parsing
├── hosting/                    # Your hosting platform (Coolify, Heroku, AWS, etc.)
│   ├── overview               # List all services
│   ├── deploy                 # Deploy an app
│   ├── manage-envs            # List/set/delete env vars
│   └── app-status             # Start/stop/restart/logs
├── ci/                         # CI/CD (Jenkins, GitHub Actions, etc.)
│   ├── overview               # All jobs & status
│   ├── trigger-build          # Start a build
│   └── build-log              # View output
├── dns/                        # DNS (Cloudflare, Route53, etc.)
│   ├── list-records
│   └── manage-record
├── db/                         # Production database (READ-ONLY)
│   ├── query                  # Run SQL
│   ├── tables                 # List tables + row counts
│   └── inspect                # Schema + sample data
├── monitoring/                 # Queue stats, job monitoring
│   ├── stats
│   ├── busy-jobs
│   └── failed-jobs
├── github/                     # PRs, workflows
│   ├── list-prs
│   └── workflow-runs
└── mail/                       # Email management
    ├── list-accounts
    └── create-account
```

## Design Principles

### 1. Minimal dependencies
Use only your language's standard library. No package manager needed:
- Ruby: `net/http`, `json`, `uri`
- Python: `urllib`, `json`, `os`
- Bash: `curl`, `jq`

Scripts work on any machine with the runtime installed. Zero setup friction.

### 2. Shared base class
All scripts inherit from one base class that handles auth, retries, JSON parsing, .env loading. One base, consistent behavior across all API integrations.

### 3. Executable scripts
Every script is `chmod +x` with a shebang. Claude runs them directly:
```bash
scripts/hosting/overview
scripts/ci/trigger-build "Deploy Production"
scripts/db/query "SELECT count(*) FROM users WHERE created_at > '2024-01-01'"
scripts/dns/manage-record add api.example.com A 1.2.3.4
```

### 4. Read-only where appropriate
Database scripts only have read access. Claude can debug production but can't accidentally drop tables.

---

## .env Files — Secrets Management

### The pattern

```
project/
├── .env              # Actual secrets — GITIGNORED
├── .env.example      # Documents every key — COMMITTED
```

### .env.example documents everything

```bash
# === Infrastructure (required for scripts/) ===

# Hosting platform
HOSTING_API_URL=https://hosting.example.com
HOSTING_API_KEY=
HOSTING_PROJECT_ID=

# CI/CD
CI_HOST=https://ci.example.com
CI_USER=admin
CI_API_TOKEN=

# DNS
DNS_API_TOKEN=
DNS_ZONE_ID=

# Production database (read-only access for debugging)
DATABASE_URL=postgresql://readonly:<password>@host:5432/production

# Container registry
REGISTRY_USER=
REGISTRY_TOKEN=

# Error tracking
SENTRY_PROJECT_ID=
SENTRY_ORG=

# === Application ===
APP_TITLE="My App"
EMAIL_ADDRESS=noreply@example.com
SMTP_HOST=mail.example.com

# === External integrations ===
STRIPE_API_KEY=
OPENAI_API_KEY=
```

### How it works

1. **`.env.example`** is committed — documents every variable with descriptions and grouping
2. **`.env`** is gitignored — actual secrets, only on the machine
3. **Base API class** loads `.env` automatically
4. **Claude never sees actual secrets** — just runs scripts that load them
5. **New server setup:** copy `.env.example` to `.env`, fill in values

### In practice

```
User: "Why is the job queue backing up?"
Claude: *scripts/monitoring/stats + scripts/monitoring/busy-jobs*
Claude: "847 jobs queued. 12 SyncJobs stuck retrying — rate limited. Fix: ..."
```

```
User: "Deploy latest"
Claude: *tests → lint → security scan → scripts/ci/trigger-build → monitor → scripts/hosting/app-status*
Claude: "Deployed. Healthy. 3 migrations ran."
```

No token copy-paste. No dashboard hopping. No SSH. Same scripts a human uses, same guardrails.
