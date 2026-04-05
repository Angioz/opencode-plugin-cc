---
description: Interactive setup wizard — authenticate a provider and choose a model for OpenCode. Run this before your first /opencode:run.
---

# OpenCode Setup Wizard

You are running the OpenCode setup wizard. Follow these steps EXACTLY. Do not skip steps, do not assume answers, do not paraphrase instructions.

---

## Step 0 — Detect Shell

Run this command using the Bash tool:

```bash
uname -s 2>/dev/null || echo "Windows-PowerShell"
```

Classify the result:

| Output contains | SHELL_TYPE |
|-----------------|------------|
| `MINGW`, `MSYS`, `CYGWIN` | `bash` (Git Bash on Windows) |
| `Linux` | `bash` |
| `Darwin` | `bash` |
| `Windows-PowerShell` or command error | `powershell` |

Store `SHELL_TYPE` for use in Steps 6 and 7.

If `SHELL_TYPE` is `powershell`, print this note before continuing:

> Note: This plugin's scripts require a bash-compatible shell (Git Bash, WSL, or native Linux/Mac). If you're on Windows, we recommend running Claude Code from Git Bash for full compatibility.

---

## Step 1 — Check Existing Config

Run this command using the Bash tool:

**bash:**
```bash
test -s ~/.opencode-plugin/config.sh && grep OPENCODE_MODEL ~/.opencode-plugin/config.sh || echo "MISSING"
```

**powershell:**
```bash
test -s "$USERPROFILE/.opencode-plugin/config.ps1" && grep OPENCODE_MODEL "$USERPROFILE/.opencode-plugin/config.ps1" || echo "MISSING"
```

If the result is `MISSING`: continue to Step 2.

If config exists: You MUST call the AskUserQuestion tool with EXACTLY these parameters:

```json
{
  "questions": [{
    "question": "OpenCode is already configured. What would you like to do?",
    "header": "Config",
    "multiSelect": false,
    "options": [
      {
        "label": "Keep current config",
        "description": "Exit setup — use existing provider and model"
      },
      {
        "label": "Reconfigure",
        "description": "Overwrite current config with new provider/model"
      }
    ]
  }]
}
```

- "Keep current config" → Print "Ready. Use `/opencode:run [task]` to delegate." → STOP.
- "Reconfigure" → Continue to Step 2.

---

## Step 2 — Check OpenCode Installed

Run this command using the Bash tool:

```bash
opencode --version 2>&1
```

If command not found or error: print the following and STOP:

```
OpenCode is not installed. Install it with:

  npm install -g opencode-ai

Then run /opencode:setup again.
```

If success: note the version (e.g. "OpenCode v1.2.27 detected.") and continue to Step 3.

---

## Step 3 — Choose Provider

You MUST call the AskUserQuestion tool with EXACTLY these parameters:

```json
{
  "questions": [{
    "question": "Which AI provider should OpenCode use?",
    "header": "Provider",
    "multiSelect": false,
    "options": [
      {
        "label": "OpenRouter (Recommended)",
        "description": "100+ models, 26 free — get a key at openrouter.ai/keys"
      },
      {
        "label": "Anthropic",
        "description": "Claude models — requires paid API key from console.anthropic.com/keys"
      },
      {
        "label": "OpenAI",
        "description": "GPT models — requires paid API key from platform.openai.com/api-keys"
      },
      {
        "label": "Google",
        "description": "Gemini models — free tier available at aistudio.google.com/apikey"
      }
    ]
  }]
}
```

Map selection to PROVIDER_ID:

| Selection | PROVIDER_ID | Has free tier |
|-----------|-------------|---------------|
| OpenRouter (Recommended) | `openrouter` | Yes (26 models) |
| Anthropic | `anthropic` | No |
| OpenAI | `openai` | No |
| Google | `google` | Yes |

If user types "Other" (via the AskUserQuestion free-text field):
- Groq → PROVIDER_ID = `groq`, note: "Free tier at console.groq.com/keys"
- Ollama → PROVIDER_ID = `ollama` → skip Step 4 (no auth needed)
- Anything else → accept as PROVIDER_ID

Store `PROVIDER_ID` and continue to Step 4.

---

## Step 4 — Authenticate (skip if PROVIDER_ID = `ollama`)

Your API key will be entered directly in your terminal — it never appears in this chat.

Tell the user:

> Run this command in your terminal (the `!` prefix runs it directly):
> ```
> ! opencode providers login -p [PROVIDER_ID]
> ```
> OpenCode will prompt you for the key interactively.

Show the provider-specific guidance for the selected PROVIDER_ID:

| Provider | Guidance |
|----------|----------|
| `openrouter` | "Don't have a key? Get a free one at openrouter.ai/keys — no credit card required." |
| `google` | "Get a free API key at aistudio.google.com/apikey" |
| `groq` | "Get a free key at console.groq.com/keys" |
| `anthropic` | "Get your API key at console.anthropic.com/keys (paid plan required)" |
| `openai` | "Get your API key at platform.openai.com/api-keys (paid plan required)" |

