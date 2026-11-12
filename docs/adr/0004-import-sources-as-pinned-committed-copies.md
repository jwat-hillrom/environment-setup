# Sources are imported as committed copies pinned to a commit

Every Source is brought in by Import: a deterministic CLI command fetches the Source at the commit SHA recorded in `sources.json` (alongside its tag), takes only the Components listed by name, renames them with the Source alias and rewrites their cross-references, fails if any reference is left unresolved (unless explicitly allowed), applies only mechanical conversions (frontmatter mapping, folder naming, dropping Source hooks), and writes the result under `sources/<alias>/` for review and commit. Vendored files are never hand-edited; anything needing judgement becomes an authored Component. This keeps Target repos reproducible from one ref of this repo and makes every upstream change a reviewable diff.

## Considered Options

- Plugin references (`.github/copilot/settings.json`): can't be prefixed (ADR-0001) and the initial Sources' plugins track default branches, so they aren't pinned.
- Fetching Sources at Apply time: Target repos would depend on upstream availability and Apply could no longer be reviewed as one ref.

## Consequences

- Sources that publish no tags (e.g. hallmark, imagegen-skills) are pinned by commit SHA alone.
- MCP-server Sources are not copied; their Component is a `.mcp.json` entry pinned to an exact published package version.
- Each Source's licence is stored with its imported Components, and Apply places a `.github/THIRD_PARTY_NOTICES.md` listing the Sources it applied.
