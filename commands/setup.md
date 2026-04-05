---
description: Check OpenCode installation and provider configuration status
---

# OpenCode Setup Check

Run the following checks and report results clearly:

## Step 1 — Check installation

Run: `opencode --version`

- If it succeeds: report the version number. OpenCode is installed.
- If it fails (command not found): report that OpenCode is not installed and provide these install options:
  ```
  npm install -g opencode-ai
  ```
  Or visit opencode.ai for alternative install methods. Then tell the user to run `/opencode:setup` again after installing.

## Step 2 — Check provider configuration

Run: `opencode providers 2>&1 | head -30`

- If the output lists at least one configured provider: report it as ready.
- If no providers are shown or the command shows an empty list: tell the user to configure a provider by running `opencode auth` or by setting an environment variable such as `ANTHROPIC_API_KEY` or `OPENAI_API_KEY`.

## Report Format

Present results in this format:

```
OpenCode Setup Status
─────────────────────
Installation:  ✓ v0.x.x  (or ✗ Not installed)
Provider:      ✓ [provider name]  (or ✗ No provider configured)

Status: Ready  (or: Action required — see above)
```

If everything is ready, confirm with: "OpenCode is ready. Use `/opencode:run [task]` to delegate tasks."
