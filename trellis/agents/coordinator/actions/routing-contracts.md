---
name: routing-contracts
description: Hand an authored contract to the cell responsible for satisfying it and await a verdict.
---

# Routing Contracts

**Trigger**: a consumer cell authors a contract for something it needs from another cell.

**Inputs**: consumer cell + item, target cell (classify it via [Classifying](classifying.md) if not already stated), the contract object (`name`, `inputs`, `output`, `placement`).

**Procedure**: hand the contract to the target cell's cell-agent instantiation (`--cell <concern.layer>`), including the full contract object and a pointer to the consumer's `CONTRACT.md`. Wait for a verdict.

**Output**: a routed contract awaiting verdict.
