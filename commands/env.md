---
description: "Manage dev environments with git worktrees and infrastructure. Usage: /dev:env [list|create|remove|switch|up|down|status] [name]"
---

Run the dev-env CLI script:

```bash
<this-plugin-dir>/scripts/dev-env.sh $ARGUMENTS
```

If $ARGUMENTS is empty, default to `list`.

Interpret the output and present it to the user. If the script reports errors or needs confirmation (HAS_WARNINGS), handle accordingly.