Wait for user confirmation. Acceptable signals: "done", "logged in", "ready", or any confirmation.

If user reports an error:
- "command not found" → OpenCode not installed → go back to Step 2
- "invalid key" or "wrong key format" → ask user to re-run the login command
- "provider not found" → wrong PROVIDER_ID → check `opencode providers list`, then re-ask

---

## Step 5 — List and Select Model

Run this command using the Bash tool (you run it — not the user — to parse the output):

```bash
opencode models [PROVIDER_ID] 2>&1
```

Replace `[PROVIDER_ID]` with the actual provider ID stored from Step 3.

Parse the output:
- Output format: one model ID per line (e.g. `openrouter/qwen/qwen3.6-plus:free`)
- Select the top 3 most relevant models from the output. Criteria:
  - Prefer models with coding/instruction capabilities
  - Prefer `:free` models if the provider has them (OpenRouter, Google)
  - Prefer well-known vendors (google, meta-llama, qwen, deepseek)

If the provider has no direct models (e.g. `anthropic` models live under `openrouter/anthropic/...`):
- Run `opencode models` (no provider filter) using the Bash tool and grep for the provider name
- If still no results: tell user this provider may need to be accessed via OpenRouter

You MUST call the AskUserQuestion tool with this exact structure, substituting REAL model IDs from the command output above:

```json
{
  "questions": [{
    "question": "Which model should OpenCode use? (from opencode models output)",
    "header": "Model",
    "multiSelect": false,
    "options": [
      {
        "label": "[model-id-1] (Recommended)",
        "description": "[vendor] — [brief note: free/paid, capability]"
      },
      {
        "label": "[model-id-2]",
        "description": "[vendor] — [brief note]"
      },
      {
        "label": "[model-id-3]",
        "description": "[vendor] — [brief note]"
      }
    ]
  }]
}
```

The options MUST contain exact model IDs copied from the `opencode models` output. Do NOT guess or hardcode model names.

If user enters a custom model ID via the "Other" field:
- Check if it appears in the `opencode models` output
- If found: accept it
- If NOT found: print "That model ID wasn't found in the available models. Here's the full list:" then run this command using the Bash tool:
  ```bash
  opencode models [PROVIDER_ID] 2>&1 | head -40
  ```
  Then You MUST call the AskUserQuestion tool again with updated options from the output.

Store `MODEL_ID` = the exact full model ID string from the output (e.g. `openrouter/qwen/qwen3.6-plus:free`).

---

## Step 6 — Save Config

Config stores ONLY the model string. No API key.

Show the command matching SHELL_TYPE and substitute the real `MODEL_ID` before showing:

**bash (SHELL_TYPE = `bash`):**

Tell the user to run:
```
! mkdir -p ~/.opencode-plugin && printf 'export OPENCODE_MODEL="[MODEL_ID]"\n' > ~/.opencode-plugin/config.sh && chmod 600 ~/.opencode-plugin/config.sh && echo "Saved."
```

**powershell (SHELL_TYPE = `powershell`):**

Tell the user to run:
```
! New-Item -ItemType Directory -Force "$env:USERPROFILE\.opencode-plugin" | Out-Null; Set-Content -Path "$env:USERPROFILE\.opencode-plugin\config.ps1" -Value '$env:OPENCODE_MODEL = "[MODEL_ID]"'; Write-Host "Saved."
```

Wait for user to confirm "Saved." before continuing to Step 7.

---

## Step 7 — Smoke Test

Run this command using the Bash tool:

**bash:**
```bash
sh -c '. ~/.opencode-plugin/config.sh && opencode run -m "$OPENCODE_MODEL" "Reply with exactly one word: ready"'
```

**powershell:**
```bash
. "$USERPROFILE/.opencode-plugin/config.ps1" 2>/dev/null; opencode run -m "$OPENCODE_MODEL" "Reply with exactly one word: ready"
```

If OpenCode returns a response: print the Final Report below.

Error routing:

| Error | Diagnosis | Route to |
|-------|-----------|----------|
| `ProviderModelNotFoundError` | Model ID is wrong | Step 5 — re-list models and re-select |
| Auth / unauthorized error | Login failed or key expired | Step 4 — re-authenticate |
| `opencode: command not found` | Not installed | Step 2 |
| Timeout / network error | Connection issue | Ask user to check network, retry Step 7 |

---

## Final Report

Print after successful smoke test:

```
OpenCode Setup Complete
───────────────────────
Provider:  [PROVIDER_ID]
Model:     [MODEL_ID]
Shell:     [SHELL_TYPE]
Auth:      Managed by OpenCode (opencode providers login)
Config:    ~/.opencode-plugin/config.sh  (model only, chmod 600)

Ready. Use /opencode:run [task] to delegate tasks to OpenCode.
```
