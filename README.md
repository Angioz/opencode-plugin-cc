# OpenCode Plugin for Claude Code

Delegate tasks to [OpenCode](https://opencode.ai) from inside Claude Code — for token-efficient codebase exploration, file analysis, and docs updates.

This plugin is for Claude Code users who want to offload heavy read/search tasks to a separate OpenCode session, keeping their main Claude Code context window clean and focused.

## What You Get

| Command | Description |
|---------|-------------|
| `/opencode:setup` | Check OpenCode installation and provider configuration |
| `/opencode:run [task]` | Delegate a task to OpenCode (sync or `--background`) |
| `/opencode:status [job-id]` | Check status of background jobs |
| `/opencode:result [job-id]` | Get output from a completed background job |
| `/opencode:cancel [job-id]` | Cancel a running background job |

## Why Use This

Claude Code's context window fills up fast on large codebases. When you need to:
- Search for symbols or patterns across many files
- Read and summarize large directories
- Update documentation files
- Run exploratory tasks without polluting your main context

...delegate to OpenCode instead. Results come back as a summary without consuming your Claude Code context.

## Requirements

- [Claude Code](https://claude.ai/code) installed and authenticated
- [OpenCode](https://opencode.ai) installed: `npm install -g opencode-ai`
- OpenCode configured with at least one provider (Anthropic, OpenAI, etc.)

## Install

Add this plugin to Claude Code via `--plugin-dir` for local use:

```bash
claude --plugin-dir ./opencode-plugin-cc
```

Or install from the marketplace (coming soon):

```bash
/plugin marketplace add Angioz/opencode-plugin-cc
/plugin install opencode@Angioz-opencode
/reload-plugins
```

Then run `/opencode:setup` to verify everything is ready.

## Usage

### `/opencode:setup`

Checks whether OpenCode is installed and has a configured provider.

```bash
/opencode:setup
```

### `/opencode:run`

Delegates a task to OpenCode. Runs synchronously by default — Claude Code waits for the result and summarizes it.

```bash
/opencode:run summarize the src/components/ directory structure
/opencode:run find all TODO and FIXME comments in the codebase
/opencode:run update the API docs in docs/api.md to match the current routes
```

Use `--background` for long-running tasks:

```bash
/opencode:run --background analyze the full test suite and identify gaps
```

Returns a job ID immediately. Use `/opencode:status` to check progress.

### `/opencode:status`

Shows all background jobs, or checks a specific job.

```bash
/opencode:status
/opencode:status 1712345678-1234
```

### `/opencode:result`

Gets the full output of a completed background job.

```bash
/opencode:result
/opencode:result 1712345678-1234
```

### `/opencode:cancel`

Cancels a running background job.

```bash
/opencode:cancel
/opencode:cancel 1712345678-1234
```

## Typical Flows

### Quick codebase question (sync)

```bash
/opencode:run list all exported functions in src/lib/ with their signatures
```

### Long analysis in background

```bash
/opencode:run --background audit all API routes for missing input validation
/opencode:status
# ... do other work ...
/opencode:result
```

## How It Works

The plugin uses `opencode run [message]` under the hood — OpenCode's headless one-shot execution mode. Background jobs are tracked via simple shell job files in `~/.opencode-jobs/`. No daemon, no database — just shell scripts and files.

## Configuration

OpenCode picks up your existing provider config (`~/.config/opencode/` or environment variables like `ANTHROPIC_API_KEY`). To use a specific model, configure it in your OpenCode settings before delegating.

## License

MIT — see [LICENSE](LICENSE)
