---
name: classifying
description: Run the Concern x Layer decision matrix on a new request or a sub-need a node agent surfaces.
---

# Classifying

**Trigger**: a new request, or a node agent surfaces a sub-need that isn't yet classified.

**Inputs**: the request or need, stated in plain language.

**Procedure** — run in order; steps 1–2 always apply, step 3 only if not ambient:

1. **Concern.** Renders → Presentation. Transforms, decides, adapts, or holds state → Logic. Crosses a boundary (HTTP, cache, storage, DB) → Data.
2. **Ambient check.** It is a **Global** if roughly all hold: provided once at the root; consumed at many levels regardless of granularity; has no owning parent. If so, record `(Concern, Global)` and stop.
3. **Layer**, via two discriminators:
   - **Entity-bound?** Does it import or hardcode the domain?
     - **No (agnostic)** — Element/Component/Module, split by composition: composes no other artifact → **Element**; composes at most Elements / is a base primitive → **Component**; composes 2+ lower artifacts → **Module**.
     - **Yes (bound)** — Feature/Page, split by route-binding: not route-owned → **Feature**; the artifact *is* what lives at a route → **Page**.

Classification is by definition, not by usage — a generic component used inside a Feature is still a Module; it becomes a Feature only when the artifact itself binds to the domain.

**Output**: `(concern, layer)` or `(concern, global)`.
