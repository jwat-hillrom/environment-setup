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

**Profile**:
A named selection of Components, such as `base`, `dotnet` or `typescript`.
_Avoid_: Preset, bundle, kit, curated list

**Target repo**:
The repository a Profile is applied into.
_Avoid_: Consumer repo, project, workspace

**Apply**:
Bring a Target repo in line with one or more Profiles; re-running converges rather than duplicates.
_Avoid_: Install, setup, scaffold, sync
