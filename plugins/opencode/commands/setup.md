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

## Step 5 — List and Select Model (Free Battery System)

Run BOTH of these commands using the Bash tool (you run them — not the user):

```bash
opencode models openrouter 2>&1 | grep ":free"
```

```bash
opencode models 2>&1 | grep "^opencode/"
```

Parse both outputs:
- **Free OpenRouter models**: lines ending in `:free` — pick top 8, prefer qwen, deepseek, meta-llama, google vendors
- **OpenCode native models**: lines starting with `opencode/` — pick top 4

Split the 8 free OpenRouter models into two batteries of 4. Present **Battery 1 first**.

---

### Battery 1 — Free models (top 4)

You MUST call the AskUserQuestion tool with EXACTLY this structure, substituting REAL model IDs from the grep output:

```json
{
  "questions": [{
    "question": "Which free model should OpenCode use? (Battery 1 of 3)",
    "header": "Free",
    "multiSelect": false,
    "options": [
      {
        "label": "[model-id-1] (Recommended)",
        "description": "[vendor] — free, coding/instruction"
      },
      {
        "label": "[model-id-2]",
        "description": "[vendor] — free"
      },
      {
        "label": "[model-id-3]",
        "description": "[vendor] — free"
      },
      {
        "label": "See 4 more free models →",
        "description": "Show next battery of free models"
      }
    ]
  }]
}
```

- If user selects a model ID → store as `MODEL_ID`, skip to Step 6.
- If user selects "See 4 more free models →" → present Battery 2 below.
- If user types a custom ID via "Other" → validate (see validation rule below).

---

### Battery 2 — Free models (next 4)

You MUST call the AskUserQuestion tool with EXACTLY this structure:

```json
{
  "questions": [{
    "question": "Which free model should OpenCode use? (Battery 2 of 3)",
    "header": "Free 2",
    "multiSelect": false,
    "options": [
      {
        "label": "[model-id-5]",
        "description": "[vendor] — free"
      },
      {
        "label": "[model-id-6]",
        "description": "[vendor] — free"
      },
      {
        "label": "[model-id-7]",
        "description": "[vendor] — free"
      },
      {
        "label": "See OpenCode free models →",
        "description": "Show OpenCode native models (also free)"
      }
    ]
  }]
}
```

- If user selects a model ID → store as `MODEL_ID`, skip to Step 6.
- If user selects "See OpenCode free models →" → present Battery 3 below.

---

### Battery 3 — OpenCode native models

You MUST call the AskUserQuestion tool with EXACTLY this structure, substituting REAL model IDs from the `grep "^opencode/"` output:

```json
{
  "questions": [{
    "question": "Which OpenCode native model should OpenCode use? (Battery 3 of 3)",
    "header": "OpenCode",
    "multiSelect": false,
    "options": [
      {
        "label": "[opencode/model-1] (Recommended)",
        "description": "OpenCode native — free with OpenCode Zen account"
      },
      {
        "label": "[opencode/model-2]",
        "description": "OpenCode native — free"
      },
      {
        "label": "[opencode/model-3]",
        "description": "OpenCode native — free"
      },
      {
        "label": "Enter model ID manually",
        "description": "Type any model ID from opencode models output"
      }
    ]
  }]
}
```

- If user selects a model ID → store as `MODEL_ID`, skip to Step 6.
- If user selects "Enter model ID manually" → accept via "Other" text input.

---

### Custom model ID validation

If user types a custom model ID at any battery:
- Run this command using the Bash tool: `opencode models openrouter 2>&1 | grep "[typed-id]"`
- If found: accept it as `MODEL_ID`
- If NOT found: print "That model ID wasn't found. Here's the full free list:" then run:
  ```bash
  opencode models openrouter 2>&1 | grep ":free"
  ```
  Then You MUST call the AskUserQuestion tool again from Battery 1.

---

Store `MODEL_ID` = the exact full model ID string (e.g. `openrouter/qwen/qwen3.6-plus:free`).

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
