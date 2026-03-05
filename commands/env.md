---
description: "Manage dev environments with git worktrees and infrastructure. Usage: /dev:env [list|create|remove|switch|up|down|status] [name]"
---

Invoke the dev:dev-env skill to manage dev environments.

<UserRequest>
  $ARGUMENTS
</UserRequest>

Read `.dev-env.yml` from the project root to determine how to handle this project. If the config doesn't exist, tell the user to run `/dev:init` first.

Parse $ARGUMENTS to determine the action (default: `list`) and execute according to the skill instructions.
