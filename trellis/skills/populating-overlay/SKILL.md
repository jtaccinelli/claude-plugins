---
name: populating-overlay
description: Backfill item leaves for code that already exists in the project — runs the same classification process as scoping-requests, but against the existing repo instead of a new request. A one-shot bootstrap, only runs on a fully empty overlay.
argument-hint: ""
---

# Populating Overlay

Adopting Trellis onto a project that already has code means `INVENTORY.md` starts empty everywhere — every first request would hit Unmet for things that already exist, scaffolding duplicates instead of finding what's already there. This skill runs the same identification process `scoping-requests` runs on a new request — the coordinator's Classifying action — but against the artifacts already sitting in the repo, so the overlay reflects reality before the first real request is ever scoped.

This is a **one-shot bootstrap**, not a recurring scan. It only runs when no cell has any items logged yet — the initial run is the record; there is no separate log file to maintain. Once any item exists anywhere in the overlay (from this run or from real work through `building-items`), this skill refuses to run again — anything new from that point on goes through `scoping-requests` like any other request.

Requires `.claude/trellis/MANIFEST.md` to exist — run `/trellis:initializing-overlay` first if it doesn't.

---

## Pre-work

1. Confirm `.claude/trellis/MANIFEST.md` exists. If not, stop and tell the user to run `/trellis:initializing-overlay` first — the placement map has to exist before there's anywhere to look.
2. Read `trellis.config.json` in full — `sourceRoot`, `placement`, `coreFiles`.
3. **Check for prior population.** Glob every cell's `INVENTORY.md` (`.claude/trellis/*/*/INVENTORY.md`) and check whether any already lists at least one item. If so, stop and report: the overlay already has items — from an earlier population run or from real work through `building-items` — so this skill will not run again. Point the user to `/trellis:scoping-requests` for anything new.

---

## Steps

### Step 1 — Discover candidate artifacts, per cell

For each of the 18 cells, resolve its real directory from `placement["<concern>.<layer>"]` (with `{item}` stripped back to the containing directory) and walk it. A candidate artifact is any subdirectory or file matching the `coreFiles` convention (e.g. an `index.*` file) — the same shape an item built through Trellis would have.

Exclude, same as Slated's `scout`:
- `node_modules/`, build output (`.next/`, `dist/`, `.output/`, `build/`)
- Lock files, generated type files, cache directories (`.cache/`, `.turbo/`, `.wrangler/`)
- Anything inside `.claude/` — framework artefacts are never scanned as project code

A file with no matching `coreFiles` entry, or sitting outside every cell's placement directory entirely, is out of scope for this skill — not every file in a repo needs to become a Trellis item, only the ones the placement map claims.

This step may run in parallel across all 18 cells — discovery within one cell has no dependency on any other.

### Step 2 — Classify each candidate

For each candidate artifact, read it and, as **coordinator**, run [Classifying](../../agents/coordinator/actions/classifying.md) against it — same decision matrix as a request, but the "request or need, stated in plain language" input is instead a description of what the artifact actually does, derived from reading it: does it render, transform, or cross a boundary; is it entity-bound; what does it compose.

### Step 3 — Backfill or flag

- If the classification **matches** the cell whose directory the artifact was found in: dispatch that cell's **cell-agent** (`--cell <concern.layer>`) to run [Backfilling Items](../../agents/cell-agent/actions/backfilling-items.md) for it.
- If the classification **disagrees** with the directory it was found in: do not backfill it into either cell. Record it as a finding — this is existing organisational debt the scan surfaced, not something Trellis silently resolves by picking a side.

Cell agents backfill in parallel across cells; within a cell, items backfill sequentially since they all write to the same `INVENTORY.md` and potentially the same `AGENT.md` Learned Patterns log.

### Step 4 — Human review gate

Use `AskUserQuestion` to surface a summary before anything is left in place: cells scanned, items backfilled per cell, and every finding from Step 3 and from each cell-agent's Backfilling Items pass (ambiguous artifacts, contradictory conventions between backfilled items in the same cell). This mirrors Slated's `scout` — never leave scaffolded item leaves in place without the user having seen what was found. If the user flags a specific backfilled item as wrong, revise or remove that item leaf before finishing.

---

## Output

- New item leaves (`CONTRACT.md`, `USAGE.md`, `CHANGELOG.md`), pointer-only, status `satisfied` — no code written, since the code already existed
- Regenerated `INVENTORY.md` per touched cell, `REFERENCES.md` per backfilled item
- `AGENT.md` Learned Patterns extended per cell where a consistent convention was actually observed across backfilled items

## Rules

- Never run when any cell's `INVENTORY.md` already lists an item — this is a one-shot bootstrap for a fully empty overlay, not a recurring or partial catch-up scan.
- Never write anything without the Step 4 human review gate — same discipline as Slated's `scout` confirming before writing `locations.md`.
- Never backfill an artifact into a cell that Classifying disagrees with — flag it instead of silently trusting the folder it happened to be found in.
- Never silently resolve a contradiction between backfilled items in the same cell — record it as a finding.
