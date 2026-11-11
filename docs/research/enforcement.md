# Enforcement mechanisms for quality checkpoints (VS Code Copilot harness and Copilot CLI)

Research for issue #10. Researched 2026-11-11 against primary sources only (code.visualstudio.com, docs.github.com). Builds on [host-discovery.md](https://github.com/jwat-hillrom/environment-setup/blob/research/host-discovery/docs/research/host-discovery.md). Every claim links its source.

## Summary

- **Agent hooks are the only first-party, in-session mechanism that can mechanically block or force continuation.** A `preToolUse` hook can deny a tool call; an `agentStop` / `subagentStop` hook can return `decision: "block"` to force another turn ("tests must pass before finishing"). VS Code's Copilot harness and Copilot CLI share one runtime and one hook format, and both read `.github/hooks/*.json` ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks), [hooks reference](https://docs.github.com/en/copilot/reference/hooks-configuration)).
- Hooks have documented holes: **timeouts are always fail-open** (even for `preToolUse` and admin policy hooks), the stop-hook loop is capped at **8 consecutive blocks**, `ask` is treated as deny when no user is present, and repository hooks need folder/workspace trust ([hooks reference](https://docs.github.com/en/copilot/reference/hooks-configuration#exit-codes-for-command-hooks)). VS Code states hooks "are not substitutes for enforced permissions, sandboxing, or enterprise controls" ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_security-considerations)).
- **The only mandatory human-approval gates are on GitHub, outside the editor/CLI**: rulesets/branch protection (required status checks, required approvals, code-owner review, path-scoped required reviewers) and environment required reviewers. Copilot cloud agent PRs are subject to these and cannot approve or merge their own PRs ([risks and mitigations](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/risks-and-mitigations)).
- Custom-agent `tools` lists, tool-approval settings, CLI `--deny-tool`, and sandboxing restrict *capabilities*; they cannot express "require X before completion". MCP elicitation can ask the user mid-turn but is canceled in Autopilot ([VS Code MCP guide](https://code.visualstudio.com/api/extension-guides/ai/mcp#elicitation)).
- **Instructions and skills are advisory**: "Instructions guide the model and are not an enforcement boundary" ([VS Code security](https://code.visualstudio.com/docs/agents/run/security#_hooks)).

| Feature | What it can stop / require | When it runs | VS Code (Copilot harness) | CLI | Key limits | Source |
| --- | --- | --- | --- | --- | --- | --- |
| Hooks: `preToolUse` | Deny, allow, ask, or rewrite args of a tool call | Before each tool call | Yes | Yes | Timeout = fail-open; `ask` = deny when no user/Autopilot; HTTP variant fail-open | [hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#pretooluse-decision-control) |
| Hooks: `agentStop` / `subagentStop` | Force another turn with a reason prompt (`decision: "block"`) | When the agent/subagent ends a turn | Yes | Yes | Overridden after 8 consecutive blocks | [hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#agentstop--subagentstop-decision-control) |
| Hooks: `permissionRequest` | Allow/deny before the permission service; `interrupt` stops the agent | Before permission evaluation | Not documented for VS Code | Yes (CLI only) | Cannot pre-approve sandbox bypass | [hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#permissionrequest-decision-control) |
| Hooks: `postToolUse` | Modify result / inject context (cannot undo) | After a successful tool call | Yes | Yes | Action already happened | [hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#posttooluse-output) |
| Agent-scoped hooks (`hooks` frontmatter) | Same as above, only while that agent is selected | Same events | Yes (preserved on conversion) | Yes | Not run for remote/org-hosted agents; need trust | [VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_agent-scoped-hooks) |
| Custom agent `tools` list | Removes tools from the agent | Session/agent selection | Yes | Yes | Unknown names ignored; `strict-tools-list` not forwarded by VS Code | [custom agents config](https://docs.github.com/en/copilot/reference/custom-agents-configuration#tools-processing) |
| MCP server tools + elicitation | Server logic decides tool result; elicitation asks user for input | During a tool call in an active turn | Yes | Server tools yes; elicitation not documented for CLI | Elicitation canceled in Autopilot, declined with no active turn | [VS Code MCP guide](https://code.visualstudio.com/api/extension-guides/ai/mcp#elicitation) |
| Rulesets / branch protection | Block merge until checks pass / approvals / code owners / path reviewers | At merge (and push rules at push) | n/a (GitHub) | n/a (GitHub) | Bypass lists; admins can configure | [rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets) |
| Environment required reviewers | Pause a workflow job until a reviewer approves | When a job references the environment | n/a | n/a | Plan-dependent for private repos; admin bypass default on | [environments](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments#required-reviewers) |
| Tool approval settings | Require manual confirmation for tools/commands/URLs | Before tool call | Yes (`chat.tools.*`, permission levels) | Yes (`--allow-tool`/`--deny-tool`, permission modes) | Allow all / Autopilot skip prompts; terminal rules best-effort; CLI deny wins over allow | [approvals](https://code.visualstudio.com/docs/agents/run/approvals), [CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#tool-permission-patterns) |
| Sandboxing | Restrict filesystem/network for shell commands | Shell/MCP process execution | Yes | Yes | Bypass prompts unless managed policy forbids | [approvals](https://code.visualstudio.com/docs/agents/run/approvals#_sandbox-agent-commands), [CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#enterprise-managed-sandbox-floor) |
| Instructions, skills | Nothing (guidance only) | Loaded into context | Yes | Yes | Advisory | [VS Code security](https://code.visualstudio.com/docs/agents/run/security#_hooks) |

## 1. Agent hooks

### Hosts and locations

- VS Code's Copilot harness, the GitHub Copilot app and Copilot CLI "share this harness and its hook format"; VS Code points to the GitHub hooks reference for configuration, inputs and outputs ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_choose-the-hook-implementation-for-your-session)). Agent hooks in VS Code are **Preview** ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks)).
- **Repository hooks:** `.github/hooks/*.json` at the repo root, listed as the VS Code "Workspace" source ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_choose-a-hook-source)); CLI loads policy → user → project → plugin hooks and runs all entries for an event ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#hooks-locations)). Also inline `hooks` in `.github/copilot/settings.json` / `settings.local.json` and `.claude/settings*.json` (both hosts).
- **Policy hooks (CLI only):** `C:\ProgramData\GitHub\Copilot\policy.d\*.json` or `/etc/github-copilot/policy.d/*.json`, or HKLM registry; load first, cannot be disabled by `disableAllHooks`, run regardless of folder trust and outside the sandbox ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#policy-hooks)).
- **Cloud agent:** only `.github/hooks/*.json` from the cloned repo; Linux, only `bash`/`command` honored ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#cloud-agent-execution-environment)). VS Code cloud sessions do not copy local hook files ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_choose-the-hook-implementation-for-your-session)).
- Local-harness settings such as `chat.useHooks` and `chat.hookFilesLocations` "do not prevent the Copilot harness from loading a configuration" / "do not configure Copilot discovery" ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_choose-a-hook-source)).

### Events relevant to checkpoints

From the event table ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#hook-events)):

| Event | Can block? | Notes |
| --- | --- | --- |
| `preToolUse` | Yes: `permissionDecision` `allow`/`deny`/`ask`, plus `modifiedArgs` | Any hook returning `deny` blocks. Cloud agent and CLI Autopilot treat `ask` as deny. |
| `permissionRequest` | Yes: `behavior` `allow`/`deny`, `message`, `interrupt: true` stops the agent | CLI only; `allow` cannot pre-approve a sandbox bypass. Exit 2 = deny. |
| `agentStop` | Yes: `decision: "block"` + `reason` forces another turn | Input includes `transcriptPath`, `stop_hook_active`. |
| `subagentStop` | Yes: same, plus `modifiedResponse` | Built-in `general-purpose` agent does not emit subagent events. |
| `postToolUse` | No block; `modifiedResult`, `additionalContext` (≤10 KB combined) | Cannot undo ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_decisions-and-failures)). |
| `postToolUseFailure` | Recovery guidance only | Exit 2 → stdout appended as context. |
| `sessionStart`, `subagentStart` | No; `additionalContext` only | |
| `userPromptSubmitted` | No; command/HTTP output dropped | `modifiedPrompt` honored only by SDK hooks. |
| `sessionEnd`, `errorOccurred`, `preCompact`, `notification` | No | `notification` is CLI-only, fire-and-forget. |

### Input / output format and exit codes

- Command hooks receive one JSON event on **stdin** and may print one JSON object on **stdout**; diagnostics should go elsewhere ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_configure-commands-and-event-data)). Config file: `{"version": 1, "hooks": {"<event>": [ {"type": "command", "bash"|"powershell"|"command": "...", "cwd", "env", "timeoutSec"} ]}}`; CLI also supports shell-free `exec`/`args`, `http` hooks (POST; must be `https://` for `preToolUse`/`permissionRequest`), and `prompt` hooks on `sessionStart` ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#hook-configuration-format)).
- The **event name selects the payload shape**: camelCase (`preToolUse` → `toolName`, `toolArgs`) vs PascalCase VS Code-compatible (`PreToolUse` → `tool_name`, `tool_input`, Claude-style tool names such as `Bash`, `Edit`) ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#hook-event-input-payloads)). Common fields: `sessionId`, `timestamp`, `cwd`; subagent hooks add `agentId`, `agentType`, `parentAgentId`.
- `matcher` (regex anchored `^(?:…)$`) filters `preToolUse`/`postToolUse`/`permissionRequest` by tool name; invalid regex skips the hook ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#matcher-filtering)).
- Stdout must parse as **exactly one** JSON object (progress lines `{"type":"progress",…}` excepted); otherwise it is treated as no output ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#progress-messages)). Output bounded at 10 MiB.
- Exit codes ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#exit-codes-for-command-hooks)):
  - `0`: success; stdout parsed.
  - `2`: warning by default; **deny** for `preToolUse` and `permissionRequest` (even if stdout says allow); context for `postToolUseFailure`.
  - Other non-zero: logged, fail-open — **except `preToolUse`, which fails closed** ("Denied by preToolUse hook (hook errored)").
  - Timeout: killed after `timeoutSec` (default **30 s**); **always fail-open**, including `preToolUse` and policy hooks.
- HTTP `preToolUse` hooks are fail-open on network error, timeout or non-2xx ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#pretooluse--pretooluse)).
- Parallel tool calls: `preToolUse` hooks for different calls run concurrently ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#pretooluse--pretooluse)).

### Continue-before-finishing

- `agentStop` `decision: "block"` "forces another agent turn using `reason` as the prompt"; after **8 consecutive** blocks the CLI ends the turn anyway; `stop_hook_active` lets the hook detect a forced turn ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#agentstop--subagentstop-decision-control)). Under cloud agent the forced turn still counts against the job timeout ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#hook-events)).
- VS Code lists `agentStop` as "Check whether the agent should finish its turn" for the Copilot harness ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_events-and-payloads)).

### Agent-scoped hooks

- `hooks` event map in agent frontmatter, runs only while the agent is selected, in addition to other sources; command paths resolve from repo root; a scoped `Stop` hook runs at `SubagentStop` when the agent is a subagent; "Remote or organization-hosted agent definitions do not run scoped hooks" ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#agent-scoped-hooks)).
- VS Code preserves `hooks` when converting agents for Copilot, including plugin agents; editor validation "doesn't prove that its hooks execute" ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_agent-scoped-hooks)).

### Trust and consent

- CLI: "Repository and inherited-agent hooks require workspace admission" ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#agent-scoped-hooks)). In `-p` mode repository hooks load only if the folder is already trusted, `COPILOT_ALLOW_ALL` is set, or `GITHUB_COPILOT_PROMPT_MODE_REPO_HOOKS=true`; `--trust-folder` trusts hooks for one non-interactive session ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#environment-variables)).
- VS Code: Restricted mode (untrusted workspace) disables agents ([VS Code security](https://code.visualstudio.com/docs/agents/run/security#_recommended-security-baseline)). No hook-specific consent prompt for the Copilot harness is documented (see Open uncertainties).
- Opt-out: `disableAllHooks: true` in a hook file skips that file; at the top of repo `settings.json` (CLI only) it skips every non-policy hook — so a contributor can disable repo hooks via `settings.local.json` ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#disable-all-hooks)).
- Sandbox: with the CLI session sandbox on, repo/user/plugin hooks run inside it ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#sandboxed-sessions)).

## 2. Custom agents with a limited `tools` list

- `tools` "filters the set of tools that are made available to the agent – whether built-in or sourced from MCP servers"; omitted = all, `[]` = none ([custom agents config](https://docs.github.com/en/copilot/reference/custom-agents-configuration#tools-processing)). CLI: `*` anywhere grants every tool ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#custom-agent-frontmatter-fields)).
- "All unrecognized tool names are ignored" — a typo silently drops a tool rather than failing ([custom agents config](https://docs.github.com/en/copilot/reference/custom-agents-configuration#tools)).
- Aliases (`execute`, `read`, `edit`, `search`, `agent`, `web`, `todo`) map to harness tools ([custom agents config](https://docs.github.com/en/copilot/reference/custom-agents-configuration#tool-aliases)). If `execute` (shell) is allowed, file writes can still happen through the shell (the alias table defines no shell restriction).
- `strict-tools-list`: "Copilot schema field for omitting standard required tools that aren't explicitly listed"; VS Code **does not forward** `strict-tools-list` (nor `mcp-servers`, `github`, `model-policy`, `deferred-tool-loading`, `include-custom-instructions`) when converting agents, and says not to rely on these fields ([VS Code agent reference](https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference#_copilot-specific-configuration)).
- Selecting an agent is a user/agent choice; a user can pick another agent. The tool list is not a repository-wide control. Tool availability is "separate from tool approval" ([VS Code tools](https://code.visualstudio.com/docs/agents/run/tools#_manage-tool-availability-for-the-copilot-harness)).
- CLI session-wide equivalents: `--available-tools` (only these tools) and `--excluded-tools` ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#command-line-options)).
- CLI plan mode blocks project-file edits but is "a safety net, not a guarantee" (shell commands with unclear effects and MCP tools pass) ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#plan-mode)).

## 3. MCP servers

- **Decline:** The Copilot runtime invokes server tools "under the session's tool availability and approval policies"; the server controls its tool result ([VS Code MCP guide](https://code.visualstudio.com/api/extension-guides/ai/mcp#mcp-support-in-copilot)). The allowed sources do not describe a protocol-level "deny" verb for a server; refusing means returning a result/error the model sees (inference — see Open uncertainties).
- **Elicitation:** "Copilot supports form and URL elicitation during an active turn. Requests are canceled in Autopilot and declined when there is no active turn." Servers must handle both outcomes "without continuing the action that needs input" ([VS Code MCP guide](https://code.visualstudio.com/api/extension-guides/ai/mcp#elicitation)). Users are told not to enter passwords in these forms ([VS Code MCP servers](https://code.visualstudio.com/docs/agent-customization/mcp-servers#_use-your-servers-tools)). CLI fires a `notification` hook of type `elicitation_dialog`, implying CLI support ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#notification-hook)).
- **Sampling** supported during an active turn with per-server approval; Autopilot requires prior approval ([VS Code MCP guide](https://code.visualstudio.com/api/extension-guides/ai/mcp#sampling)).
- **Tool annotations** "describe behavior, not permission grants"; don't assume `readOnlyHint` bypasses approval ([VS Code MCP guide](https://code.visualstudio.com/api/extension-guides/ai/mcp#tools)).
- **State:** Not described by the sources. A stdio server is a local process the host starts ([MCP configuration](https://code.visualstudio.com/docs/agents/reference/mcp-configuration)); state across calls/sessions is the server's own implementation. VS Code notes "Stopping a server only ends its current run, and a later request can start it again" ([VS Code MCP servers](https://code.visualstudio.com/docs/agent-customization/mcp-servers#_manage-an-existing-server)).
- **Trust:** CLI loads workspace `.mcp.json`/`.github/mcp.json` only after folder trust ([CLI MCP](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers#adding-per-repository-mcp-servers)); VS Code workspace `.mcp.json` inherits Workspace Trust ([VS Code security](https://code.visualstudio.com/docs/agents/run/security#_trust-boundaries)). Enterprise allowlist is fail-closed in CLI ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#enterprise-mcp-allowlist)). VS Code: sandboxed stdio MCP servers' tool calls are auto-approved ([VS Code security](https://code.visualstudio.com/docs/agents/run/security#_mcp-server-sandboxing)).
- MCP tool invocations are subject to the session's permission rules ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#mcp-server-trust-levels)).

## 4. GitHub repository settings

- **Required status checks:** "all required status checks must pass before collaborators can merge"; can be pinned to a specific GitHub App as source; strict/loose up-to-date mode ([rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#require-status-checks-to-pass-before-merging)).
- **Required approvals / code owners:** required approving reviews from writers or code owners; dismiss stale approvals; require approval from someone other than the last pusher; require conversation resolution ([rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#additional-settings)).
- **Path-scoped required reviewers:** up to 15 teams, each with file patterns (gitignore syntax) and 0–10 approvals — directly usable for "a human must approve schema changes"; not available on user-owned repos ([rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#required-reviewers)).
- **Push rules:** restrict file paths/extensions/size (push rulesets, private/internal repos) ([rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#restrict-file-paths)). Also code scanning, code quality, coverage and secret-scanning merge gates.
- **Bypass:** rulesets can grant bypass to roles, teams or apps ([rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets)).
- **Environments:** required reviewers (up to 6, any one approves), optional "prevent self-reviews", wait timer, branch restrictions, custom protection rules; admins can bypass by default unless disallowed; private-repo availability depends on plan ([environments](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments#deployment-protection-rules)).
- **Copilot cloud agent PRs** ([risks and mitigations](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/risks-and-mitigations#copilot-cloud-agent-can-push-code-changes-to-your-repository)):
  - pushes only to its own `copilot/` branch (or the PR branch it was asked to work on) and is "subject to any branch protections and required checks";
  - cannot mark PRs ready, approve or merge; the requesting user cannot approve the PR;
  - Actions workflows do not run until a writer clicks **Approve and run workflows** (configurable);
  - unattributed Copilot PRs need **one extra approval** (default on in rulesets, always in branch protection) ([rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#additional-approval-for-unattributed-copilot-pull-requests));
  - a ruleset can prevent Copilot from creating/updating PRs; admins can grant Copilot a bypass ([about cloud agent](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/about-cloud-agent#repository-compatibility)).
- These gates apply to any PR regardless of whether it came from VS Code or CLI, because they act at merge time on GitHub.

## 5. Tool-approval and sandbox settings

### VS Code (Copilot harness)

- Permission levels: Manual (default, uses approval settings), Assisted (LLM judge), Allow all; Autopilot is a mode that auto-approves all tools and auto-answers questions; default via `chat.permissions.default` ([approvals](https://code.visualstudio.com/docs/agents/run/approvals#_permission-levels)). Worktree sessions in the Agents window "always use Allow all".
- `chat.tools.eligibleForAutoApproval` (`false` = always manual), `chat.tools.terminal.autoApprove` (a `false` rule "requires approval. It does not block the command"), `chat.tools.urls.autoApprove`, `chat.tools.edits.autoApprove` for sensitive files, `chat.tools.global.autoApprove` ([approvals](https://code.visualstudio.com/docs/agents/run/approvals), [VS Code security](https://code.visualstudio.com/docs/agents/run/security#_recommended-security-baseline)).
- In Copilot harness sessions the runtime can approve read-only or sandboxed commands **before** terminal rules are evaluated, even with a matching `false` rule; terminal auto-approval "is a best-effort convenience, not a security boundary" ([approvals](https://code.visualstudio.com/docs/agents/run/approvals#_automatically-approve-terminal-commands)).
- "Always Allow" grants sync to **user** settings; "Workspace settings are not copied" to the Agent Host ([approvals](https://code.visualstudio.com/docs/agents/run/approvals#_scope-approvals-in-copilot-sessions)).
- Enterprise policies (`ChatToolsAutoApprove`, `ChatToolsEligibleForAutoApproval`, `ChatToolsTerminalEnableAutoApprove`, `ChatMCP`, `ChatAgentMode`) — but Copilot harness sessions "don't fully enforce some of these" (e.g. `ChatToolsEligibleForAutoApproval` removes Allow all but Assisted and saved approvals can still approve) ([VS Code security](https://code.visualstudio.com/docs/agents/run/security#_enterprise-policies)).
- Sandboxing (`chat.agent.sandbox.enabled`) restricts terminal FS/network independently of permission level; managed settings can require it ([approvals](https://code.visualstudio.com/docs/agents/run/approvals#_sandbox-agent-commands)).

### Copilot CLI

- `--allow-tool` / `--deny-tool` with patterns `shell(git:*)`, `write(path)`, `read`, `url(...)`, `memory`, `ServerName(tool)`; "Deny rules always take precedence over allow rules, even when `--allow-all` is set"; `write` deny matches exact/trailing path, no globs yet ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#tool-permission-patterns)).
- `--allow-all-tools`: "Deny rules and enterprise restrictions still apply". `permissions.disableBypassPermissionsMode: "disable"` (user, managed or MDM) suppresses all allow-all options; unknown values enforce `disable` ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#restricting-the-allow-all-options)).
- Interactive approvals can be saved per location or always ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#permission-approval-responses)). Skill `allowed-tools` pre-approves tools when the skill is active ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#skill-frontmatter-fields)).
- Sandbox: `--sandbox`/`--no-sandbox`; managed `sandbox.enabled` floor; `sandbox.allowBypass: false` prevents opt-out; `sandbox.failIfUnavailable` blocks execution if the sandbox can't be established ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#enterprise-managed-sandbox-floor)).

## Advisory only

- Custom instructions (`AGENTS.md`, `.github/copilot-instructions.md`, `.instructions.md`): "Instructions guide the model and are not an enforcement boundary" ([VS Code security](https://code.visualstudio.com/docs/agents/run/security#_hooks)). CLI `--no-custom-instructions` removes them entirely ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#command-line-options)).
- Skills: content "is injected into the conversation" ([CLI ref](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-command-reference#skills-reference)); `allowed-tools` loosens rather than tightens approval.
- Custom agent bodies: instructions only; only the frontmatter `tools`/`hooks` fields have mechanical effect.
- Cloud agent's built-in CodeQL/secret-scanning/code-review pass "attempts to resolve issues" but can be disabled ([risks and mitigations](https://docs.github.com/en/copilot/concepts/agents/cloud-agent/risks-and-mitigations#unvalidated-code-can-introduce-vulnerabilities)).

## Implications for checkpoints

Facts only, mapped to the two example checkpoints.

**"Tests must pass before the agent reports completion"**

- An `agentStop` hook can run tests and return `decision: "block"` with the failure as `reason`; both hosts support it via `.github/hooks/*.json` ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#agentstop--subagentstop-decision-control)).
- The block is overridden after 8 consecutive blocks; a timed-out hook (default 30 s) is fail-open; hooks need folder/workspace trust and can be disabled via repo `settings.local.json` `disableAllHooks` (CLI) ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#disable-all-hooks)).
- The merge-time equivalent that cannot be bypassed by the agent is a required status check in a ruleset ([rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#require-status-checks-to-pass-before-merging)).

**"A human must approve schema changes"**

- In-session: a `preToolUse` hook with `matcher` on edit tools can return `ask` or `deny` for schema paths. `ask` becomes deny under cloud agent and CLI Autopilot, and VS Code says not to assume `ask` pauses an unattended session ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#pretooluse-decision-control), [VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_decisions-and-failures)). Shell-based writes need the hook to inspect shell tool args too ([VS Code security](https://code.visualstudio.com/docs/agents/run/security#_hooks): "A filter that misses a tool or argument shape does not validate that operation").
- VS Code `chat.tools.edits.autoApprove` with `"<glob>": false` requires manual approval for edits to matching files, but Allow all/Autopilot skip prompts ([approvals](https://code.visualstudio.com/docs/agents/run/approvals#_permission-levels)).
- At merge: ruleset **Required reviewers** with file patterns, or CODEOWNERS + "require review from code owners" ([rulesets](https://docs.github.com/en/repositories/configuring-branches-and-merges-in-your-repository/managing-rulesets/available-rules-for-rulesets#required-reviewers)). For deployment of schema migrations: environment required reviewers with "prevent self-reviews" ([environments](https://docs.github.com/en/actions/reference/workflows-and-actions/deployments-and-environments#required-reviewers)).

**General**

- Hooks, approvals and sandboxing are local, per-machine and user-overridable unless reinforced by CLI policy hooks / managed settings or VS Code enterprise policy. GitHub rulesets and environments are the only server-enforced gates in scope.
- One hook file serves both hosts if it uses one event-name style consistently, because event name selects payload shape ([VS Code hooks](https://code.visualstudio.com/docs/agent-customization/hooks#_events-and-payloads)).
- Use `command`/`powershell`+`bash` so the same hook works on Windows and in cloud agent (Linux, `bash` only) ([hooks ref](https://docs.github.com/en/copilot/reference/hooks-configuration#command-hooks)).

## Open uncertainties

1. Whether VS Code's Copilot harness shows its own consent/trust prompt specifically for repository hooks (beyond Workspace Trust) is not documented; CLI requires "workspace admission".
2. `permissionRequest` is documented as CLI-only for cloud agent comparison; whether it fires in VS Code's Copilot harness (shared runtime) is not stated.
3. Whether the 8-block runaway guard applies identically in VS Code is not stated (the reference says "the CLI overrides").
4. `strict-tools-list` semantics in Copilot CLI are not described in the fetched CLI frontmatter table; only VS Code's one-line definition and its non-forwarding note were found.
5. MCP: no allowed source documents a server-side "deny" primitive or session-state guarantees; refusing a call and keeping state are server implementation concerns (inference). CLI elicitation support is implied only by the `elicitation_dialog` notification type.
6. VS Code links `docs.github.com/.../hooks-reference`; this research fetched `.../hooks-configuration` (same title, "GitHub Copilot hooks reference"). Assumed to be the same page.
7. Whether "Always Allow" grants in CLI (`Config file`) can be overridden by a committed repo `settings.json` deny rule is not stated in the fetched pages.
