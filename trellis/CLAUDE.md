# Framework: Layered Coordination

This project uses Trellis — a coordination overlay that classifies every artifact by **Concern** (Presentation / Logic / Data) and **Layer** (Element → Component → Module → Feature → Page, plus a Globals plane), then governs work through that grid. The overlay lives entirely under `.claude/trellis/`; the code it governs lives wherever the host project already puts it (`src/`, `app/`, `lib/`, …). Trellis is the lattice, not the plant — it holds the shape, the host project holds the code.

---

## Folder Reference

### Framework (`trellis/`)

Bundled with the plugin, always available regardless of project:

| Path | Purpose |
|---|---|
| `trellis/agents/coordinator.md` | The router — classifies requests, routes contracts, sequences builds, reviews fidelity |
| `trellis/agents/cell-agent.md` | The governor — one persistent agent type, instantiated per cell via `--cell` |
| `trellis/skills/initializing-overlay/` | Scaffold `.claude/trellis/` for a project (one-time, idempotent) |
| `trellis/skills/populating-overlay/` | Backfill item leaves for code that already exists in the project (one-time or occasional catch-up) |
| `trellis/skills/scoping-requests/` | Classify a request, run the contract handshake, get human sign-off on a build plan |
| `trellis/skills/building-items/` | Execute a ratified plan — worktree-per-attempt, fidelity review, wrap + PR |

### Project-level (`.claude/trellis/`)

| Path | Purpose |
|---|---|
| `.claude/trellis/MANIFEST.md` | Entry point every agent reads first |
| `.claude/trellis/trellis.config.json` | Placement map (cell → real host path) + settings — the single source of truth for where code actually lives |
| `.claude/trellis/<concern>/<NN-layer>/` | One of 18 cells — `AGENT.md` (remit + Learned Patterns log), `INVENTORY.md` (generated), `REQUESTS.md` |
| `.claude/trellis/<concern>/<NN-layer>/<item>/` | An item leaf — `CONTRACT.md`, `USAGE.md`, `CHANGELOG.md`, `REFERENCES.md` (generated), `attempts/` |
| `.claude/trellis/<concern>/globals/` | The Globals cell for that concern — same file shape as a layer cell |
| `.claude/trellis/requests/<slug>/` | One classified request's session record — `PLAN.md`, `SUMMARY.md` |

---

## How It Works

1. **Concern** is the track an artifact serves — Presentation (renders), Logic (transforms/decides/adapts/holds state), or Data (crosses a boundary).
2. **Layer** is compositional granularity — Element → Component → Module → Feature → Page — resolved by two discriminators: is the artifact entity-bound, and what does it compose.
3. **Globals** is an off-ladder plane for ambient artifacts (stores, providers, theme, auth, i18n) provided once at the root and consumed everywhere — no owning layer.
4. A **Cell** is one Concern × Layer slot (or Concern × Globals) — one governing `cell-agent` instantiation, one `INVENTORY.md`, one `REQUESTS.md`. All 18 cells are fixed and framework-scaffolded; none are discovered or authored per project.
5. An **Item** is a single named thing living inside a cell — a component, a service, an endpoint — governed by its own `CONTRACT.md` / `USAGE.md` / `CHANGELOG.md`, with a `Source` pointer to the real code. Items are created lazily, only when the reuse guardrails hold.
6. The **coordinator** classifies work and routes **contracts** between cells — the consumer always authors the contract; the provider returns a verdict (Met / Partial · additive / Partial · breaking / Unmet / Countered).
7. Unresolved consumer/provider conflicts escalate to the user — the coordinator has no design authority and never breaks a design tie.
8. Each cell also accumulates a **Learned Patterns** log in its own `AGENT.md` — naming, structure, and file-organisation conventions derived only from what the cell has actually built, never prescribed up front. The coordinator checks every build attempt against this log during Reviewing Fidelity, and the cell agent extends it during Self-Documenting. A pattern is never reversed silently — a change that contradicts a prior entry escalates to the user, the same discipline as a cast/background Refinement Log.

---

## Lifecycle

### Initialisation

1. Run `/trellis:initializing-overlay` once per project — scaffolds `.claude/trellis/` (manifest, config, all 18 cells). Idempotent; safe to re-run to check status.

### Population (existing projects only)

2. Run `/trellis:populating-overlay` — a one-shot bootstrap that backfills item leaves for code that already exists, so the overlay reflects reality before the first real request is scoped. Only runs while every cell's `INVENTORY.md` is still empty; refuses to run again once anything has been captured, whether by this skill or by real work through `building-items`.

### Scoping

3. Run `/trellis:scoping-requests <description>` — the coordinator classifies the request into entry cell(s), the consumer-authors-the-contract chain runs across every cell the work touches, and a consolidated build plan is presented for human sign-off before anything is written.

### Building

4. Run `/trellis:building-items <request-slug>` — cell agents build each item in dependency-ordered waves, one worktree per attempt, with the coordinator reviewing contract fidelity after every attempt. Wraps automatically on completion: summary written, PR opened.

---

## Agents

| Agent | Responsibility | Behaviour varies by cell? |
|---|---|---|
| `coordinator` | Classification, contract routing, build sequencing, fidelity review | No — one fixed definition |
| `cell-agent` | Assessing/creating/building items within one cell | No — one fixed definition; context comes from the cell's `AGENT.md`, not a different identity per cell |

### `coordinator` actions

Classifying · Routing Contracts · Reconciling Verdicts · Merging Duplicates · Sequencing Build Waves · Reviewing Fidelity

### `cell-agent` actions

Assessing Contracts · Creating Items · Executing Build Attempts · Self-Documenting · Consolidating · Backfilling Items

Each agent's `.md` file carries a Trigger/Action table; the full Trigger / Inputs / Procedure / Output for each action lives in its own file under `trellis/agents/<agent>/actions/`.

---

## Skills Reference

| Skill | Purpose |
|---|---|
| `/trellis:initializing-overlay` | Scaffold `.claude/trellis/` — manifest, config, all 18 cells |
| `/trellis:populating-overlay` | One-shot backfill of item leaves for pre-existing code — only runs on a fully empty overlay |
| `/trellis:scoping-requests` | Classify a request, run the contract handshake, get human sign-off |
| `/trellis:building-items` | Execute a ratified plan — worktree-per-attempt build loop, wrap + PR |

---

## Naming Conventions

- Cells: `.claude/trellis/<presentation\|logic\|data>/<01-element\|02-component\|03-module\|04-feature\|05-page\|globals>/`
- Items: `.claude/trellis/<cell>/<item-slug>/` — kebab-case
- Item attempts: `.claude/trellis/<cell>/<item-slug>/attempts/ATTEMPT_NNN.md`
- Requests: `.claude/trellis/requests/<request-slug>/`
- Placement map (real code location): `trellis.config.json`'s `placement["<concern>.<layer>"]`, with `{item}` substituted for the item slug
