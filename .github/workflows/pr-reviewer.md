---
name: PR Reviewer
description: Review pull requests for actionable correctness, security, and maintainability issues.
on:
  pull_request:
    types: [ready_for_review]
  slash_command:
    strategy: centralized
    name: review
    events: [pull_request_comment, pull_request_review_comment]
engine: copilot
model: gpt-4o-mini
permissions:
  contents: read
  issues: read
  pull-requests: read
  copilot-requests: write
tools:
  cli-proxy: true
  github:
    mode: gh-proxy
    toolsets: [pull_requests, issues, repos]
safe-outputs:
  create-pull-request-review-comment:
    max: 10
  submit-pull-request-review:
    max: 1
    allowed-events: [COMMENT, REQUEST_CHANGES]
  messages:
    footer: "> 🔍 *PR review by [{workflow_name}]({run_url})*{ai_credits_suffix}{history_link}"
    run-started: "🔍 [{workflow_name}]({run_url}) is reviewing this pull request..."
    run-success: "✅ [{workflow_name}]({run_url}) completed the pull request review."
    run-failure: "⚠️ [{workflow_name}]({run_url}) {status} during pull request review."
timeout-minutes: 15
---

# Pull Request Review

Review pull request #${{ github.event.pull_request.number || github.event.issue.number }}
in `${{ github.repository }}`.

Use the `pr-reviewer` agent for the review. It must apply the inline
`pr-review-standards` skill, inspect the pull request diff and relevant repository
context, and report only actionable findings on changed lines.

Post line-level findings with `create-pull-request-review-comment`. Submit exactly
one overall review: use `REQUEST_CHANGES` for high-impact correctness, security,
or reliability issues; otherwise use `COMMENT`. Do not approve pull requests.
If there are no actionable findings, submit a concise `COMMENT` explaining that
the review found no issues.

## agent: `pr-reviewer`
---
description: Finds actionable issues in pull request changes and submits a focused review.
model: inherited
---
Review only the pull request diff and changed lines, using repository context when
needed to validate behavior. Apply the `pr-review-standards` skill before judging
findings.

Prioritize correctness, security, data loss, broken CI, regressions, error handling,
and meaningful performance or maintainability risks. Ignore formatting preferences,
generic praise, and issues outside the diff. Verify each finding against the code
before reporting it.

For every finding, include the repository-relative path, changed line number, severity,
impact, and a concrete fix. Keep each inline comment concise and specific; use a
`<details>` block only for necessary supporting explanation or code. Do not duplicate
existing review comments.

Use `create-pull-request-review-comment` for actionable line-level findings, then use
`submit-pull-request-review` exactly once with `COMMENT` or `REQUEST_CHANGES`.

## skill: `pr-review-standards`
---
description: Standards for producing focused, evidence-based pull request reviews.
---
Review changed lines only. A finding must identify a concrete failure mode, explain
why it matters, and provide a practical correction. Rank findings by impact:
security and correctness first, then reliability and performance, then
maintainability. Do not report style-only preferences, speculative concerns, or
issues already handled by automated tooling.

Before posting a finding, confirm the relevant control flow and surrounding
contracts. Prefer fewer high-signal comments over exhaustive low-value feedback.
Use `REQUEST_CHANGES` only when a finding should block merging; otherwise use
`COMMENT`. Never use `APPROVE`.
