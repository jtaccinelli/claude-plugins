---
name: initializing-overlay
description: Scaffold the Trellis overlay (.claude/trellis/) for a project — manifest, placement config, and all 18 cells. One-time, idempotent.
argument-hint: ""
---

# Initializing Overlay

Scaffold `.claude/trellis/` for this project: the manifest, the placement config, and all 18 fixed Concern × Layer cells (plus the three Globals cells). This is the only skill that runs before any request can be scoped or built — `scoping-requests` and `building-items` both assume the overlay already exists.

---

## Pre-work

1. Check whether `.claude/trellis/MANIFEST.md` already exists.
   - **If it exists**: this project is already initialised. Glob `.claude/trellis/*/*/AGENT.md` to confirm all 18 cells are present, report the current state (cells present, any missing, item counts per `INVENTORY.md`), and stop. Do not overwrite anything — re-running this skill is a status check, never a destructive re-scaffold.
   - **If it does not exist**: proceed.

---

## Steps

### Step 1 — Determine the placement map

`trellis.config.json`'s `placement` map is inherently project-specific — it cannot be hardcoded in the plugin. Determine it by:

1. Inspecting the project for stack signals (`package.json` dependencies, existing `src/`/`app/`/`lib/` directories, framework config files) to propose a default mapping.
2. Presenting the proposed `sourceRoot` and all 18 `placement` entries to the user via `AskUserQuestion` (or a plain confirmation if the defaults are obvious) before writing anything.

If the project has no discernible stack yet (a truly empty repo), ask the user directly what source layout to use.

### Step 2 — Write the manifest and config

Write `.claude/trellis/MANIFEST.md` from `${CLAUDE_SKILL_DIR}/templates/MANIFEST.md` verbatim — it is static, not templated.

Write `.claude/trellis/trellis.config.json` from `${CLAUDE_SKILL_DIR}/templates/trellis-config.json`, substituting `sourceRoot` and every `placement` value determined in Step 1. Leave `coreFiles` as-is unless the project's stack uses different entry-file conventions.

### Step 3 — Scaffold all 18 cells

For each of the three concerns (`presentation`, `logic`, `data`) and each of the six layers (`01-element`, `02-component`, `03-module`, `04-feature`, `05-page`, `globals`) — 18 combinations total:

1. Create the cell directory: `.claude/trellis/<concern>/<layer>/`.
2. Write `AGENT.md` from `${CLAUDE_SKILL_DIR}/templates/AGENT.md`, substituting `{{concern}}` (title case, e.g. "Presentation"), `{{layer}}` (title case, e.g. "Component", or "Globals"), and `{{remit}}` from the Remit Table below — this text is fully determined by concern + layer, never authored per project.
3. Write `INVENTORY.md` from `${CLAUDE_SKILL_DIR}/templates/INVENTORY.md`, substituting `{{concern}}`/`{{layer}}`.
4. Write `REQUESTS.md` from `${CLAUDE_SKILL_DIR}/templates/REQUESTS.md`, substituting `{{concern}}`/`{{layer}}`.

### Step 4 — Report

Report the full scaffolded tree (`.claude/trellis/` recursively) for the user to sanity-check.

---

## Remit Table

The fixed text substituted into each cell's `AGENT.md` `## Remit` section. Derived directly from the Concern × Layer decision matrix — never varies by project.

### Presentation

| Layer | Remit |
|---|---|
| Element | Owns raw design tokens — colors, spacing, radii, typography scales — indivisible, composing no other artifact. Does NOT own: anything that renders a DOM/native element (→ Component). |
| Component | Owns domain-agnostic presentation primitives that compose at most Elements (e.g. Button, Price). Does NOT own: entity-bound UI (→ Feature), multi-composite reusable UI (→ Module), ambient providers/theme (→ Globals). |
| Module | Owns domain-agnostic presentation compositions of 2+ lower artifacts, reusable in an unrelated app (e.g. Gallery, Card). Does NOT own: entity-bound UI (→ Feature), single-primitive UI (→ Component). |
| Feature | Owns entity-bound presentation wired to specific entities/data, serving a user goal (e.g. Reviews, RecCarousel). Does NOT own: domain-agnostic UI (→ Module/Component), route-owned views (→ Page). |
| Page | Owns the presentation artifact that lives at a route (e.g. the PDP view + layout). Does NOT own: reusable UI composed inside it (→ Feature/Module/Component). |
| Globals | Owns ambient presentation artifacts provided once at the root and consumed at every layer (e.g. ThemeProvider, i18n). Does NOT own: anything with a fixed layer rung (→ the matching layer cell). |

### Logic

| Layer | Remit |
|---|---|
| Element | Owns indivisible pure functions/utilities that compose no other artifact (e.g. slugify, formatPrice). Does NOT own: anything that composes other Logic artifacts (→ Component/Module). |
| Component | Owns domain-agnostic logic primitives that compose at most Elements (e.g. useToggle, validate). Does NOT own: entity-bound logic (→ Feature), multi-composite logic (→ Module). |
| Module | Owns domain-agnostic logic compositions of 2+ lower artifacts, reusable in an unrelated app (e.g. ReviewService, sumLineItems). Does NOT own: entity-bound logic (→ Feature). |
| Feature | Owns entity-bound logic wired to specific entities/data (e.g. useProduct(slug)). Does NOT own: domain-agnostic logic (→ Module/Component), route orchestration (→ Page). |
| Page | Owns logic that orchestrates a specific route. Does NOT own: reusable logic composed inside it (→ Feature/Module/Component). |
| Globals | Owns ambient state and logic provided once at the root and consumed at every layer (e.g. cartStore, session) — including all state that is not ambient, which belongs to whichever layer actually holds it, not here. Does NOT own: component-local or form state (→ the owning layer's Logic cell). |

### Data

| Layer | Remit |
|---|---|
| Element | Owns indivisible boundary-crossing constants/config that compose nothing (e.g. a base-URL const). Does NOT own: anything that composes other Data artifacts (→ Component/Module). |
| Component | Owns domain-agnostic data primitives that compose at most Elements (e.g. apiClient). Does NOT own: entity-bound data access (→ Feature), multi-composite data logic (→ Module). |
| Module | Owns domain-agnostic data compositions of 2+ lower artifacts, reusable in an unrelated app (e.g. useResource). Does NOT own: entity-bound data access (→ Feature). |
| Feature | Owns entity-bound data access wired to specific entities (e.g. productLoader). Does NOT own: domain-agnostic data primitives (→ Module/Component), route endpoints (→ Page). |
| Page | Owns the data artifact that lives at a route — GET/POST handlers. Does NOT own: data access composed inside it (→ Feature/Module/Component). |
| Globals | Owns ambient data infrastructure provided once at the root and consumed at every layer (e.g. configured client, cache). Does NOT own: anything with a fixed layer rung (→ the matching layer cell). |

---

## Output

- `.claude/trellis/MANIFEST.md`
- `.claude/trellis/trellis.config.json`
- 18 cell directories, each with `AGENT.md`, `INVENTORY.md`, `REQUESTS.md`

## Rules

- Never overwrite an existing overlay — re-running this skill on an initialised project is read-only.
- Never write a `placement` entry without user confirmation — it's the single source of truth for where code actually lands.
- Never scaffold item leaves here — items are created lazily by `cell-agent`'s Create Item capability, only when the reuse guardrails hold.
