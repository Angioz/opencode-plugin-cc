# OpenCode Plugin for Claude Code

Delegate tasks to [OpenCode](https://opencode.ai) from inside Claude Code — for token-efficient codebase exploration, file analysis, and docs updates.

When Claude Code's context window fills up on large codebases, offload heavy tasks to a separate OpenCode session. Results come back as a summary without consuming your main context.

## Commands

| Command | Description |
|---------|-------------|
| `/opencode:setup` | Interactive wizard — choose provider, model, and store API key securely |
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

The wizard will ask which provider and model to use, then prompt you to enter your API key **directly in your terminal** (never in the chat) using hidden input.

### Developer / local testing

```bash
claude --plugin-dir ./plugins/opencode
```

## Usage

### `/opencode:setup`

Interactive wizard that walks through:
1. Provider selection (Anthropic, OpenAI, Google, Groq, Ollama, or custom)
2. Model selection with recommended defaults per provider
3. Secure API key entry via terminal `read -s` — key never appears in the chat
4. Smoke test to confirm everything works

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

- Provider config and API key are stored in `~/.opencode-plugin/config.sh` (chmod 600)
- The plugin uses `opencode run -m [model] [message]` for one-shot headless execution
- Background jobs are tracked as JSON + log files in `~/.opencode-jobs/`
- Zero dependencies — pure POSIX shell scripts, no npm install required

## Security

API keys are **never** handled through the Claude Code chat. During `/opencode:setup`, the plugin generates a `read -s` terminal command for you to run with the `!` prefix. The key is entered with hidden input directly in your terminal and written to `~/.opencode-plugin/config.sh` with `600` permissions.

## Supported Providers

| Provider | Env Var |
|----------|---------|
| Anthropic | `ANTHROPIC_API_KEY` |
| OpenAI | `OPENAI_API_KEY` |
| Google | `GOOGLE_API_KEY` |
| Groq | `GROQ_API_KEY` |
| Ollama | (no key needed) |

## License

MIT
