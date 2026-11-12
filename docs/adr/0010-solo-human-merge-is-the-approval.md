# In a solo repo, the human merge is the approval

An agent working locally acts with the user's own GitHub credentials, so GitHub cannot tell the user and the agent apart, and a user cannot approve their own PR. So approval-required changes (detected deterministically by `check` from the diff) are approved by the human performing the merge: a ruleset requires PRs into the default branch and blocks force-pushes, and an authored `preToolUse` hook denies `gh pr merge`, pushes to the default branch and force-pushes in agent sessions. Where author and reviewer differ (teams, Copilot cloud agent PRs), the same path globs drive CODEOWNERS plus a ruleset requiring code-owner review.

## Consequences

- The hook is a guardrail only (needs folder trust, can be disabled locally, fails open on timeout); the ruleset is the hard control.
- Authored hooks are Node scripts (`node .github/hooks/scripts/<name>.mjs`) referenced from `.github/hooks/<name>.json`, so they run on Windows, macOS and Linux.
