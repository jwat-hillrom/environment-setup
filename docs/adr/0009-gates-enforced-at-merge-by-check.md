# Gates are enforced at merge time by one `check` command

All Gates run in a deterministic `check` command of the Apply CLI, executed locally by `/ship` and in GitHub Actions as a required status check, so both give the same answer from one versioned implementation. Gates are enforced at merge time only: GitHub rulesets/required checks are the only hard enforcement the hosts offer, while session hooks fail open on timeout and give up after repeated blocks. Reviewer agents report severity-graded findings, never numeric scores; their judgement is advisory, and the only hard rule about it is that Critical findings must carry a resolution note.

## Consequences

- The Work-item-complete and Critical-findings Gates check the recorded `state.json`, which is agent-written: they verify the record, not independent evidence.
- Making `check` required needs a ruleset or branch protection in the Target repo, which Apply cannot set; it's a one-time setup step.
- PR evidence (Gate results, Workflow record, findings) is rendered by `check --format markdown`, never restated by the model; a Gate that cannot run counts as failed unless the Target repo explicitly disables it with a reason.
- Gate tuning lives in a Target-repo-owned Gate config that Apply never writes, so editing it never triggers Apply's local-edit refusal; `check` falls back to Profile defaults when it is absent.
