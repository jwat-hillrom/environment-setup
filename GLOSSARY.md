# Environment Setup

A curated, repeatable set of AI-agent customizations that can be applied to any repository.

## Language

**Component**:
A single installable unit of agent customization: an agent, skill, MCP server, instruction file, or hook. Either drawn from a Source or authored in this repo.
_Avoid_: Item, asset, tool, plugin

**Source**:
An upstream repository Components are drawn from, identified by a short alias (e.g. `pocock`, `ecc`) that prefixes the names of its Components.
_Avoid_: Upstream, vendor, provider

**Import**:
Bring selected Components from a Source, at a pinned commit, into this repo: renamed with the Source alias, mechanically converted for the hosts, and committed.
_Avoid_: Sync, vendor, pull, fetch

**Catalog**:
The platform's authored, unprefixed, user-invocable skills (e.g. `/research`, `/review-change`); each builds on Source skills, which it hides from the slash menu.
_Avoid_: Commands, front doors, skill library

**Workflow**:
An ordered sequence of Stages for one kind of work (feature, bug, refactor), run by a `flow-` skill.
_Avoid_: Pipeline, process, state machine

**Stage**:
One step of a Workflow, performed by a Catalog skill (e.g. `/design`); completed, skipped with a reason, or failed.
_Avoid_: Step, phase, gate

**Work item**:
One feature, bug or refactor a Workflow runs on, with a committed folder holding its Stage outputs and state.
_Avoid_: Task, ticket, job

**Gate**:
A deterministic check run by the `check` command that blocks merging a Work item when it fails; reviewer judgements are never Gates.
_Avoid_: Quality gate score, threshold, approval

**Profile**:
A named selection of Components, such as `base`, `frontend` or `dotnet`. Every Profile includes `base` and may include other Profiles.
_Avoid_: Preset, bundle, kit, curated list

**Target repo**:
The repository a Profile is applied into.
_Avoid_: Consumer repo, project, workspace

**Apply**:
Bring a Target repo in line with one or more Profiles; re-running converges rather than duplicates.
_Avoid_: Install, setup, scaffold, sync

**Manifest**:
The committed record in a Target repo of what Apply placed there: the version applied, the Profiles chosen, and every file and config key each Component owns.
_Avoid_: Lockfile, state file, config

**Excluded**:
A Component a Target repo has opted out of: Apply never places, updates or removes it, and any of its files left behind belong to the Target repo.
_Avoid_: Detached, skipped, ignored, pinned
