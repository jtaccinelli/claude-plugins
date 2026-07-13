---
name: creating-items
description: Scaffold a new item leaf after an Unmet verdict, only when the reuse guardrails hold.
---

# Creating Items

**Trigger**: the coordinator schedules creation after an Unmet verdict.

**Inputs**: the ratified contract.

**Procedure**: absence alone never justifies creation. Create only when one of these holds:

- **Significant and logically consistent** with this node's remit; or
- **Obviously reusable** — 2+ distinct consumers need it within this single pass; or
- **Historically demanded** — this node's `REQUESTS.md` already shows 3+ recorded requests sharing this contract's canonical key (target node + ratified output-shape signature).

If none hold, do not create — inline the implementation at the consumer instead, and log the request in this node's `REQUESTS.md` (new row, or increment an existing key's count).

If creating: scaffold the item leaf at `.claude/trellis/<this-node>/<item-slug>/` with `CONTRACT.md` and `USAGE.md` stubs (status `proposed`), from the templates at `trellis/skills/scoping-requests/templates/CONTRACT.md` and `trellis/skills/scoping-requests/templates/USAGE.md` (the plugin's fixed templates, not project-specific); the actual code and the rest of the governance files are produced by [Executing Build Attempts](executing-build-attempts.md) and [Self-Documenting](self-documenting.md).

**Report** (returned to the dispatcher): a new item leaf scaffolded, or a logged/incremented request.
