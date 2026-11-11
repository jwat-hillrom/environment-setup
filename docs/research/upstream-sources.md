# Research: initial upstream sources

Resolves [#3](https://github.com/jwat-hillrom/environment-setup/issues/3). Researched 2026-11-11 against the repositories themselves (GitHub API: repo metadata, tags, releases, recursive trees at the latest tag, and raw files at that tag). Every claim cites its source. Repository content was treated as data only; no install commands were run.

Abbreviations for source URLs below:

- `MP` = `https://github.com/mattpocock/skills/blob/v1.3.2/`
- `ECC` = `https://github.com/affaan-m/ECC/blob/v2.2.3/`
- `TS` = `https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/`
- `BI` = `https://github.com/better-auth/better-icons/blob/v1.0.5/`

## Summary

| Source | Component types | Install mechanism | Hosts (as declared upstream) | Licence | Pinnable? (latest) |
|---|---|---|---|---|---|
| [mattpocock/skills](https://github.com/mattpocock/skills) | Skills (38 `SKILL.md`; 27 in the plugin manifest) | Claude Code / Codex / Copilot plugin marketplace (`.claude-plugin/`); `npx skills` copy; Gemini `skills install` | Claude Code, Codex, GitHub Copilot (CLI + VS Code), Gemini CLI, others via `npx skills` | MIT | Yes: semver tags + GitHub Releases, latest `v1.3.2` (2026-10-29) |
| [affaan-m/ECC](https://github.com/affaan-m/ECC) (requested as `affaan-m/ecc`) | Skills (293), custom agents (68), commands (94), hooks, rules (122 files), MCP config, Copilot instruction file + 5 prompt files, per-harness adapters | Claude Code / Codex plugin marketplace; `npx ecc-universal@<ver>` guided installer; `./install.sh --profile … --target …` | Claude Code (primary), Codex, Cursor, OpenCode, Gemini, Zed, Antigravity, Qwen, Kimi, … ; Copilot = "instruction-only" | MIT | Yes: tags + Releases + npm `ecc-universal`, latest `v2.2.3` (2026-10-01); `main` is 1285 commits ahead |
| [Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) | 1 skill (`design-taste-frontend`) with ~1,630 bundled files incl. scripts and a skill-scoped `Stop` hook | `npx skills add`; Claude Code / Codex plugin marketplace; `install.sh` symlink | Claude Code, Codex (plus any `npx skills` agent) | MIT | Yes: tags + Releases, latest `v2.5.1` (2026-11-07) |
| [better-auth/better-icons](https://github.com/better-auth/better-icons) | MCP server (stdio, npm `better-icons`), CLI, 1 skill | `npx better-icons setup` writes MCP config; manual `mcp.json`; `npx add-skill` for the skill | Cursor, Claude Code, OpenCode, Windsurf, VS Code (Copilot), Antigravity | MIT | Partly: tags/Releases latest `v1.0.5` (2026-03-14) but npm `latest` is `1.0.4`; upstream configs use unpinned `npx -y better-icons` |

Repository existence: all four exist. `affaan-m/ecc` resolves to the canonical `affaan-m/ECC` (case only; [API](https://api.github.com/repos/affaan-m/ecc) returns `full_name: affaan-m/ECC`). `leonxlnx/taste-skill` is canonically `Leonxlnx/taste-skill`. None is archived.

## 1. mattpocock/skills

- **Identity:** MIT, default branch `main`, ~2.2 MB, 175 files at `v1.3.2` ([repo API](https://api.github.com/repos/mattpocock/skills), [LICENSE](https://github.com/mattpocock/skills/blob/v1.3.2/LICENSE)).
- **Component types:** skills only. 38 `SKILL.md` files under `skills/{engineering,productivity,misc,in-progress}/<name>/` ([tree](https://github.com/mattpocock/skills/tree/v1.3.2/skills)). `skills/deprecated/` holds only a README. Some skills bundle scripts (e.g. `MP skills/misc/git-guardrails-claude-code/scripts/block-dangerous-git.sh`, `MP skills/engineering/wizard/template.sh`) and reference docs (`DEEPENING.md`, `ADR-FORMAT.md`, …).
- **Plugin scope:** [`.claude-plugin/plugin.json`](https://github.com/mattpocock/skills/blob/v1.3.2/.claude-plugin/plugin.json) (`name: mattpocock-skills`, `version: 1.3.2`) lists 27 skills (20 engineering + 7 productivity). `misc/` and `in-progress/` skills are **not** in the plugin.
- **Format:** `SKILL.md` with YAML frontmatter `name`, `description`, and sometimes `disable-model-invocation: true` (e.g. [`grill-me`](https://github.com/mattpocock/skills/blob/v1.3.2/skills/productivity/grill-me/SKILL.md)); README describes user-invoked vs model-invoked split ([README L211](https://github.com/mattpocock/skills/blob/v1.3.2/README.md#L211)). Each skill also has `agents/openai.yaml` (`interface.display_name`, `short_description`) for Codex ([example](https://github.com/mattpocock/skills/blob/v1.3.2/skills/engineering/tdd/agents/openai.yaml)).
- **Install** ([README L25-L95](https://github.com/mattpocock/skills/blob/v1.3.2/README.md#L25-L95)):
  - Claude Code: `claude plugin install mattpocock-skills@claude-plugins-official`.
  - Codex: `codex plugin marketplace add mattpocock/skills`.
  - GitHub Copilot CLI: `copilot plugin marketplace add mattpocock/skills` plus an `extraKnownMarketplaces` entry in `~/.copilot/settings.json`; VS Code: **Chat: Install Plugin From Source** with the repo URL ("It updates daily").
  - Gemini CLI: `gemini skills install … --path skills/engineering`.
  - Others (cursor, opencode, windsurf, …): `npx skills@latest add mattpocock/skills -a <agent>` (copies editable files).
  - Marketplace manifest: [`.claude-plugin/marketplace.json`](https://github.com/mattpocock/skills/blob/v1.3.2/.claude-plugin/marketplace.json) (`name: mattpocock`, plugin `source: ./`).
- **Setup dependency:** README asks users to run `setup-matt-pocock-skills` once per repo; it writes per-repo config such as issue-tracker and triage-label docs ([README L95-L101](https://github.com/mattpocock/skills/blob/v1.3.2/README.md#L95-L101), [skill dir](https://github.com/mattpocock/skills/tree/v1.3.2/skills/engineering/setup-matt-pocock-skills)).
- **Versioning:** Changesets-driven semver; `package.json` and `plugin.json` versions are synced by script ([package.json](https://github.com/mattpocock/skills/blob/v1.3.2/package.json), [`scripts/sync-plugin-version.mjs`](https://github.com/mattpocock/skills/blob/v1.3.2/scripts/sync-plugin-version.mjs)). Tags `v1.2.2 … v1.3.2` with GitHub Releases; latest `v1.3.2` published 2026-10-29 ([releases](https://github.com/mattpocock/skills/releases)). `main` is 3 commits ahead of `v1.3.2` ([compare](https://github.com/mattpocock/skills/compare/v1.3.2...main)). `package.json` is `private: true` — not on npm.
- **Vendoring/pinning concerns:** small and clean. Plugin installs track the default branch rather than a tag (README: "A plugin updates itself"); pinning requires vendoring a tag or using a ref-pinned marketplace source. Skills cross-reference each other by name (e.g. `implement-spec` closes with `/code-review`, README L227), so subsetting can break chains. `setup-matt-pocock-skills` expects to write per-repo docs, which overlaps with what Apply would own.

## 2. affaan-m/ECC

- **Identity:** MIT, ~61 MB, 4,212 files at `v2.2.3` ([repo API](https://api.github.com/repos/affaan-m/ECC), [LICENSE](https://github.com/affaan-m/ECC/blob/v2.2.3/LICENSE)). Description: "agent harness performance optimization system". Also published as npm `ecc-universal` (`2.2.3`, [package.json](https://github.com/affaan-m/ECC/blob/v2.2.3/package.json)); `npm view` shows `latest: 2.2.3`.
- **Component types** (counts from the `v2.2.3` tree; plugin manifest states "68 agents, 293 skills, 94 legacy command shims" — [plugin.json](https://github.com/affaan-m/ECC/blob/v2.2.3/.claude-plugin/plugin.json)):
  - Skills: 293 at `skills/<name>/SKILL.md` (flat). The tree contains 1,027 `SKILL.md` total, the rest being translations under `docs/{ja-JP,zh-CN,es,tr,zh-TW,ko-KR}` and per-harness copies under `pi/`, `.kiro/`, `.agents/`, `.cursor/`.
  - Custom agents: 68 at `agents/*.md`; Claude Code format frontmatter `name`, `description`, `tools`, `model` ([example](https://github.com/affaan-m/ECC/blob/v2.2.3/agents/code-reviewer.md)).
  - Commands (prompts): 94 at `commands/*.md`, frontmatter `description`, `argument-hint` ([example](https://github.com/affaan-m/ECC/blob/v2.2.3/commands/plan.md)).
  - Hooks: [`hooks/hooks.json`](https://github.com/affaan-m/ECC/blob/v2.2.3/hooks/hooks.json) (Claude Code `PreToolUse` etc., running inline `node -e` bootstrap code that locates the plugin root under `~/.claude`), [`hooks/codex-hooks.json`](https://github.com/affaan-m/ECC/blob/v2.2.3/hooks/codex-hooks.json); configurable `hook_profile` minimal/standard/strict via plugin `userConfig`.
  - Rules / instruction files: 122 files under `rules/` (common + 21 language/framework packs, e.g. `csharp`, `typescript`, `react`).
  - MCP: [`.mcp.json`](https://github.com/affaan-m/ECC/blob/v2.2.3/.mcp.json) (pins `chrome-devtools-mcp@1.10.1`) and a catalogue [`mcp-configs/mcp-servers.json`](https://github.com/affaan-m/ECC/blob/v2.2.3/mcp-configs/mcp-servers.json). `plugin.json` declares `mcpServers: {}`.
  - Copilot: [`.github/copilot-instructions.md`](https://github.com/affaan-m/ECC/blob/v2.2.3/.github/copilot-instructions.md), five `.github/prompts/*.prompt.md` (plan, tdd, security-review, build-fix, refactor), [`.vscode/settings.json`](https://github.com/affaan-m/ECC/blob/v2.2.3/.vscode/settings.json) enabling `chat.promptFiles` ([README L411](https://github.com/affaan-m/ECC/blob/v2.2.3/README.md#L411)).
  - Other: install manifests/profiles ([`manifests/install-profiles.json`](https://github.com/affaan-m/ECC/blob/v2.2.3/manifests/install-profiles.json), `install-modules.json`, `install-components.json`), a gitagent [`agent.yaml`](https://github.com/affaan-m/ECC/blob/v2.2.3/agent.yaml), Python dashboard, Docker, an `ecc2/` subtree (not inspected), tests, Codex plugin under `.codex-plugin/` and `plugins/ecc/`.
- **Install** ([README L162-L417](https://github.com/affaan-m/ECC/blob/v2.2.3/README.md#L162-L417)):
  - Guided: `npx ecc-universal@2.2.3 setup` (Claude Code plugin) and `npx ecc-universal@2.2.3 install --guided` (Claude Code, Codex, Kimi); supports `--profile core|minimal|full`, `--target`, `--dry-run`.
  - Native plugin: `/plugin marketplace add https://github.com/affaan-m/ECC` then `/plugin install ecc@ecc` ([marketplace.json](https://github.com/affaan-m/ECC/blob/v2.2.3/.claude-plugin/marketplace.json)); Codex `codex plugin marketplace add affaan-m/ECC`.
  - Other harnesses: `./install.sh --profile minimal --target <cursor|gemini|zed|antigravity|qwen|…>` (also `install.ps1`).
  - Copilot: no installer; files are "already in place" in the repo, i.e. copy `.github/copilot-instructions.md` and prompts ([README L1653-L1693](https://github.com/affaan-m/ECC/blob/v2.2.3/README.md#L1653-L1693)).
- **Hosts:** "works best with Claude Code today … supported Codex sync path … capability-limited adapters for Cursor, OpenCode, Gemini, Zed, GitHub Copilot, Antigravity, Qwen" ([README L140](https://github.com/affaan-m/ECC/blob/v2.2.3/README.md#L140)). Platform matrix lists GitHub Copilot as "Instruction-only … No ECC hooks, runtime agents, delegation, or native skill discovery" ([README L1293](https://github.com/affaan-m/ECC/blob/v2.2.3/README.md#L1293)).
- **Versioning:** tags + Releases up to `v2.2.3` (2026-10-01, [releases](https://github.com/affaan-m/ECC/releases)); `VERSION` file, `package.json`, `plugin.json`, `marketplace.json` and `agent.yaml` all say `2.2.3`. npm "releases are cut per version tag" ([README L345](https://github.com/affaan-m/ECC/blob/v2.2.3/README.md#L345)). `main` is **1,285 commits ahead** of `v2.2.3` ([compare](https://github.com/affaan-m/ECC/compare/v2.2.3...main)).
- **Vendoring/pinning concerns:** very large and fast-moving; the tag lags `main` heavily. Content is Claude-Code-first: agents use Claude tool names (`Read, Grep, Glob, Bash`) and `model: sonnet`; hooks depend on `CLAUDE_PLUGIN_ROOT` and Node scripts under `scripts/hooks/`, so hooks/agents cannot be vendored without the script runtime. Skill descriptions are aggressive (e.g. agent description "MUST BE USED for all code changes", [code-reviewer.md](https://github.com/affaan-m/ECC/blob/v2.2.3/agents/code-reviewer.md)), which can dominate model-invoked selection if many are installed. Name collisions likely with other sources (ECC has skill `tdd-workflow`, agent `chief-of-staff`, command `/plan`, …; mattpocock ships skills `tdd` and in-progress `chief-of-staff`). README warns against stacking install methods ([L277](https://github.com/affaan-m/ECC/blob/v2.2.3/README.md#L277)). Copilot-support claims ("Copilot has no hook system / subagent API", L1693) are upstream's statement and may be stale relative to current VS Code; not verified here.

## 3. Leonxlnx/taste-skill

- **Identity:** MIT, ~102 MB, 1,663 files at `v2.5.1` ([repo API](https://api.github.com/repos/Leonxlnx/taste-skill), [LICENSE](https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/LICENSE)).
- **Component types:** a single skill. The repo "holds one skill, `design-taste-frontend`"; earlier skills moved to `Leonxlnx/taste-skill-classics`, `Leonxlnx/imagegen-skills`, or were removed ([README "Moved skills"](https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/README.md#moved-skills)). The skill directory `skills/taste-skill/` contains 1,631 files: `SKILL.md`, Node scripts (checker/gate), CSS/JS component, effect and diagram libraries, and ~160 image files.
- **Format:** [`skills/taste-skill/SKILL.md`](https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/skills/taste-skill/SKILL.md) frontmatter has `name: design-taste-frontend` (differs from the folder name `taste-skill`), a long `description`, and a **`hooks.Stop`** entry: a shell command that searches `$CLAUDE_SKILL_DIR`, `$CLAUDE_PLUGIN_ROOT`, `.claude/skills/…`, `.agents/skills/…` and `~/.claude|.agents/skills/…` for `scripts/gate.mjs` and `exec node` it, with a 600 s timeout. Requires Node 22+ and Chrome/Edge/Chromium; ffmpeg optional ([README L86](https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/README.md#L86)).
- **Install** ([README L78-L121](https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/README.md#L78-L121)):
  - `npx skills add https://github.com/Leonxlnx/taste-skill --skill "design-taste-frontend"`.
  - Claude Code plugin: `/plugin marketplace add Leonxlnx/taste-skill`, `/plugin install taste-skill@taste-skill` ([`.claude-plugin/marketplace.json`](https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/.claude-plugin/marketplace.json)).
  - Codex plugin: [`.codex-plugin/plugin.json`](https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/.codex-plugin/plugin.json); [`.agents/plugins/marketplace.json`](https://github.com/Leonxlnx/taste-skill/blob/v2.5.1/.agents/plugins/marketplace.json) points at the git URL with `ref: main` (not a tag).
  - `./install.sh` symlinks into Claude Code (`--codex`, `--all-agents`); Windows via Git Bash junction.
- **Hosts:** Claude Code and Codex explicitly; anything `npx skills` supports. No Copilot-specific packaging.
- **Versioning:** tags + Releases (`v2.1.3 … v2.5.1`); latest `v2.5.1` 2026-11-07, and `main` is identical to it ([compare](https://github.com/Leonxlnx/taste-skill/compare/v2.5.1...main)). Version string is also embedded in the skill description text. Release cadence is fast (5 releases in ~3 weeks).
- **Vendoring/pinning concerns:** heavy payload (~100 MB repo, many binaries) for one skill. Executable skill-scoped `Stop` hook in frontmatter (Claude Code semantics; Bash syntax). Name mismatch (`design-taste-frontend` vs folder `taste-skill`) matters for hosts that key skills by folder name. Codex marketplace tracks `main`. Frequent repo splits/moves have already renamed where skills live.

## 4. better-auth/better-icons

- **Identity:** MIT, ~68 KB, 24 files at `v1.0.5` ([repo API](https://api.github.com/repos/better-auth/better-icons), [LICENSE](https://github.com/better-auth/better-icons/blob/v1.0.5/LICENSE)). Description: "Skill and MCP server for searching and retrieving icons". Last push 2026-04-02.
- **Component types:**
  - MCP server + CLI: TypeScript in `src/` (`server.ts`, `commands/*`), built with Bun to `dist/`, published as npm `better-icons` with `bin: better-icons` ([package.json](https://github.com/better-auth/better-icons/blob/v1.0.5/package.json)). Tools `search_icons`, `get_icon`, … backed by Iconify ([README MCP Tools](https://github.com/better-auth/better-icons/blob/v1.0.5/README.md#mcp-tools)).
  - Skill: one file at [`skills/SKILL.md`](https://github.com/better-auth/better-icons/blob/v1.0.5/skills/SKILL.md) (not in a named subfolder), frontmatter `name: better-icons`, `description`; documents CLI usage.
- **Install** ([README](https://github.com/better-auth/better-icons/blob/v1.0.5/README.md)):
  - Skill: `npx add-skill better-auth/better-icons`.
  - MCP: `npx better-icons setup` interactively writes MCP config; [`src/agents.ts`](https://github.com/better-auth/better-icons/blob/v1.0.5/src/agents.ts) targets Cursor (`~/.cursor/mcp.json`), Claude Code (`~/.claude.json` or `~/.claude/settings.json`, project `.mcp.json`), OpenCode, Windsurf, VS Code Copilot (`~/.vscode/mcp.json` / project `.vscode/mcp.json`), Antigravity.
  - Manual: `{"command": "npx", "args": ["-y", "better-icons"]}` — unpinned.
- **Versioning:** tags incl. pre-releases; latest GitHub Release `v1.0.5` (2026-03-14), `main` 2 commits ahead ([compare](https://github.com/better-auth/better-icons/compare/v1.0.5...main)). **npm `latest` is `1.0.4`** (`npm view better-icons`), so tag `v1.0.5` appears unpublished on npm.
- **Vendoring/pinning concerns:** the useful artifact is an npm package, not files — pin as `better-icons@1.0.4` (or whatever is published) in `mcp.json` rather than vendoring. Runtime calls the Iconify API (network). `dist/` is not committed, so vendoring from git requires a Bun build. User-level VS Code path `~/.vscode/mcp.json` written by `setup` is upstream's choice; not verified as a location VS Code reads.

## Open uncertainties

- Whether each host actually honours the upstream manifests as claimed (e.g. VS Code "Install Plugin From Source" consuming `.claude-plugin/` for mattpocock; Copilot reading skill-level `hooks:` frontmatter in taste-skill). That belongs to host-discovery research, not verified here.
- ECC's statement that Copilot lacks hooks/subagents (README L1693) is upstream opinion and may be out of date.
- Exact overlap/collision list between sources (skill/agent/command names) was only spot-checked (`chief-of-staff`, `tdd`).
- better-icons: why `v1.0.5` is tagged but npm `latest` is `1.0.4`; whether `~/.vscode/mcp.json` is a valid VS Code location.
- Plugin-based installs (mattpocock, taste-skill Codex marketplace) track default branches; whether the hosts' plugin systems support pinning a marketplace to a tag/commit was not investigated.
- File counts are from the git tree at the tag; repo `size` from the API is approximate (includes history).

## Prompt-injection / agent-directed content

No prompt-injection attempts were found in the READMEs (spot regex scan for "ignore previous", "if you are an AI", HTML comments, etc.). Noted agent-directed or executable content, treated as data:

- READMEs contain install commands (`npx …`, `/plugin install …`, `./install.sh`) — normal docs, not executed.
- taste-skill `SKILL.md` frontmatter embeds a shell `Stop` hook that executes `node …/gate.mjs` whenever an agent stops.
- ECC `hooks/hooks.json` runs inline `node -e` code on `PreToolUse` that searches `~/.claude/plugins` for an ECC root and `require`s scripts from it.
- ECC agent descriptions use imperative directives ("MUST BE USED for all code changes") aimed at model routing.
