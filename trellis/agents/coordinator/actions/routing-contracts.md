---
name: routing-contracts
description: Hand an authored contract to the node responsible for satisfying it and await a verdict.
---

# Routing Contracts

**Trigger**: a consumer node authors a contract for something it needs from another node.

**Inputs**: consumer node + item, target node (classify it via [Classifying](classifying.md) if not already stated), the contract object (`name`, `inputs`, `output`, `placement`).

**Procedure**: hand the contract to the target node's node-agent instantiation (`--node <concern.layer>`), including the full contract object and a pointer to the consumer's `CONTRACT.md`. Wait for a verdict.

**Output**: a routed contract awaiting verdict.
