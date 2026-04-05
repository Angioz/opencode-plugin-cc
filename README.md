# OpenCode Plugin for Claude Code

Delegate tasks to [OpenCode](https://opencode.ai) from inside Claude Code — for token-efficient codebase exploration, file analysis, and docs updates.

When Claude Code's context window fills up on large codebases, offload heavy tasks to a separate OpenCode session. Results come back as a summary without consuming your main context.

## Commands

| Command | Description |
|---------|-------------|
| `/opencode:setup` | Interactive wizard — choose provider, authenticate in terminal, select model from live CLI output |
| `/opencode:run [task]` | Delegate a task to OpenCode (sync by default, or `--background`) |
| `/opencode:status [job-id]` | Check status of background jobs |
| `/opencode:result [job-id]` | Get output from a completed background job |
| `/opencode:cancel [job-id]` | Cancel a running background job |

## Install

**Step 1** — Install [OpenCode](https://opencode.ai):
```bash
npm install -g opencode-ai
```

**Step 2** — Add the marketplace and install the plugin in Claude Code:
```
/plugin marketplace add Angioz/opencode-plugin-cc
/plugin install opencode@angioz-opencode
/reload-plugins
```

**Step 3** — Run the setup wizard:
```
/opencode:setup
```

The wizard guides you through provider selection, authenticates via `opencode providers login` in your terminal (API key never in chat), then picks a model from real CLI output.

### Developer / local testing

```bash
claude --plugin-dir ./plugins/opencode
```

## Usage

### `/opencode:setup`

Interactive wizard that walks through:

1. **Shell detection** — detects bash vs PowerShell for correct command variants
2. **Config check** — skips re-setup if already configured (or lets you reconfigure)
3. **Provider selection** — choose from a menu with free-tier info and key URLs
4. **Authentication** — runs `! opencode providers login -p [provider]` in your terminal; API key entered interactively, never in chat
5. **Model selection** — Claude runs `opencode models [provider]`, parses the live output, and presents real model IDs via a widget
6. **Config save** — writes `OPENCODE_MODEL` to `~/.opencode-plugin/config.sh` (model string only, no API key)
7. **Smoke test** — verifies everything works end-to-end

Re-run any time to reconfigure.

### `/opencode:run`

Delegates a task to OpenCode synchronously — Claude Code waits for the result and summarizes it.

```
/opencode:run summarize the src/components/ directory structure
/opencode:run find all TODO and FIXME comments in the codebase
/opencode:run update the API docs in docs/api.md to match the current routes
```

Use `--background` for long-running tasks:

```
/opencode:run --background audit all API routes for missing input validation
```

Returns a job ID immediately. Check progress with `/opencode:status`.

### `/opencode:status`

```
/opencode:status                    # list all jobs
/opencode:status 1712345678-1234    # check a specific job
```

### `/opencode:result`

```
/opencode:result                    # result of the most recent job
/opencode:result 1712345678-1234    # result of a specific job
```

### `/opencode:cancel`

```
/opencode:cancel                    # cancel the most recent running job
/opencode:cancel 1712345678-1234    # cancel a specific job
```

## Typical Flows

### Quick codebase question (sync)

```
/opencode:run list all exported functions in src/lib/ with their signatures
```

### Long analysis in background

```
/opencode:run --background audit all API routes for missing input validation
/opencode:status
# ... continue working in Claude Code ...
/opencode:result
```

## How It Works

- The plugin stores `OPENCODE_MODEL` in `~/.opencode-plugin/config.sh` (chmod 600) — model string only, no credentials
- API keys are managed entirely by OpenCode via `opencode providers login` — stored in `~/.local/share/opencode/auth.json`
- The plugin uses `opencode run -m [model] [message]` for one-shot headless execution
- Background jobs are tracked as JSON + log files in `~/.opencode-jobs/`
- Zero dependencies — pure POSIX shell scripts, no npm install required

## Security

**API keys never pass through the Claude Code chat conversation** — not as user input, not as Claude output, not as command arguments.

During `/opencode:setup`, you run `! opencode providers login -p [provider]` in your terminal. OpenCode handles the interactive key prompt itself and stores credentials in its own secure store (`~/.local/share/opencode/auth.json`). The plugin only reads the model choice it saved during setup.

## Supported Providers

| Provider | Free tier | Key URL |
|----------|-----------|---------|
| OpenRouter | Yes — 26 free models | openrouter.ai/keys |
| Google | Yes — Gemini free tier | aistudio.google.com/apikey |
| Groq | Yes — rate-limited | console.groq.com/keys |
| Anthropic | No | console.anthropic.com/keys |
| OpenAI | No | platform.openai.com/api-keys |
| Ollama | N/A (local) | — |

## License

MIT
