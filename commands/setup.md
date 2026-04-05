---
description: Interactive setup wizard — configure provider, API key, and model for OpenCode. Run this before your first /opencode:run.
---

# OpenCode Setup Wizard

You are running an interactive setup wizard. Guide the user step by step. Do NOT skip steps or assume answers.

---

## Step 0 — Check if already configured

Check if `~/.opencode-plugin/config.sh` exists and is non-empty.

If it exists:
- Read it and show the current config (provider and model, but MASK the API key — show only the first 8 characters + `...`)
- Ask: "OpenCode is already configured. Reconfigure? (yes/no)"
- If no: confirm "Ready to use. Run `/opencode:run [task]` to start." and stop.
- If yes: continue to Step 1.

If it does not exist: continue to Step 1.

---

## Step 1 — Check OpenCode installation

Run: `opencode --version 2>&1`

- If it fails (command not found): tell the user OpenCode is not installed.
  Provide install options:
  ```
  npm install -g opencode-ai
  ```
  Or: visit opencode.ai for alternative methods.
  Say: "Install OpenCode, then run `/opencode:setup` again." and stop.

- If it succeeds: note the version and continue.

---

## Step 2 — Choose a provider

Ask the user:

> Which AI provider do you want OpenCode to use?
>
> 1. Anthropic (Claude models)
> 2. OpenAI (GPT models)
> 3. Google (Gemini models)
> 4. Groq (fast inference)
> 5. Ollama (local, no API key needed)
> 6. Other (I'll enter manually)

Wait for the user's answer. Map their selection to a provider ID:

| Selection | Provider ID | Env var needed |
|-----------|-------------|----------------|
| 1 / Anthropic | `anthropic` | `ANTHROPIC_API_KEY` |
| 2 / OpenAI | `openai` | `OPENAI_API_KEY` |
| 3 / Google | `google` | `GOOGLE_API_KEY` |
| 4 / Groq | `groq` | `GROQ_API_KEY` |
| 5 / Ollama | `ollama` | none |
| 6 / Other | ask user for provider ID and env var name |

Store: `PROVIDER_ID`, `ENV_VAR_NAME`.

---

## Step 3 — API key (skip for Ollama)

If provider is Ollama: skip this step (no key needed).

Tell the user:
> Paste your `[ENV_VAR_NAME]` API key below. It will be stored locally in `~/.opencode-plugin/config.sh` and never sent anywhere except to [provider] when OpenCode runs.

Wait for the user's response. Store as `API_KEY`.

Do not validate the format — just store whatever they paste.

---

## Step 4 — Choose a model

Show the model list for the selected provider and ask the user to pick one:

**Anthropic:**
> 1. claude-opus-4-6 (most capable, slower)
> 2. claude-sonnet-4-6 (recommended — fast + capable)
> 3. claude-haiku-4-5 (fastest, cheapest)

**OpenAI:**
> 1. gpt-4o (recommended)
> 2. gpt-4o-mini (faster, cheaper)
> 3. o3-mini (reasoning)

**Google:**
> 1. gemini-2.0-flash (recommended — fast)
> 2. gemini-2.5-pro (most capable)

**Groq:**
> 1. llama-3.3-70b-versatile (recommended)
> 2. llama-3.1-8b-instant (fastest)

**Ollama:**
> Enter the model name you have pulled locally (e.g. `llama3`, `mistral`, `codellama`).

**Other:**
> Enter the model name in `provider/model` format as used by OpenCode.

Wait for user selection. Store as `MODEL_NAME`.

The full model ID for OpenCode will be: `PROVIDER_ID/MODEL_NAME`
(e.g. `anthropic/claude-sonnet-4-6`)

---

## Step 5 — Write config

Create the directory and write the config file:

```bash
mkdir -p ~/.opencode-plugin
```

Write `~/.opencode-plugin/config.sh` with this content (substitute actual values):

```sh
# OpenCode plugin config — written by /opencode:setup
# Edit manually or re-run /opencode:setup to reconfigure

export [ENV_VAR_NAME]="[API_KEY]"
export OPENCODE_MODEL="[PROVIDER_ID]/[MODEL_NAME]"
```

For Ollama (no API key), write:
```sh
# OpenCode plugin config — written by /opencode:setup
export OPENCODE_MODEL="ollama/[MODEL_NAME]"
```

Set permissions: `chmod 600 ~/.opencode-plugin/config.sh`

---

## Step 6 — Verify

Run a quick smoke test. Source the config and run a minimal OpenCode task:

```bash
sh -c '. ~/.opencode-plugin/config.sh && opencode run "Reply with exactly one word: ready"'
```

- If it succeeds: report success with the response from OpenCode.
- If it fails: show the error output and suggest:
  - Check the API key is correct
  - Check the model name is valid for the provider
  - Run `/opencode:setup` again to reconfigure

---

## Final report

```
OpenCode Setup Complete
───────────────────────
Provider:  [provider]
Model:     [provider/model]
API Key:   [first 8 chars]...
Config:    ~/.opencode-plugin/config.sh

Ready. Use /opencode:run [task] to delegate tasks.
```
