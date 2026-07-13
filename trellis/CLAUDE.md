# Framework: Layered Coordination

This project uses Trellis — a coordination overlay that classifies every artifact by **Concern** (Presentation / Logic / Data) and **Layer** (Element → Component → Module → Feature → Page, plus a Globals plane), then governs work through that grid. The overlay lives entirely under `.claude/trellis/`; the code it governs lives wherever the host project already puts it (`src/`, `app/`, `lib/`, …). Trellis is the lattice, not the plant — it holds the shape, the host project holds the code.

---

## Folder Reference

### Framework (`trellis/`)

Bundled with the plugin, always available regardless of project:

| Path | Purpose |
|---|---|
| `trellis/agents/coordinator.md` | The router — classifies requests, routes contracts, sequences builds, reviews fidelity |
| `trellis/agents/node-agent.md` | The governor — one persistent agent type, instantiated per node via `--node` |
| `trellis/skills/initializing-overlay/` | Scaffold `.claude/trellis/` for a project (one-time, idempotent) |
| `trellis/skills/populating-overlay/` | One-shot backfill of item leaves for code that already exists in the project |
| `trellis/skills/scoping-requests/` | Classify a request, run the contract handshake, get human sign-off on a build plan |
| `trellis/skills/building-items/` | Execute a ratified plan — worktree-per-attempt, fidelity review, wrap + PR |
| `trellis/skills/visualizing-consumption/` | Publish an on-demand graph of item consume/consumed-by relationships (read-only, no agent dispatch) |

### Project-level (`.claude/trellis/`)

| Path | Purpose |
|---|---|
| `.claude/trellis/MANIFEST.md` | Entry point every agent reads first |
| `.claude/trellis/trellis.config.json` | Placement map (node → real host path) + settings — the single source of truth for where code actually lives |
| `.claude/trellis/<concern>/<NN-layer>/` | One of 18 nodes — `AGENT.md` (remit + Learned Patterns log), `INVENTORY.md` (generated), `REQUESTS.md` |
| `.claude/trellis/<concern>/<NN-layer>/<item>/` | An item leaf — `CONTRACT.md`, `USAGE.md`, `CHANGELOG.md`, `REFERENCES.md` (generated), `attempts/` |
| `.claude/trellis/<concern>/globals/` | The Globals node for that concern — same file shape as a layer node |
| `.claude/trellis/requests/<slug>/` | One classified request's session record — `PLAN.md`, `SUMMARY.md` |

---

## How It Works

1. **Concern** is the track an artifact serves — Presentation (renders), Logic (transforms/decides/adapts/holds state), or Data (crosses a boundary).
2. **Layer** is compositional granularity — Element → Component → Module → Feature → Page — resolved by two discriminators: is the artifact entity-bound, and what does it compose.
3. **Globals** is an off-ladder plane for ambient artifacts (stores, providers, theme, auth, i18n) provided once at the root and consumed everywhere — no owning layer.
4. A **Node** is one Concern × Layer slot (or Concern × Globals) — one governing `node-agent` instantiation, one `INVENTORY.md`, one `REQUESTS.md`. All 18 nodes are fixed and framework-scaffolded; none are discovered or authored per project.
5. An **Item** is a single named thing living inside a node — a component, a service, an endpoint — governed by its own `CONTRACT.md` / `USAGE.md` / `CHANGELOG.md`, with a `Source` pointer to the real code. Items are created lazily, only when the reuse guardrails hold.
6. The **coordinator** classifies work and routes **contracts** between nodes — the consumer always authors the contract; the provider returns a verdict (Met / Partial · additive / Partial · breaking / Unmet / Countered).
7. Unresolved consumer/provider conflicts escalate to the user — the coordinator has no design authority and never breaks a design tie.
8. Each node also accumulates a **Learned Patterns** log in its own `AGENT.md` — naming, structure, and file-organisation conventions derived only from what the node has actually built, never prescribed up front. The coordinator checks every build attempt against this log during Reviewing Fidelity, and the node agent extends it during Self-Documenting. A pattern is never reversed silently — a change that contradicts a prior entry escalates to the user, the same discipline as a cast/background Refinement Log.
9. Every workflow runs like a real team, not a solo pass. A node agent is never just "run" — it's dispatched with a specific work order and expected to report back with something the dispatcher can act on: a verdict, a delivered artifact, a finding. Whenever multiple node agents' work is independent, they're dispatched together, at once, and every report is collected before the coordinator proceeds — never serialized just for convenience.

