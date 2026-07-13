---
name: scoping-requests
description: Classify a request into its entry node(s), run the consumer-authors-the-contract handshake across every node it touches, and get human sign-off on the resulting build plan before anything is written.
argument-hint: "<request description>"
---

# Scoping Requests

Turn a plain-language request into a ratified, human-approved build plan. The coordinator classifies the request, the consumer-authors-the-contract chain runs outward from the entry node(s) until every need bottoms out at an Element or a Global, and the consolidated plan is presented for sign-off before `building-items` touches anything.

Requires `.claude/trellis/MANIFEST.md` to exist — run `/trellis:initializing-overlay` first if it doesn't.

---

## Pre-work

1. Confirm `.claude/trellis/MANIFEST.md` exists. If not, stop and tell the user to run `/trellis:initializing-overlay` first.
2. Establish the request from `$ARGUMENTS`. If vague or underspecified, interview the user until it's concrete enough to classify — a request must describe an observable capability or fix, not an implementation.
3. Derive a request slug (kebab-case, short) and check whether `.claude/trellis/requests/<slug>/PLAN.md` already exists.
   - If it exists and its status is `approved`: report that this request is already scoped and point to `building-items`. Stop.
   - If it exists and its status is `draft`: this is a resume — read it in full and continue from where it left off.
   - If it does not exist: this is a fresh scope.

---

## Steps

### Step 1 — Classify entry node(s)

As **coordinator**, run [Classifying](../../agents/coordinator/actions/classifying.md) on the request itself. Requests are not force-fed to a default layer — a page-level feature enters at Page, a recolor enters at Presentation · Element, a webhook enters at Data. A request may be multi-rooted (it needs work starting in more than one node at once); record every entry node.

### Step 2 — Interrogate outward from each entry node

Treat every entry node as a team member with a standing work order, not a queue to work through one at a time. Dispatch one sub-agent per entry node **at once**, using the **node-agent** agent (`--node <concern.layer>`):

- **Work order**: determine what the request needs from *this* node; for anything this node cannot itself provide, author a contract for what it needs from another node (per its Assessing Contracts / Creating Items actions and the request description).
- **Report expected back**: this node's own needs, plus any contracts it authored for cross-node needs.

Wait for every dispatched report before proceeding — entry nodes with no dependency on each other are dispatched and interrogated in parallel, never serialized just for convenience.

For each authored contract, the same dispatch-and-report cycle runs up the chain:

1. As **coordinator**, run [Routing Contracts](../../agents/coordinator/actions/routing-contracts.md) — classify the target node if not already named, and dispatch the contract to that node's node-agent as its work order.
2. That node-agent reports back via [Assessing Contracts](../../agents/node-agent/actions/assessing-contracts.md): Met / Partial · additive / Partial · breaking / Unmet / Countered.
3. As **coordinator**, run [Reconciling Verdicts](../../agents/coordinator/actions/reconciling-verdicts.md) against the returned report. On an unresolved Countered conflict (consumer rejects the provider's counter and neither will yield), use `AskUserQuestion` to escalate — do not proceed past this contract until the user resolves it.
4. If the verdict is Unmet, and that provider node in turn needs something from a further node to satisfy it, that provider becomes a consumer and authors the next contract — recurse into Step 2 for it, dispatched the same way.

The chain follows real dependency edges and bottoms out at Elements and Globals. Two providers with no dependency on each other are dispatched and interrogated in parallel, as separate team members working simultaneously; serialize only along a genuine edge (e.g. Presentation → Logic → Data when a value actually flows through Logic).

### Step 3 — Merge duplicates and convergence

As **coordinator**, run [Merging Duplicates](../../agents/coordinator/actions/merging-duplicates.md) whenever two contract chains reach the same provider item, or two contracts share a canonical key (target node + ratified output-shape signature). Do this continuously as Step 2 proceeds, not as an afterthought — it keeps the historical-demand counters in every node's `REQUESTS.md` trustworthy.

### Step 4 — Assemble the plan

Write `.claude/trellis/requests/<slug>/PLAN.md` from `${CLAUDE_SKILL_DIR}/templates/PLAN.md`:

- The request, verbatim.
- Every entry node.
- The full contract graph — every contract, its consumer, provider, verdict, and status.
- A Met / Modify / Create breakdown.
- Proposed build waves — a first pass at [Sequencing Build Waves](../../agents/coordinator/actions/sequencing-build-waves.md) (Globals + Elements first, then anything whose dependencies are satisfied), refined for real once the plan is approved and `building-items` runs.
- Any escalations that occurred and how they were resolved.

Status: `draft`.

### Step 5 — Human sign-off gate

Use `EnterPlanMode` if not already in a planning context, present the assembled plan (the node/contract graph, the Met/Modify/Create breakdown, the proposed build waves — as a navigable summary, not a wall of raw file contents), and call `ExitPlanMode` to request approval. Build must not proceed without this.

### Step 6 — Confirm

On approval, set `PLAN.md`'s status to `approved`. Report that `/trellis:building-items <slug>` is ready to run. On rejection with feedback, revise the plan (re-running Steps 1–4 as needed for what changed) and return to Step 5.

---

## Output

- `.claude/trellis/requests/<slug>/PLAN.md` — status `draft` while scoping, `approved` once signed off
- New item leaves may be scaffolded (via node-agent's Creating Items action) for Unmet contracts that passed the reuse guardrails — `CONTRACT.md`/`USAGE.md` stubs only, status `proposed`; no code is written yet
- Updated `REQUESTS.md` rows in any node where a need didn't clear the creation guardrails

## Rules

- Never skip the human sign-off gate — no contract is built on until the plan is approved.
- Never resolve an unconverged Countered contract without `AskUserQuestion` — that's the coordinator's job, and it never self-resolves a design tie.
- Never dispatch a node-agent without `--node` — every instantiation must declare which node it governs.
- Never treat a request as single-rooted by default — always run [Classifying](../../agents/coordinator/actions/classifying.md) against the request itself first.
