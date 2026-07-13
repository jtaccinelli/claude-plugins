---
name: assessing-contracts
description: Check a routed contract against this cell's inventory and return a Met / Partial / Unmet / Countered verdict.
---

# Assessing Contracts

**Trigger**: the coordinator routes a contract to this cell.

**Inputs**: the contract's required output shape and inputs, this cell's `INVENTORY.md`.

**Procedure**: check the contract against existing items.

- **Met** — an existing item satisfies the contract exactly as written.
- **Partial · additive** — an existing item can be extended to satisfy it without breaking any consumer listed in that item's `REFERENCES.md`. Modify in place, bump `CHANGELOG.md`, notify referencers.
- **Partial · breaking** — satisfying it as written would break an existing consumer. Do not modify the existing item — spin a new variant instead; let the reuse rule consolidate later if warranted.
- **Unmet** — nothing in this cell addresses it.
- **Counter** — the requested shape doesn't match how this cell actually works. Propose the shape you would actually produce and return it for the consumer to ratify.

**Output**: one verdict plus rationale, returned to the coordinator.
