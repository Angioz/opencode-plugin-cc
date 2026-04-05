---
description: Interactive setup wizard — configure provider, API key, and model for OpenCode. Run this before your first /opencode:run.
---

# OpenCode Setup Wizard

Guide the user step by step. Do NOT skip steps or assume answers.

---

## Step 0 — Check if already configured

Run: `test -s ~/.opencode-plugin/config.sh && echo "EXISTS" || echo "MISSING"`

If EXISTS:
- Run: `grep OPENCODE_MODEL ~/.opencode-plugin/config.sh` — show the current model only (never read/display the API key line)
- Ask: "OpenCode is already configured with the above model. Reconfigure? (yes/no)"
- If no: say "Ready. Run `/opencode:run [task]` to start." and stop.
- If yes: continue to Step 1.

If MISSING: continue to Step 1.

---

## Step 1 — Check OpenCode installation

Run: `opencode --version 2>&1`

- If command not found: tell the user to install OpenCode:
  ```
  npm install -g opencode-ai
  ```
  Then say "Run `/opencode:setup` again after installing." and stop.
- If succeeds: note the version and continue.

---

## Step 2 — Choose a provider

Ask:

> Which AI provider should OpenCode use?
>
> 1. Anthropic (Claude) — `ANTHROPIC_API_KEY`
> 2. OpenAI (GPT) — `OPENAI_API_KEY`
> 3. Google (Gemini) — `GOOGLE_API_KEY`
> 4. Groq — `GROQ_API_KEY`
> 5. Ollama (local, no API key)
> 6. Other

Map selection:

| Choice | PROVIDER_ID | ENV_VAR_NAME |
|--------|-------------|--------------|
| 1 | `anthropic` | `ANTHROPIC_API_KEY` |
| 2 | `openai` | `OPENAI_API_KEY` |
| 3 | `google` | `GOOGLE_API_KEY` |
| 4 | `groq` | `GROQ_API_KEY` |
| 5 | `ollama` | (none) |
| 6 | ask user | ask user |

---

## Step 3 — Choose a model

Show options for the chosen provider:

**Anthropic:** `claude-sonnet-4-6` (recommended) / `claude-opus-4-6` / `claude-haiku-4-5`
**OpenAI:** `gpt-4o` (recommended) / `gpt-4o-mini` / `o3-mini`
**Google:** `gemini-2.0-flash` (recommended) / `gemini-2.5-pro`
**Groq:** `llama-3.3-70b-versatile` (recommended) / `llama-3.1-8b-instant`
**Ollama:** ask user for the locally pulled model name (e.g. `llama3`, `codellama`)
**Other:** ask user for full `provider/model` string

Store: `MODEL_NAME`. Full model string = `PROVIDER_ID/MODEL_NAME`.

---

## Step 4 — Save API key via terminal (skip for Ollama)

**CRITICAL: Never ask the user to paste an API key into the chat.** It must go directly into their terminal.

For non-Ollama providers, tell the user:

> Your API key will be entered directly in your terminal — it will never appear in this conversation.
>
> Run this command (key input will be hidden):
>
> ```
> ! mkdir -p ~/.opencode-plugin && read -rsp "Paste your [ENV_VAR_NAME] (hidden): " _k && printf 'export [ENV_VAR_NAME]="%s"\nexport OPENCODE_MODEL="[PROVIDER_ID]/[MODEL_NAME]"\n' "$_k" > ~/.opencode-plugin/config.sh && chmod 600 ~/.opencode-plugin/config.sh && unset _k && echo "Saved."
> ```

Substitute the real values of `ENV_VAR_NAME`, `PROVIDER_ID`, `MODEL_NAME` before showing the command.

Wait for the user to confirm they ran it and saw "Saved."

For **Ollama** (no key needed), tell the user:

> Run this to save your config:
>
> ```
> ! mkdir -p ~/.opencode-plugin && printf 'export OPENCODE_MODEL="ollama/[MODEL_NAME]"\n' > ~/.opencode-plugin/config.sh && chmod 600 ~/.opencode-plugin/config.sh && echo "Saved."
> ```

Substitute the model name.

---

## Step 5 — Smoke test

Run:

```bash
sh -c '. ~/.opencode-plugin/config.sh && opencode run -m "$OPENCODE_MODEL" "Reply with exactly one word: ready"'
```

- Success: report OpenCode's response. Setup complete.
- Auth error: "Your API key may be incorrect — re-run Step 4."
- Model error: "That model name may be invalid — run `/opencode:setup` again to pick a different one."
- Not found: "OpenCode not installed — run `npm install -g opencode-ai` first."

---

## Final report

```
OpenCode Setup Complete
───────────────────────
Provider:  [PROVIDER_ID]
Model:     [PROVIDER_ID]/[MODEL_NAME]
Config:    ~/.opencode-plugin/config.sh  (chmod 600)

Ready. Use /opencode:run [task] to delegate tasks.
```
