---
name: backfilling-items
description: Scaffold an item leaf for code that already exists in this cell's placement directory, pointer-only, with no build.
---

# Backfilling Items

**Trigger**: `populating-overlay` discovers an existing artifact within this cell's placement-mapped directory that has no item leaf yet.

**Inputs**: the discovered file/directory path, this cell's `trellis.config.json` placement entry, this cell's current `INVENTORY.md`, the coordinator's Classifying verdict for the artifact.

**Procedure**:

1. Skip if an item leaf already exists whose `CONTRACT.md` `Source` pointer matches this path — already known, whether built through Trellis previously or backfilled in an earlier population run.
2. Derive the item slug from the file or directory name.
3. Read the artifact's actual code to determine what it provides — name, inputs (signature/props/params as actually written), output (what it renders, returns, or exposes), and how it composes other artifacts.
4. Check this against the coordinator's Classifying verdict for the artifact. If the verdict disagrees with the cell whose directory it was found in (e.g. something entity-bound sitting in a domain-agnostic folder), do **not** silently accept the folder's placement — this is existing organisational debt, not a Trellis decision to paper over. Record it as a finding instead of scaffolding an item leaf for it.
5. If it matches: scaffold the item leaf at `.claude/trellis/<this-cell>/<item-slug>/`:
   - `CONTRACT.md` — `Source` pointer set immediately (the code already exists); `Provides` section reverse-engineered from the real code, not from a ratified need; `Handshake` consumer recorded as `— (backfilled)`; status `satisfied`.
   - `USAGE.md` — best-effort what-it-is / when-to-use / placement / example, drawn from the code itself and any existing call sites or doc comments.
   - `CHANGELOG.md` — one seed row: date, "Backfilled from existing code during populating-overlay", no ratified-with (there was no contract).
6. Regenerate this item's `REFERENCES.md` and this cell's `INVENTORY.md` — same generation procedure as Self-Documenting: follow the `Source` pointer, scan imports.
7. **Seed Learned Patterns.** Once several items in this cell have been backfilled in the same pass, note any convention consistent across all of them (naming, structure, file organisation) and log it into `AGENT.md`'s Learned Patterns, same conflict-check discipline as Self-Documenting. If backfilled items in the same cell show *contradictory* conventions, do not silently pick one — record it as a finding: existing inconsistency for the user to resolve, not something Trellis invents an answer to.

**Output**: new item leaves scaffolded (pointer-only, `satisfied` status, no code written), this cell's `INVENTORY.md` regenerated, `AGENT.md` Learned Patterns extended where a consistent convention was actually observed, and a findings list of anything ambiguous, contradictory, or misplaced — handed back to `populating-overlay` for the human review gate.
