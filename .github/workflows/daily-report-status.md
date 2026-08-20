---
name: Daily Report Status
description: Generate a daily activity report in a new issue.
on:
  schedule: daily
  workflow_dispatch:
permissions:
  contents: read
  issues: read
  copilot-requests: write
tools:
  github:
    mode: gh-proxy
    toolsets: [repos, issues]
safe-outputs:
  create-issue:
---

# Daily Report Status

## Task

Generate an activity report in a new GitHub issue for the current repository.

Use the reporting window of the previous 24 hours ending at the workflow run start time. Summarize relevant repository activity, including newly opened, updated, and closed issues and notable commits. Include the reporting window and source links in the report, and write it as concise GitHub-flavored Markdown.

Create exactly one issue using the configured `create-issue` safe output. If there is no meaningful activity in the reporting window, call `noop` with a brief explanation instead.

## Safe Outputs

- Use `create-issue` for the report.
- Call `noop` when no visible issue is needed.
