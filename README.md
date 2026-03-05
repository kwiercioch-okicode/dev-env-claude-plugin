# Dev Env Plugin for Claude Code

Manage isolated dev environments using git worktrees. Each environment gets its own branch checkout and running infrastructure - containers, dev servers, reverse proxy.

## Installation

```bash
/plugin marketplace add kwiercioch-okicode/dev-env-claude-plugin
/plugin install dev@dev-env
```

## Quick Start

```bash
/dev:init                          # configure for your project (one-time)
/dev:env create feature/my-feat    # create worktree + start infra
/dev:env                           # list all environments with status
/dev:env status my-feat            # detailed status
/dev:env down my-feat              # stop infra (keep worktree)
/dev:env up my-feat                # restart infra
/dev:env remove my-feat            # stop infra + remove worktree
```

## How It Works

1. **`/dev:init`** scans your project and generates `.dev-env.yml` - a config file that describes your dev stack
2. **`/dev:env`** reads that config to manage worktrees and infrastructure

The plugin auto-detects project types:

| Type | What it manages |
|------|----------------|
| **Node.js / Bun** | Dev server on unique port per worktree |
| **Next.js** | Next dev server with env overrides |
| **SST** | `sst dev --stage <name>` per worktree |
| **Docker Compose** | Isolated compose project per worktree |
| **Multi-repo** | Worktrees across multiple repos + Docker + proxy |
| **PHP** | Composer install + Docker/FPM container |

## Config Example

Simple Next.js project:

```yaml
name: my-app

worktrees:
  dir: .worktrees

services:
  app:
    type: dev-server
    command: npm run dev
    env:
      template: .env.local
    ports:
      dev: auto
```

Complex multi-repo with Docker and nginx:

```yaml
name: my-platform

worktrees:
  dir: .worktrees
  repos:
    - path: api
      branch_from: staging
    - path: frontend
      branch_from: staging

services:
  backend:
    type: docker-compose
    file: docker-compose.worktree.yml
    project_prefix: myapp

  frontend:
    type: dev-server
    command: npm run dev
    env:
      template: .env.dev
      overrides:
        API_URL: https://<name>.dev.api.myapp.co
    ports:
      dev: auto
      ssr: auto
    node_modules: symlink

  gateway:
    type: nginx
    template: nginx/worktree.conf.template
    conf_dir: nginx/conf.d
    reload_command: docker exec nginx nginx -s reload

dns:
  type: dnsmasq
  domain: dev.myapp.co
```

## Commands

| Command | Description |
|---------|-------------|
| `/dev:init [path]` | Initialize config for a project |
| `/dev:env` or `/dev:env list` | List environments with infra status |
| `/dev:env create <branch>` | Create worktree + start infrastructure |
| `/dev:env remove <name>` | Stop infra + remove worktree (with safety checks) |
| `/dev:env switch <name\|number>` | Switch main checkout to a branch |
| `/dev:env up <name>` | Start infrastructure for existing worktree |
| `/dev:env down <name>` | Stop infrastructure (keep worktree) |
| `/dev:env status <name>` | Detailed status of one environment |

## Custom Scripts

If your project has complex setup needs, point to custom scripts:

```yaml
scripts:
  up: ./scripts/worktree-up.sh
  down: ./scripts/worktree-down.sh
  status: ./scripts/worktree-status.sh
```

The plugin will call these with the worktree name as the first argument.

## License

MIT
