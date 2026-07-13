---
name: reconciling-verdicts
description: Bind, counter-ratify, schedule, or escalate a verdict a cell agent returns on a routed contract.
---

# Reconciling Verdicts

**Trigger**: a cell agent returns a verdict on a routed contract — Met, Partial · additive, Partial · breaking, Unmet, or Countered.

**Inputs**: the verdict and its rationale.

**Procedure**:
- **Met / Partial · additive / Partial · breaking** — bind the contract to the returned item as-is; record the agreement.
- **Countered** — the provider proposed a different implementation shape. Return it to the consumer to ratify or reject. If the consumer ratifies, bind. If the consumer rejects and the provider will not yield, this is an unresolved conflict — escalate.
- **Unmet** — schedule creation via the provider cell's [Creating Items](../../cell-agent/actions/creating-items.md) action, subject to its reuse guardrails.
- **Unresolved conflict** (consumer and provider cannot converge on shape-of-need vs. shape-of-implementation) — stop and use `AskUserQuestion` to surface the conflict. Never self-resolve a design tie.

**Output**: contract status set to `satisfied | countered→resolved | create | escalated`.
