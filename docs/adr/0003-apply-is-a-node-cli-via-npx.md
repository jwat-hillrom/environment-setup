# Apply is a Node CLI run via npx from a GitHub ref

Apply is a deterministic program, not an agent skill, because re-runnable convergence and refuse-and-report rules need exact behaviour. It is a Node CLI run with `npx github:jwat-hillrom/environment-setup#<ref> apply ...`: Node is cross-platform and already required by the MCP servers and taste-skill, no npm publishing is needed, and one git ref pins both the CLI and the curated Components it applies. Apply never prompts; it prints what it changed and refused and exits non-zero on any refusal, so it behaves the same in a terminal, CI or when an agent runs it.

## Considered Options

- `dotnet tool`: forces the .NET SDK onto non-.NET repos.
- `gh` extension: requires a separate `gh-*` repo, and script extensions on Windows need bash or a compiled binary.
- PowerShell script: cross-platform only with `pwsh` 7, which isn't a given.
- An agent skill performing Apply: not deterministic.
