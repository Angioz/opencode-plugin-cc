---
description: Delegate a task to OpenCode. Add --background for async execution. Use for codebase search, file analysis, and docs updates.
---

# OpenCode: Delegate Task

Delegate the following task to OpenCode: $ARGUMENTS

## Parse arguments

First, check if `$ARGUMENTS` starts with `--background`:
- If yes: set mode = background, extract the task message (everything after `--background`)
- If no: set mode = sync, full `$ARGUMENTS` is the task message

If the task message is empty after parsing: ask the user "What task should I delegate to OpenCode?"

## Execute

Run via the bin script (in PATH when plugin loaded):

**Sync mode:**
```
opencode-delegate "[task message]"
```
Stream the output. When complete, summarize the key findings in 3-5 bullet points.

**Background mode:**
```
opencode-delegate --background "[task message]"
```
This returns immediately with a job ID. Report the job ID to the user and tell them:
- Check progress: `/opencode:status`
- Get result: `/opencode:result`
- Cancel: `/opencode:cancel`

## Notes

- The task runs in the current working directory (same repo as Claude Code)
- OpenCode uses whatever provider/model is configured in its own settings
- For large codebase tasks, background mode is recommended to avoid blocking
- Good tasks to delegate: symbol search, directory summarization, TODO audits, docs updates
- Keep Claude Code context: do NOT read the files yourself — let OpenCode do the heavy lifting
