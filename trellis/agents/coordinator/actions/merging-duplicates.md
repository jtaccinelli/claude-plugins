---
name: merging-duplicates
description: Collapse colliding contract chains or requests that share a canonical key into one.
---

# Merging Duplicates

**Trigger**: two or more contract chains reach the same provider item (multi-root convergence), or two contracts share a canonical key of `target node + ratified output-shape signature`.

**Inputs**: the colliding contracts or requests.

**Procedure**: merge them into a single contract the provider satisfies once. When merging requests logged in a node's `REQUESTS.md`, increment the shared key's count rather than adding a new row — this keeps the 3+ historical-demand counter in [Creating Items](../../node-agent/actions/creating-items.md) trustworthy.

**Output**: one collapsed contract, or one incremented request-count entry.
