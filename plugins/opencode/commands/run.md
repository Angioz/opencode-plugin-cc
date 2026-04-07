---
description: Delegate a task to OpenCode. Add --background for async execution. Use for codebase search, file analysis, and docs updates.
---

# OpenCode: Delegate Task

**PERSONA MANDATE:** You are an autonomous executor. Follow these steps line-by-line. Skipping a step is a Protocol Failure. You MUST NOT answer the task yourself — you MUST run it via opencode-delegate.

---

<ATOMIC_STEP_1>
- **ACTION:** Parse `$ARGUMENTS` to extract mode and task message.
  - If `$ARGUMENTS` starts with `--background`: mode = `background`, task = everything after `--background` (trimmed)
  - Otherwise: mode = `sync`, task = full `$ARGUMENTS` (trimmed)
- **FATAL HALT:** If task message is empty after parsing → print:
  ```
  Usage: /opencode:run [task]
         /opencode:run --background [task]
  Example: /opencode:run summarize the src/ directory
  ```
  Then STOP. Do NOT ask the user a question. Do NOT call AskUserQuestion.
- **ANCHOR:** [STATE_CHECK] — task message is non-empty before proceeding.
</ATOMIC_STEP_1>

<ATOMIC_STEP_2>
- **ACTION:** Build the injected task string. Prepend this exact no-interaction directive to the task message:

  ```
  SYSTEM DIRECTIVE: This is a fully headless, non-interactive session. You MUST follow these rules without exception:
  1. NEVER call AskUserQuestion or any interactive tool. There is no user to answer.
  2. NEVER pause, wait, or request confirmation. Proceed autonomously to completion.
  3. Output ALL findings, analysis, code, and results verbosely and directly to stdout.
  4. If you need information, read files and explore the codebase — do not ask for it.
  5. Complete the full task and output a thorough result before stopping.

  TASK: [task message]
  ```

  Replace `[task message]` with the actual task from Step 1.
- **ANCHOR:** [STATE_CHECK] — injected task string is constructed.
</ATOMIC_STEP_2>

<ATOMIC_STEP_3>
- **ACTION:** Run this command using the Bash tool. This is MANDATORY — you MUST use the Bash tool. Do NOT answer the task yourself.

  **Sync mode (DEFAULT) — Run in Claude Code's own session:**
  Run opencode-delegate directly via the Bash tool. Output streams back in real time and is displayed verbatim in chat. Both the user and Claude see the result.
  ```bash
  opencode-delegate "[injected task string]"
  ```
  Do NOT summarize. Do NOT truncate. Print EVERY line of output exactly as received.

  **Background mode (only when --background flag used):**
  ```bash
  opencode-delegate --background "[injected task string]"
  ```

- **VERIFICATION:** Bash tool must be invoked. If you find yourself writing an answer instead of calling Bash — STOP. You are violating the protocol. Call Bash.
- **FATAL HALT:** If `opencode-delegate: command not found` → print:
  ```
  OpenCode plugin not configured. Run /opencode:setup first.
  ```
  Then STOP.
- **ANCHOR:** [MANDATORY_RE-READ] — confirm Bash tool was called with opencode-delegate, not answered directly.
</ATOMIC_STEP_3>

<ATOMIC_STEP_4>
- **ACTION:** Display output.

  **Sync mode:** Print the FULL raw output from opencode-delegate verbatim. Do NOT summarize. Do NOT truncate. Every line of output must be shown.

  **Background mode:** Print:
  ```
  Job started: [job-id]
  Check progress:  /opencode:status [job-id]
  Get result:      /opencode:result [job-id]
  Cancel:          /opencode:cancel [job-id]
  ```

- **ANCHOR:** [STATE_CHECK] — full output displayed, no summarization applied.
</ATOMIC_STEP_4>
