# Source Components are renamed with the Source alias at import

Components drawn from a Source are renamed `<alias>-<name>` (e.g. `pocock-tdd`, `ecc-tdd-workflow`) across every Component type, while Components authored in this repo stay unprefixed. This lets overlapping Sources coexist and keeps the user's own curated Components (which compose Source ones) as the primary namespace. The rename, including rewriting the cross-references Source skills make to each other by name, happens once when a Source is imported into this repo and is verified there; Apply only copies already-renamed files.

## Consequences

- A Source brought in as a plugin reference keeps its upstream names and cannot be prefixed.
- Upstream text is no longer byte-identical to the Source, so updating a Source means re-running the import rename, not a plain copy.
