---
description: Get the output from a completed OpenCode background job. Optionally pass a job ID; defaults to the most recent job.
---

# OpenCode: Job Result

Retrieve and display the output of a completed OpenCode background job.

## Parse arguments

- If `$ARGUMENTS` contains a job ID: run `opencode-jobs result [JOB_ID]`
- If `$ARGUMENTS` is empty: run `opencode-jobs result` (defaults to most recent job)

## Execute

Run `opencode-jobs result [JOB_ID]` and display the output.

After showing the raw output, provide a brief summary:
- 3-5 bullet points covering the key findings or changes made
- If the output is very long (>50 lines), summarize the most important parts

If the job is still running, note that the output shown is partial and suggest checking back with `/opencode:status`.

If no job is found: tell the user to run `/opencode:status` to see available jobs.
