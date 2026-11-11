# Environment Setup

A curated, repeatable set of AI-agent customizations that can be applied to any repository.

## Language

**Component**:
A single installable unit of agent customization: an agent, skill, MCP server, instruction file, or prompt.
_Avoid_: Item, asset, tool, plugin

**Profile**:
A named selection of Components, such as `base`, `dotnet` or `typescript`.
_Avoid_: Preset, bundle, kit, curated list

**Target repo**:
The repository a Profile is applied into.
_Avoid_: Consumer repo, project, workspace

**Apply**:
Bring a Target repo in line with one or more Profiles; re-running converges rather than duplicates.
_Avoid_: Install, setup, scaffold, sync
