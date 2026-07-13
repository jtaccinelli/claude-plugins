---
name: reviewing-fidelity
description: Check a build attempt's delivered artifact against its ratified contract shape, placement path, and the cell's logged patterns.
---

# Reviewing Fidelity

**Trigger**: a build attempt has produced output.

**Inputs**: the ratified `CONTRACT.md` output shape for the item, the actual delivered artifact (code + governance files), this cell's `AGENT.md` Learned Patterns section.

**Procedure**: verify three things only:

1. The code was written to the path resolved from `trellis.config.json`'s placement map for that item's cell.
2. The delivered shape matches what the `CONTRACT.md` says was ratified.
3. The delivered artifact is consistent with every pattern already logged in this cell's `AGENT.md` Learned Patterns — naming, structure, file organisation, and argument-shape idioms this cell has settled into across prior items.

This is not an open-ended quality or behavioural review — every check is against something already established (the ratified contract, or a pattern the cell has already logged from its own prior output), never a new judgment invented on the spot. Record specific findings when any check fails: what was expected — citing the contract, or the specific logged pattern with its date and source item — what was delivered instead, and where.

**Output**: pass/fail verdict plus notes, written into the item's `attempts/ATTEMPT_<NNN>.md`. Pattern findings on a passing attempt are handed to the cell agent's [Self-Documenting](../../cell-agent/actions/self-documenting.md) action to log or reconcile against the existing Learned Patterns log.
