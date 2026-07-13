---
name: self-documenting
description: Record what was just built, regenerate the item's and cell's derived files, and log any new pattern into the cell's AGENT.md.
---

# Self-Documenting

**Trigger**: immediately after a build attempt passes Reviewing Fidelity, still within the same build step, back in the main tree.

**Inputs**: what was just built, the coordinator's Reviewing Fidelity notes (including any pattern findings), this cell's `AGENT.md` Learned Patterns section.

**Procedure**:

1. Set the item's `CONTRACT.md` `Source` pointer(s) to the resolved host path(s).
2. Write or update `USAGE.md` (what it is, when to use it, placement, a usage example, do/don't).
3. Append a dated row to `CHANGELOG.md` (change, why, what it was ratified with) — if this is the item's first pass, create it from `trellis/skills/building-items/templates/CHANGELOG.md`.
4. Regenerate this item's `REFERENCES.md` and this cell's `INVENTORY.md` by following the `Source` pointer out to the real code and scanning its imports — both are generated, never hand-maintained, so a full regeneration each time is correct, not wasteful.
5. **Log patterns.** Read the coordinator's pattern-consistency findings from this attempt's Reviewing Fidelity notes.
   - If a finding simply confirms an existing logged pattern, do nothing — do not log a repeat confirmation.
   - If a finding surfaces a genuinely new, consistent convention not yet logged (e.g. this is the cell's first item, or a new idiom compatible with everything already logged), append a dated row to `AGENT.md`'s `## Learned Patterns` table: date, this item/attempt, the pattern stated in one sentence, and this item as the source.
   - Before appending, check the proposed pattern against every existing row. If it would reverse a previously logged pattern — contradicts what an earlier item established — do not apply it. Stop and use `AskUserQuestion` to surface the conflict, citing the prior entry (date, source item, pattern) against the new one. Never silently overwrite an established pattern.

**Output**: updated item leaf (`CONTRACT.md`, `USAGE.md`, `CHANGELOG.md`, `REFERENCES.md`), this cell's `INVENTORY.md`, and — when a new pattern was logged or a conflict was raised — this cell's `AGENT.md`.
