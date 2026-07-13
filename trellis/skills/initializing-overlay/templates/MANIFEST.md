# Trellis — Manifest

> Coordination overlay, not a source tree. Governance lives under
> .claude/trellis/; code lives in the host project. Leaves point at real
> files; they never hold copies. First file every agent reads.

## Location

.claude/trellis/  — overlay root. Nothing outside .claude/ is restructured.

## Structure

Concern → Layer → Node → Item. Item leaves hold CONTRACT/USAGE/CHANGELOG +
pointer; INVENTORY/REFERENCES are generated.

## Placement

Node → real path is declared in trellis.config.json (single source of truth).

## Authoring

Classify → create leaf → resolve path from placement map → write code there →
set Source pointer → regenerate INVENTORY/REFERENCES → log CHANGELOG/REQUESTS.

## Authored: MANIFEST, config, AGENT, REQUESTS, CONTRACT, USAGE, CHANGELOG.

## Generated: INVENTORY, REFERENCES (via Source pointer → import scan).
