---
name: self-documenting
description: Record what was just built — pointer, usage, changelog — and regenerate the item's and cell's derived files.
---

# Self-Documenting

**Trigger**: immediately after a build attempt passes Fidelity Review, still within the same build step, back in the main tree.

**Inputs**: what was just built.

**Procedure**: set the item's `CONTRACT.md` `Source` pointer(s) to the resolved host path(s); write or update `USAGE.md` (what it is, when to use it, placement, a usage example, do/don't); append a dated row to `CHANGELOG.md` (change, why, what it was ratified with) — if this is the item's first pass, create it from `trellis/skills/building-items/templates/CHANGELOG.md`; regenerate this item's `REFERENCES.md` and this cell's `INVENTORY.md` by following the `Source` pointer out to the real code and scanning its imports — both are generated, never hand-maintained, so a full regeneration each time is correct, not wasteful.

**Output**: updated item leaf (`CONTRACT.md`, `USAGE.md`, `CHANGELOG.md`, `REFERENCES.md`) and this cell's `INVENTORY.md`.
