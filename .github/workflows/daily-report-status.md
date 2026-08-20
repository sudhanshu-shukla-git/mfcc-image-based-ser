---
name: Daily Report Status
description: Generate a daily activity report in a new issue.
on:
  schedule: daily
  workflow_dispatch:
model: gpt-4o
steps:
  - id: recent
    name: Fetch commits from the last 24 hours
    shell: bash
    run: |
      mkdir -p /tmp/gh-aw
      git log --since='24 hours ago' --format='%h %ad %an %s' --date=iso \
        > /tmp/gh-aw/recent-commits.txt
      {
        echo 'commit_log<<EOF'
        cat /tmp/gh-aw/recent-commits.txt
        echo 'EOF'
      } >> "$GITHUB_OUTPUT"
  - id: issues
    name: Fetch open issues
    shell: bash
    env:
      GH_TOKEN: ${{ secrets.GITHUB_TOKEN }}
    run: |
      mkdir -p /tmp/gh-aw
      gh issue list --state open --limit 100 --json number,title,url,updatedAt \
        > /tmp/gh-aw/open-issues.json
      {
        echo 'issues<<EOF'
        cat /tmp/gh-aw/open-issues.json
        echo 'EOF'
      } >> "$GITHUB_OUTPUT"
permissions:
  contents: read
  issues: read
  pull-requests: read
  copilot-requests: write
tools:
  github:
    mode: gh-proxy
    toolsets: [default]
safe-outputs:
  create-issue:
---

# Daily Report Status

## Task

Use the GitHub tools to fetch the 5 most recent commits and all open issues
labelled `bug` in the current repository. Write a concise daily summary from
that information and post it as a new GitHub issue.

Include source links and the reporting date in the GitHub-flavored Markdown
summary.

Use these workflow-collected sources when preparing the report. Read
`/tmp/gh-aw/recent-commits.txt` for the recent commit log and
`/tmp/gh-aw/open-issues.json` for all open issues before summarizing.

Create exactly one issue using the configured `create-issue` safe output. If there is no meaningful activity in the reporting window, call `noop` with a brief explanation instead.

## Safe Outputs

- Use `create-issue` for the report.
- Call `noop` when no visible issue is needed.
