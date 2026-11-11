# Only hooks authored in this repo are Components

Hooks are the one deterministic guardrail agents have, so they are a Component type, but only hooks authored (and reviewed) in this repo are applied; hooks shipped by Sources are never applied directly. Source hooks run arbitrary code on every matching agent event, would change silently on re-Apply, and as shipped are not portable (taste-skill's is bash-only, ECC's resolve scripts from `~/.claude`). A Source hook worth having is ported by hand into a cross-platform authored hook.