---

## Lifecycle

### Initialisation

1. Run `/trellis:initializing-overlay` once per project — scaffolds `.claude/trellis/` (manifest, config, all 18 nodes). Idempotent; safe to re-run to check status.

### Population (existing projects only)

2. Run `/trellis:populating-overlay` — a one-shot bootstrap that backfills item leaves for code that already exists, so the overlay reflects reality before the first real request is scoped. Only runs while every node's `INVENTORY.md` is still empty; refuses to run again once anything has been captured, whether by this skill or by real work through `building-items`.

### Scoping

3. Run `/trellis:scoping-requests <description>` — the coordinator classifies the request into entry node(s), the consumer-authors-the-contract chain runs across every node the work touches, and a consolidated build plan is presented for human sign-off before anything is written.

### Building

4. Run `/trellis:building-items <request-slug>` — node agents build each item in dependency-ordered waves, one worktree per attempt, with the coordinator reviewing contract fidelity after every attempt. Wraps automatically on completion: summary written, PR opened.

### Visualizing (any time, not part of the sequence)

Run `/trellis:visualizing-consumption` whenever you want to see the current consume/consumed-by graph — it doesn't gate or feed into any other step, and can run as often as useful once items exist. Purely a read of existing `CONTRACT.md`/`REFERENCES.md` documentation, rendered as a published artifact; nothing is written back to the overlay.

---

## Agents

| Agent | Responsibility | Behaviour varies by node? |
|---|---|---|
| `coordinator` | Classification, contract routing, build sequencing, fidelity review | No — one fixed definition |
| `node-agent` | Assessing/creating/building items within one node | No — one fixed definition; context comes from the node's `AGENT.md`, not a different identity per node |

### `coordinator` actions

Classifying · Routing Contracts · Reconciling Verdicts · Merging Duplicates · Sequencing Build Waves · Reviewing Fidelity

### `node-agent` actions

Assessing Contracts · Creating Items · Executing Build Attempts · Self-Documenting · Consolidating · Backfilling Items

Each agent's `.md` file carries a Trigger/Action table; the full Trigger / Inputs / Procedure / Output for each action lives in its own file under `trellis/agents/<agent>/actions/`.

---

## Skills Reference

| Skill | Purpose |
|---|---|
| `/trellis:initializing-overlay` | Scaffold `.claude/trellis/` — manifest, config, all 18 nodes |
| `/trellis:populating-overlay` | One-shot backfill of item leaves for pre-existing code — only runs on a fully empty overlay |
| `/trellis:scoping-requests` | Classify a request, run the contract handshake, get human sign-off |
| `/trellis:building-items` | Execute a ratified plan — worktree-per-attempt build loop, wrap + PR |
| `/trellis:visualizing-consumption` | On-demand, read-only graph of item consume/consumed-by relationships |

---

## Naming Conventions

- Nodes: `.claude/trellis/<presentation\|logic\|data>/<01-element\|02-component\|03-module\|04-feature\|05-page\|globals>/`
- Items: `.claude/trellis/<node>/<item-slug>/` — kebab-case
- Item attempts: `.claude/trellis/<node>/<item-slug>/attempts/ATTEMPT_NNN.md`
- Requests: `.claude/trellis/requests/<request-slug>/`
- Placement map (real code location): `trellis.config.json`'s `placement["<concern>.<layer>"]`, with `{item}` substituted for the item slug
