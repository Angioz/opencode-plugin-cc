---
description: Cancel a running OpenCode background job. Optionally pass a job ID; defaults to the most recent running job.
---

# OpenCode: Cancel Job

Cancel a running OpenCode background job.

## Parse arguments

- If `$ARGUMENTS` contains a job ID: run `opencode-jobs cancel [JOB_ID]`
- If `$ARGUMENTS` is empty: first run `opencode-jobs list` to show running jobs, then run `opencode-jobs cancel` (defaults to most recent)

## Execute

Run `opencode-jobs cancel [JOB_ID]` and report the result.

- If cancelled successfully: confirm "Job [JOB_ID] cancelled."
- If already complete: report "Job [JOB_ID] was already finished — use `/opencode:result [JOB_ID]` to see the output."
- If not found: tell the user to run `/opencode:status` to see available jobs.
