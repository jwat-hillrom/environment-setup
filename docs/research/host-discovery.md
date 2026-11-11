# Host discovery of Components (VS Code Copilot agent mode vs GitHub Copilot CLI)

Research for issue #2. Researched 2026-11-11 against primary sources only (code.visualstudio.com, docs.github.com). Every claim links its source.

## Summary answer

- **Every Component type can live in the repo**, but not every host reads every type. All five types (custom agents, skills, MCP servers, instruction files, prompt files) have a repo-level location in VS Code; **prompt files are not supported by Copilot CLI** ([cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet#ide-and-surface-support)).
- **VS Code now has two relevant "harnesses".** The **Copilot harness** (default, built on the Copilot SDK shared with Copilot CLI) and the **deprecated Local harness** (VS Code's built-in agent, removed by **February 2027**) ([agent harnesses](https://code.visualstudio.com/docs/agents/run/agent-harnesses)). Discovery rules depend on the harness, not the model ([custom instructions reference](https://code.visualstudio.com/docs/agents/reference/custom-instructions-reference#_supported-formats-and-locations)). Prompt files only run in the Local harness ([prompt files](https://code.visualstudio.com/docs/agent-customization/prompt-files)), so they are effectively end-of-life; VS Code recommends converting them to skills.
- **Canonical shared repo paths** that work in both VS Code (Copilot harness) and Copilot CLI:
  - Agents: `.github/agents/*.agent.md`
  - Skills: `.github/skills/<name>/SKILL.md`
  - Instructions: `.github/copilot-instructions.md`, `AGENTS.md` (incl. nested), `.github/instructions/**/*.instructions.md`
  - MCP: workspace-root `.mcp.json` with a top-level `mcpServers` object. (CLI also reads `.github/mcp.json`; VS Code's legacy `.vscode/mcp.json` uses a different `servers` schema and is **not** read by the CLI.)
- **Plugins/marketplaces at repo level: yes.** A committed `.github/copilot/settings.json` with `extraKnownMarketplaces` + `enabledPlugins` is read by Copilot CLI (auto-installs and activates the plugin, scoped to that repo), by the VS Code Copilot harness (as "repository plugin and marketplace defaults"), and by Copilot in the cloud. `.claude/settings.json` is also read by both. Users can opt out via an uncommitted `.github/copilot/settings.local.json`.
- **User-level only:** nothing among the five Component types is strictly user-only. What is user-only is plugin *installation state* (`~/.copilot/installed-plugins/`), the VS Code settings `chat.plugins.marketplaces` / `chat.pluginLocations` (documented for user/remote `settings.json`), Local-harness profile-stored prompts/instructions/agents, and organization-level agents/instructions (managed on GitHub, not in a repo file).

## Host 1: VS Code Copilot (agent mode)

Unless noted, rows describe the **Copilot harness** (default). Local-harness differences are called out because the Local harness still exists until Feb 2027.

| Component type | Repo-level path(s) | Format / frontmatter | Notes | Source |
| --- | --- | --- | --- | --- |
| Custom agent | `.github/agents/` (`.agent.md` or `.md`); `.claude/agents/` (Claude format `.md`) | Optional YAML frontmatter + Markdown body. Fields: `name` (defaults to file name), `description` (**required by Copilot schema**), `tools`, `model` (string or ordered array), `reasoning-effort`, `skills`, `user-invocable`, `disable-model-invocation`, `target` (`vscode` / `github-copilot`), `hooks`, plus Copilot-schema fields (`model-policy`, `mcp-servers`, `github`, `strict-tools-list`, `include-custom-instructions`, `deferred-tool-loading`). Copilot harness rejects `handoffs`, `agents`, `argument-hint`, `advancedOptions` (Local-only). | When VS Code converts agents for the Copilot runtime (incl. plugin agents) it does **not** forward `model-policy`, `mcp-servers`, `github`, `deferred-tool-loading`, `strict-tools-list`, `include-custom-instructions`. `.chatmode.md` is legacy, rename to `.agent.md`. `chat.agentFilesLocations` adds locations (still supported, forwarded to the Copilot harness, but not inherited by Copilot CLI). Org-level agents toggled by `github.copilot.chat.organizationCustomAgents.enabled`. | [Custom agent file reference](https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference) |
| Skill | `.github/skills/<name>/SKILL.md` (documented Copilot location). Local harness also reads `.claude/skills/`, `.agents/skills/`. | YAML frontmatter: `name` (**required**, lowercase/digits/hyphens, ≤64 chars, **must match parent dir**, no namespace prefixes, else silently skipped), `description` (**required**, ≤1024 chars). Optional: `argument-hint`, `user-invocable`, `disable-model-invocation`, `context: fork` (experimental; needs `github.copilot.chat.skillTool.enabled`). Directory may include scripts/resources. | Agent Skills open standard (agentskills.io). `chat.agentSkillsLocations` is **deprecated** (Local only; forwarded to other harnesses for compatibility). | [Agent skills](https://code.visualstudio.com/docs/agent-customization/agent-skills) |
| MCP server | Workspace-root `.mcp.json` (portable `mcpServers` format). Legacy: `.vscode/mcp.json` (`servers` + `inputs` format, Local harness). | `{"mcpServers": {"<name>": {"type": "stdio"\|"http"\|"sse", "command", "args", "env", "cwd", "url", "headers", "tools", "oauthClientId"}}}`. Env refs `${NAME}` / `${NAME:-default}`. No `inputs`, `envFile`, `dev`, `sandbox*` in portable format. | For new Copilot configs use `.mcp.json`, not `.vscode/mcp.json`. `.vscode/mcp.json` entries can be *forwarded* to the Copilot harness unless they use `${input:...}`/`${command:...}`. `chat.mcp.workspaceRootConfig.enabled` (experimental) only controls whether guided setup *offers* `.mcp.json`; existing files load regardless. `chat.mcp.access` governs which servers can be used. | [MCP configuration reference](https://code.visualstudio.com/docs/agents/reference/mcp-configuration) |
| Instructions (repo-wide) | `.github/copilot-instructions.md` or `AGENTS.md` (repo root) | Plain Markdown, **no frontmatter required**. | Copilot harness discovers `AGENTS.md` at repo root, working dir, intermediate dirs, and along paths to files it works with. Local harness: root `AGENTS.md` needs `chat.useAgentsMdFile`; nested needs `chat.useNestedAgentsMdFiles` (experimental, off). Local also reads `CLAUDE.md` (`chat.useClaudeMdFile`). | [Custom instructions reference](https://code.visualstudio.com/docs/agents/reference/custom-instructions-reference) |
| Instructions (scoped) | `.github/instructions/**/*.instructions.md` (searched recursively); nested `AGENTS.md` | Optional YAML: `name`, `description`, `applyTo` (comma-separated globs relative to workspace root; no default). `excludeAgent` is a GitHub-feature field. | In Copilot sessions `applyTo` targets files the agent works with. `description`-based on-demand loading is Local-only. `chat.instructionsFilesLocations` adds locations (not inherited by CLI). Local-only toggles: `chat.includeApplyingInstructions`, `chat.includeReferencedInstructions`, `github.copilot.chat.codeGeneration.useInstructionFiles`. | [Custom instructions reference](https://code.visualstudio.com/docs/agents/reference/custom-instructions-reference#_file-syntax) |
| Prompt file | `.github/prompts/*.prompt.md` | Optional YAML: `name`, `description`, `argument-hint`, `agent`, `model`, `tools`. Must end `.prompt.md`. | **Local harness only**; not runnable in the Copilot harness. Local harness removed by Feb 2027, so convert to skills. `chat.promptFilesRecommendations` surfaces prompts. `chat.useCustomizationsInParentRepositories` for monorepos (Local). | [Prompt files](https://code.visualstudio.com/docs/agent-customization/prompt-files) |

Cross-cutting VS Code settings: `chat.plugins.enabled` (master switch for plugins), `chat.useCustomizationsInParentRepositories` (monorepo parent discovery, Local harness only) ([agent plugins](https://code.visualstudio.com/docs/agent-customization/agent-plugins), [custom instructions reference](https://code.visualstudio.com/docs/agents/reference/custom-instructions-reference#_additional-search-locations)).

## Host 2: GitHub Copilot CLI

| Component type | Repo-level path(s) | Format / frontmatter | Notes | Source |
| --- | --- | --- | --- | --- |
| Custom agent | `.github/agents/` and `.claude/agents/` in the project, plus parent dirs (monorepo) and `--add-dir` roots | `.agent.md` (or `.md`). YAML: `description` (**required**), `name` (optional display name), `tools`, `model`, `target`, `disable-model-invocation`, `user-invocable`, `mcp-servers`, `metadata`; CLI also documents `model-policy`, `strict-tools-list`, `include-custom-instructions`. Body ≤30,000 chars. | Agent **ID = file name** minus extension (subdirs joined with `--`). **Personal `~/.copilot/agents/` wins over project on same ID** (first-found order: user, project `.github`, parents, project `.claude`, …, plugins, remote org). Plugin agents with a clashing ID are silently ignored. `customAgents.defaultLocalOnly` disables remote org agents. | [Create custom agents for CLI](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/create-custom-agents-for-cli), [Custom agents configuration](https://docs.github.com/en/copilot/reference/custom-agents-configuration), [CLI plugin reference: loading order](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference#loading-order-and-precedence) |
| Skill | `.github/skills/`, `.agents/skills/`, `.claude/skills/` (each `<name>/SKILL.md`), plus parent dirs | YAML: `name` (**required**, lowercase-hyphen, typically = dir name), `description` (**required**), `license` (optional), `allowed-tools` (optional pre-approval, e.g. `shell`). | **Project skills win over personal** on same name (opposite of agents). Settings: `disabledSkills`, `ignoredSkillsLocations`, `skillDirectories`; env `COPILOT_SKILLS_DIRS`; `/skills` command enables/disables. | [Add skills (CLI)](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-skills), [CLI config dir reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference#user-settings-copilotsettingsjson) |
| MCP server | `.mcp.json` (any dir from cwd up to repo root) and `.github/mcp.json` | Top-level `mcpServers` object **or** bare map of server names. Types `local`/`stdio`, `http`, `sse`; fields `command`, `args`, `env`, `url`, `headers`, `tools`. | `.mcp.json` beats `.github/mcp.json` in same dir; closer-to-cwd wins; project beats `~/.copilot/mcp-config.json`. **Only loaded after folder trust**; in `-p` mode untrusted dirs skip them unless `GITHUB_COPILOT_PROMPT_MODE_WORKSPACE_MCP=true`. **`.vscode/mcp.json` is not read.** Repo `disabledMcpServers` can disable. | [Add MCP servers (CLI)](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-mcp-servers#adding-per-repository-mcp-servers) |
| Instructions (repo-wide) | `.github/copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md` (also `.claude/CLAUDE.md`), `GEMINI.md`, discovered at repo root, cwd, intermediate dirs, and dirs on the path to files being worked on | Plain Markdown. `@relative/path` includes other files (in repo only) in `copilot-instructions.md`, `AGENTS.md`, `CLAUDE.md`. | All applicable files are combined; no precedence order. `/instructions` toggles individual files. `COPILOT_CUSTOM_INSTRUCTIONS_DIRS` adds dirs. `--no-custom-instructions` exists (see CLI command reference). Edits need a new/resumed session. | [Add custom instructions (CLI)](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/add-custom-instructions) |
| Instructions (scoped) | `.github/instructions/**/*.instructions.md` (standard locations except intermediate dirs) | YAML `applyTo` (comma-separated globs); optional `excludeAgent` (`code-review` / `cloud-agent`). | Applied only when `applyTo` matches a file being worked on. No `@` file expansion in these files. | same as above |
| Prompt file | **Not supported** | n/a | Cheat sheet marks prompt files ✗ for Copilot CLI. | [Customization cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet#ide-and-surface-support) |

Related repo-level CLI files (not Components but relevant to Apply): `.github/hooks/*.json` (hooks), `.github/copilot/settings.json` (repo settings), `.github/copilot/settings.local.json` (uncommitted overrides), `.github/allowed_models.txt` ([CLI config dir reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference#repository-settings-githubcopilotsettingsjson)).

## Plugins and marketplaces at repo level

**Yes — both hosts support a committed file that declares marketplaces and enables plugins for the repo.**

File: `.github/copilot/settings.json` (also `.claude/settings.json` for the cross-tool subset):

```json
{
  "extraKnownMarketplaces": {
    "company-tools": { "source": { "source": "github", "repo": "your-org/plugin-marketplace" } }
  },
  "enabledPlugins": { "code-formatter@company-tools": true }
}
```

- Marketplace key must equal the `name` in that marketplace's `marketplace.json`; plugin IDs are `<plugin>@<marketplace>` ([VS Code plugin reference: workspace plugin settings](https://code.visualstudio.com/docs/agents/reference/agent-plugin-reference#_workspace-plugin-settings)).
- **Copilot CLI:** only a whitelist of keys is honored at repo level; `enabledPlugins` and `extraKnownMarketplaces` are merged (repo overrides user per key). A plugin enabled only via repo `enabledPlugins` **auto-installs and activates in that repo only** and stays disabled globally; leaving the repo deactivates it. Repo-level `autoUpdate: true` on a marketplace is accepted but **ignored**. These keys are also read by Copilot in the cloud. Opt out with `false` in `.github/copilot/settings.local.json`. Precedence: defaults → MDM → user → repo → local → env → flags; managed plugin entries always win ([CLI config dir reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference#repository-settings-githubcopilotsettingsjson), [finding/installing plugins](https://docs.github.com/en/copilot/how-tos/copilot-cli/customize-copilot/plugins-finding-installing#turn-off-a-repository-default)).
- **VS Code:** Copilot harness reads `.github/copilot/settings.json` as "repository plugin and marketplace defaults"; users can turn optional plugins off in the Plugins UI without changing shared defaults. Local harness treats these as *recommendations* (notification on first chat message, user installs via Browse). Enterprise settings override both ([agent plugins: workspace recommendations](https://code.visualstudio.com/docs/agent-customization/agent-plugins#_workspace-plugin-recommendations)). Plugins require `chat.plugins.enabled`.
- VS Code also discovers plugins installed by Copilot CLI under `~/.copilot/installed-plugins/` ([agent plugins](https://code.visualstudio.com/docs/agent-customization/agent-plugins#_plugins-installed-by-github-copilot-cli)).
- Plugin package formats: Agent Plugins 1.0 (`plugin.json` with `$schema` `https://agent-plugins.org/schemas/1.0.0/plugin.schema.json`; CLI also accepts 1.1.0) where only **skills** (`skills/`) and **MCP** (`mcp.json`) are portable and agents/hooks/commands/rules live under `com.github.copilot/`; legacy Copilot (`plugin.json`, `.plugin/`, `.github/plugin/`), Claude (`.claude-plugin/plugin.json`) ([VS Code plugin reference](https://code.visualstudio.com/docs/agents/reference/agent-plugin-reference#_plugin-formats), [CLI plugin reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference#pluginjson)).
- Marketplaces are Git repos with `marketplace.json` (in root, `.plugin/`, `.github/plugin/` or `.claude-plugin/`). Pin plugin sources with `sha` (40-char) for reproducibility ([CLI plugin reference: marketplace.json](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference#marketplacejson)). VS Code `chat.plugins.marketplaces` accepts `owner/repo#<sha>` ([VS Code plugin reference](https://code.visualstudio.com/docs/agents/reference/agent-plugin-reference#_marketplace-source-formats)).
- Built-in marketplace: CLI ships `awesome-copilot` (not removable); `github/copilot-plugins` is no longer default ([CLI plugin reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference#copilot-plugin-update-options)).

## Host differences

| Topic | VS Code (Copilot harness) | Copilot CLI |
| --- | --- | --- |
| Prompt files | Local harness only (deprecated) | Not supported |
| Agent dirs | `.github/agents`, `.claude/agents` | Same, plus parent dirs and `--add-dir` |
| Agent precedence | Not documented for VS Code beyond picker | User beats project on same ID |
| Skill dirs | `.github/skills` documented; Local also `.claude/skills`, `.agents/skills` | `.github/skills`, `.agents/skills`, `.claude/skills`; project beats personal |
| Instruction files | `.github/copilot-instructions.md`, `AGENTS.md` (nested), `.github/instructions` | Same plus `CLAUDE.md`, `.claude/CLAUDE.md`, `GEMINI.md`; `@file` includes |
| MCP repo file | `.mcp.json` (portable); legacy `.vscode/mcp.json` forwarded | `.mcp.json` (walk up to root) + `.github/mcp.json`; **ignores** `.vscode/mcp.json` |
| MCP gating | Workspace trust prompt; `chat.mcp.access` | Folder trust required |
| Extra-location settings | `chat.agentFilesLocations`, `chat.instructionsFilesLocations`, `chat.agentSkillsLocations` (deprecated) | `skillDirectories`, `COPILOT_SKILLS_DIRS`, `COPILOT_CUSTOM_INSTRUCTIONS_DIRS`; does **not** read VS Code settings |
| Agent frontmatter | Copilot schema; some fields not forwarded on conversion | Full Copilot schema incl. `mcp-servers`, `model-policy` |
| Repo plugin defaults | Read from `.github/copilot/settings.json`; Local treats as recommendation | Auto-install + activate, repo-scoped |

VS Code explicitly warns that custom-location settings aren't portable because "GitHub Copilot CLI and the GitHub Copilot app don't inherit editor settings" ([custom agent file reference](https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference)).

## User-level only

None of the five Component types is user-only; all have a repo location (prompt files only for VS Code Local). Things that are user/org-level only:

- Installed plugin files and enablement state (`~/.copilot/installed-plugins/`, VS Code plugin enablement stored outside shared workspace settings) ([CLI config dir](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference#installed-plugins), [agent plugins](https://code.visualstudio.com/docs/agent-customization/agent-plugins#_enable-or-disable-plugins)).
- Marketplace `autoUpdate` opt-in (repo value ignored) ([CLI config dir](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference#repository-settings-githubcopilotsettingsjson)).
- Local-harness profile-stored prompts/instructions/agents ([custom instructions reference](https://code.visualstudio.com/docs/agents/reference/custom-instructions-reference#_user-instructions)).
- Organization-level agents and instructions (managed on GitHub) ([custom agent file reference](https://code.visualstudio.com/docs/agents/reference/custom-agent-file-reference)).
- CLI user-only settings: `skillDirectories`, `ignoredSkillsLocations`, `customAgents.defaultLocalOnly`, sandbox settings (only whitelisted keys are repo-overridable) ([CLI config dir](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-config-dir-reference#repository-settings-githubcopilotsettingsjson)).

## Version requirements

- VS Code Local harness deprecated, removed by **February 2027**; admin controls `vscode.temporaryDefaultToLocalHarness` (≥1.142) and `vscode.temporaryAllowLocalHarness` (≥1.143) ([agent harnesses](https://code.visualstudio.com/docs/agents/run/agent-harnesses)).
- Agent Plugins manifest `$schema` v1.0.0 (VS Code, CLI) and v1.1.0 (CLI); unsupported versions are rejected by CLI ([CLI plugin reference](https://docs.github.com/en/copilot/reference/copilot-cli-reference/cli-plugin-reference#pluginjson)).
- GitHub recommends latest stable CLI/IDE versions; no minimum CLI version is stated for these features ([cheat sheet](https://docs.github.com/en/copilot/reference/customization-cheat-sheet#ide-and-surface-support)).

## Open uncertainties

1. VS Code docs list only `.github/skills/` as the Copilot-harness project location; whether the Copilot harness also reads `.agents/skills/` and `.claude/skills/` (as CLI does, sharing the same SDK) is inferred, not stated.
2. Whether VS Code's `chat.plugins.marketplaces` / `chat.pluginLocations` can be set in workspace `.vscode/settings.json` is not documented; examples only show user/remote settings. Prefer `.github/copilot/settings.json`.
3. VS Code docs don't state whether the Copilot harness *auto-installs* repo `enabledPlugins` (as CLI does) or only enables them once installed; wording is "repository plugin and marketplace defaults".
4. Agent-name precedence between user and workspace in VS Code is not documented (CLI: user wins).
5. No minimum Copilot CLI version is published for repo `settings.json` plugin keys or Agent Plugins 1.1.0.
6. Whether VS Code's Copilot harness honors the full CLI `.mcp.json` walk-up (subdirectory `.mcp.json`) and `.github/mcp.json` is not stated; VS Code docs only name workspace-root `.mcp.json`.
