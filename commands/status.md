---
description: Show status of OpenCode background jobs. Optionally pass a job ID to check a specific job.
---

# OpenCode: Job Status

Check the status of OpenCode background jobs.

## Parse arguments

- If `$ARGUMENTS` contains a job ID: run `opencode-jobs status [JOB_ID]`
- If `$ARGUMENTS` is empty: run `opencode-jobs list`

## Execute

Run the appropriate command and display the results.

For `list` output: present as a readable table.

For `status [JOB_ID]` output: present the job details clearly, including whether it is still running or complete.

If a job shows as complete, suggest: "Run `/opencode:result [JOB_ID]` to get the full output."

If no jobs are found: tell the user "No background jobs yet. Use `/opencode:run --background [task]` to start one."
