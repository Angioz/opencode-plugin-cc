---
description: Interactive setup wizard — authenticate a provider and choose a model for OpenCode. Run this before your first /opencode:run.
---

# OpenCode Setup Wizard

Guide the user step by step. Do NOT skip steps or assume answers.

---

## Step 0 — Check if already configured

Run: `test -s ~/.opencode-plugin/config.sh && echo "EXISTS" || echo "MISSING"`

If EXISTS:
- Run: `cat ~/.opencode-plugin/config.sh` — show the model line only
- Ask: "OpenCode is already configured. Reconfigure? (yes/no)"
- If no: say "Ready. Use `/opencode:run [task]`." and stop.
- If yes: continue.

---

## Step 1 — Check OpenCode installation

Run: `opencode --version 2>&1`

- If command not found: tell the user to install OpenCode:
  ```
  npm install -g opencode-ai
  ```
  Say "Run `/opencode:setup` again after installing." and stop.
- If it succeeds: note the version and continue.

---

## Step 2 — Choose a provider

Ask the user which provider to use. Show this list:

> Which provider should OpenCode use?
>
> 1. Anthropic
> 2. OpenAI
> 3. OpenRouter (access 100+ models with one key)
> 4. Google
> 5. Groq
> 6. Ollama (local, no key needed)
> 7. Other — I'll type the provider ID

Map common selections to OpenCode provider IDs:

| Choice | Provider ID |
|--------|-------------|
| Anthropic | `anthropic` |
| OpenAI | `openai` |
| OpenRouter | `openrouter` |
| Google | `google` |
| Groq | `groq` |
| Ollama | `ollama` |
| Other | ask user to type exact ID |

Store as `PROVIDER_ID`.

---

## Step 3 — Authenticate (skip for Ollama)

For all providers except Ollama, the API key must be entered **directly in the terminal** — never in this chat. OpenCode handles this natively.

Tell the user:

> Run this to log in to [PROVIDER_ID]. OpenCode will prompt for your API key in the terminal:
>
> ```
> ! opencode providers login -p [PROVIDER_ID]
> ```

Wait for the user to confirm they completed login successfully.

For **Ollama**: skip this step (no authentication needed).

---

## Step 4 — Pick a model

Tell the user to run this to see the real list of available models for their provider:

> ```
> ! opencode models [PROVIDER_ID] 2>&1 | head -50
> ```

Show this output to the user and ask them to pick a model from the list. Tell them to copy the exact model ID as shown (including any `:free` or version suffixes).

Wait for the user's selection. Store as `MODEL_ID`.

The full model string is: `[PROVIDER_ID]/[MODEL_ID]`

---

## Step 5 — Save model config

Write only the model choice to the config file (no API key — OpenCode manages that):

Tell the user:

> ```
> ! mkdir -p ~/.opencode-plugin && printf 'export OPENCODE_MODEL="[PROVIDER_ID]/[MODEL_ID]"\n' > ~/.opencode-plugin/config.sh && chmod 600 ~/.opencode-plugin/config.sh && echo "Saved."
> ```

Substitute the real `PROVIDER_ID` and `MODEL_ID` values. Wait for "Saved." confirmation.

---

## Step 6 — Smoke test

Run:

```bash
sh -c '. ~/.opencode-plugin/config.sh && opencode run -m "$OPENCODE_MODEL" "Reply with exactly one word: ready"'
```

- Success: report OpenCode's response. Setup complete.
- `ProviderModelNotFoundError`: the model ID is wrong. Ask user to re-run Step 4 and copy the exact ID from `opencode models` output.
- Auth error: login may have failed. Ask user to re-run Step 3.
- Any other error: show the full error and help diagnose.

---

## Final report

```
OpenCode Setup Complete
───────────────────────
Provider:  [PROVIDER_ID]
Model:     [PROVIDER_ID]/[MODEL_ID]
Auth:      managed by OpenCode (opencode providers login)
Config:    ~/.opencode-plugin/config.sh  (model only, chmod 600)

Ready. Use /opencode:run [task] to delegate tasks.
```
