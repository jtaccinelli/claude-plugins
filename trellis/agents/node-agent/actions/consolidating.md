---
name: consolidating
description: Extract a dedicated item once a request key is historically demanded, and migrate its ad hoc consumers.
---

# Consolidating

**Trigger**: a key in this node's `REQUESTS.md` reaches 3+ recorded requests.

**Inputs**: the request key, the `REFERENCES.md` of whichever item(s) are currently serving that need ad hoc (inlined at each consumer).

**Procedure**: extract a dedicated item for the key (via [Creating Items](creating-items.md), which will now pass the "historically demanded" guardrail), then migrate every consumer listed as depending on the ad hoc implementations to consume the new item instead. This is a scheduled refactor, not a build-blocker — it can run in its own build wave.

**Report** (returned to the dispatcher): a new item plus updated imports at every migrated consumer; the `REQUESTS.md` row marked `EXTRACTED <date>`.
