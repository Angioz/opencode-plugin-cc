---
description: Interactive setup wizard — authenticate a provider and choose a model for OpenCode. Run this before your first /opencode:run.
---

# OpenCode Setup Wizard

Guide the user step by step. Do NOT skip steps or assume answers.

---

## Step 0 — Detect shell environment

Run this first to know which commands to use for the rest of the wizard:

```bash
uname -s 2>/dev/null || echo "Windows-PowerShell"
```

Classify the result:

| Output | Shell type |
|--------|------------|
| `Linux`, `Darwin`, `MINGW*`, `MSYS*`, `CYGWIN*` | **bash** (use bash commands below) |
| `Windows-PowerShell` or command fails | **PowerShell** (use PowerShell commands below) |

Store as `SHELL_TYPE` (either `bash` or `powershell`). Use this throughout the wizard.

> **PowerShell note:** The `bin/` scripts in this plugin are POSIX shell scripts. On PowerShell, they require Git Bash or WSL to run. If you are on Windows with PowerShell, we recommend running Claude Code from Git Bash for full compatibility.

---

## Step 1 — Check if already configured

**bash:**
```bash
test -s ~/.opencode-plugin/config.sh && cat ~/.opencode-plugin/config.sh || echo "MISSING"
```

**PowerShell:**
```powershell
if (Test-Path "$env:USERPROFILE\.opencode-plugin\config.ps1") { Get-Content "$env:USERPROFILE\.opencode-plugin\config.ps1" } else { "MISSING" }
```

If config exists: show the current model line, then use the **AskUserQuestion tool** with:
- question: "OpenCode is already configured. What would you like to do?"
- options: "Use current config (exit setup)" / "Reconfigure (overwrite existing)"

If user selects "Use current config": say "Ready. Use `/opencode:run [task]`." and stop.

---

## Step 2 — Check OpenCode installation

Run: `opencode --version 2>&1`

- If command not found: tell the user to install: `npm install -g opencode-ai`. Stop.
- If it succeeds: note the version and continue.

---

## Step 3 — Choose a provider

Use the **AskUserQuestion tool** with:
- question: "Which AI provider should OpenCode use?"
- options:
  - "Anthropic" / description: "Claude models — ANTHROPIC_API_KEY"
  - "OpenAI" / description: "GPT models — OPENAI_API_KEY"
  - "OpenRouter" / description: "100+ models with one key — OPENROUTER_API_KEY"
  - "Google" / description: "Gemini models — GOOGLE_API_KEY"

If the user selects "Other" or needs Groq/Ollama, accept a free-text answer via the "Other" option that AskUserQuestion provides automatically.

Map to OpenCode provider IDs:

| Choice | PROVIDER_ID |
|--------|-------------|
| Anthropic | `anthropic` |
| OpenAI | `openai` |
| OpenRouter | `openrouter` |
| Google | `google` |
| Groq | `groq` |
| Ollama | `ollama` |
| Other | ask user |

---

## Step 4 — Authenticate (skip for Ollama)

OpenCode handles auth natively in the terminal — the API key never enters this chat.

Tell the user to run this (works the same on all platforms):

```
! opencode providers login -p [PROVIDER_ID]
```

OpenCode will prompt for the API key interactively in the terminal.

Wait for the user to confirm login succeeded.

---

## Step 5 — Pick a model

Tell the user to run this to see valid model IDs (works on all platforms):

```
! opencode models [PROVIDER_ID] 2>&1 | head -50
```

After showing the list, use the **AskUserQuestion tool** with:
- question: "Which model ID do you want to use? Copy the exact ID from the list above."
- options: "I'll type it" (the user will type the exact ID in the Other field)

Accept whatever the user types via the "Other" field. Do not guess or suggest — use only what they provide. Store as `MODEL_ID`.
Full model string: `[PROVIDER_ID]/[MODEL_ID]`

---

## Step 6 — Save model config

Show the command matching their shell type:

**bash (Linux / Mac / Git Bash on Windows):**
```bash
! mkdir -p ~/.opencode-plugin && printf 'export OPENCODE_MODEL="[PROVIDER_ID]/[MODEL_ID]"\n' > ~/.opencode-plugin/config.sh && chmod 600 ~/.opencode-plugin/config.sh && echo "Saved."
```

**PowerShell (Windows):**
```powershell
! New-Item -ItemType Directory -Force "$env:USERPROFILE\.opencode-plugin" | Out-Null; Set-Content -Path "$env:USERPROFILE\.opencode-plugin\config.ps1" -Value '$env:OPENCODE_MODEL = "[PROVIDER_ID]/[MODEL_ID]"'; Write-Host "Saved."
```

Substitute real `PROVIDER_ID` and `MODEL_ID` values. Wait for "Saved." confirmation.

---

## Step 7 — Smoke test

**bash:**
```bash
sh -c '. ~/.opencode-plugin/config.sh && opencode run -m "$OPENCODE_MODEL" "Reply with exactly one word: ready"'
```

**PowerShell:**
```powershell
. "$env:USERPROFILE\.opencode-plugin\config.ps1"; opencode run -m $env:OPENCODE_MODEL "Reply with exactly one word: ready"
```

- Success: report OpenCode's response. Setup complete.
- `ProviderModelNotFoundError`: model ID is wrong — re-run Step 5 and copy the exact ID from `opencode models` output.
- Auth error: login failed — re-run Step 4.
- Other error: show full error output and help diagnose.

---

## Final report

```
OpenCode Setup Complete
───────────────────────
Provider:  [PROVIDER_ID]
Model:     [PROVIDER_ID]/[MODEL_ID]
Shell:     [bash / PowerShell]
Auth:      managed by OpenCode (opencode providers login)
Config:    ~/.opencode-plugin/config.sh  (bash)
           %USERPROFILE%\.opencode-plugin\config.ps1  (PowerShell)

Ready. Use /opencode:run [task] to delegate tasks.
```
