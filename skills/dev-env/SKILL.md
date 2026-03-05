---
name: dev-env
description: "Use when managing isolated dev environments - creating worktrees, starting/stopping infrastructure, checking status. Triggers on /dev:env command or when user mentions dev environments, worktrees, or feature branch isolation."
---

# Dev Environment Management

Run the CLI script and interpret its output:

```bash
<plugin-dir>/scripts/dev-env.sh <action> [args]
```

Actions: `list`, `create <branch>`, `remove <name>`, `up <name>`, `down <name>`, `status <name>`, `switch <name|number>`

The script reads `.dev-env.yml` from the project root. If missing, tell user to run `/dev:init`.

## When script needs help

- **`remove` returns HAS_WARNINGS=true** - show the warnings to the user and ask for confirmation. If confirmed, run with `FORCE=true` env var.
- **`up`/`down` says "No scripts.up defined"** - the project doesn't have custom scripts. Help the user by running the appropriate commands based on `.dev-env.yml` service definitions.
- **Script fails** - diagnose from error output, suggest fixes.

## Init (interactive - no script)

`/dev:init` requires Claude to interactively detect project type and generate `.dev-env.yml`. Follow the init command instructions.
