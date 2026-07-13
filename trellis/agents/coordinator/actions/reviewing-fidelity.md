---
name: reviewing-fidelity
description: Check a build attempt's delivered artifact against its ratified contract shape and placement path.
---

# Reviewing Fidelity

**Trigger**: a build attempt has produced output.

**Inputs**: the ratified `CONTRACT.md` output shape for the item, the actual delivered artifact (code + governance files).

**Procedure**: verify two things only — the code was written to the path resolved from `trellis.config.json`'s placement map for that item's cell, and the delivered shape matches what the `CONTRACT.md` says was ratified. This is not a quality or behavioural review. Record specific findings when it fails: what shape was expected, what was delivered, and where.

**Output**: pass/fail verdict plus notes, written into the item's `attempts/attempt-<NNN>.md`.
