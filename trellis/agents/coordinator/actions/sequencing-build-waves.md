---
name: sequencing-build-waves
description: Topologically order a ratified plan's items into parallel build waves.
---

# Sequencing Build Waves

**Trigger**: a plan has been ratified and approved at the human sign-off gate.

**Inputs**: the full contract graph for the request (every item and its dependency edges).

**Procedure**: topological sort, not layer-order. Globals and Elements always build in wave 1 — everything else consumes them. Thereafter, any item whose dependencies are all satisfied builds next, in parallel with any other such item, regardless of layer number. Layer order is the common shape of the graph, not the scheduler.

**Output**: an ordered list of build waves, each wave a set of items ready to build in parallel, handed to `building-items`.
