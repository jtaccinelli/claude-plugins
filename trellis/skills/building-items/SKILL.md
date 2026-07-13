---
name: building-items
description: Execute an approved request plan — build every item in dependency-ordered waves, one worktree per attempt, with coordinator fidelity review after each attempt, then wrap with a summary and PR.
argument-hint: "<request-slug>"
---

# Building Items

Run an approved plan to completion. For each build wave, every ready item is built by its owning cell-agent inside its own worktree, reviewed by the coordinator for contract fidelity, retried on failure up to a cap, and self-documented on pass. When every item across every wave has landed (or is blocked), the coordinator wraps the request inline: `SUMMARY.md` written, PR opened — without returning control to the user between waves.

---

## Pre-work

1. Resolve the request from `$ARGUMENTS` — find `.claude/trellis/requests/<slug>/PLAN.md`. If not found, stop and report that the request has not been scoped; run `/trellis:scoping-requests` first.
2. Read `PLAN.md` in full — entry cells, contract graph, Met/Modify/Create breakdown, proposed build waves.
3. Confirm `PLAN.md`'s status is `approved`. If not, stop and report the plan still needs sign-off via `scoping-requests`.
4. As **coordinator**, run [Sequencing Build Waves](../../agents/coordinator/actions/sequencing-build-waves.md) on the plan's full contract graph — this is the authoritative build order, superseding `PLAN.md`'s first-pass proposal.
5. **Assess resume state**: for every item slated to build, check its `CONTRACT.md` status.
   - `satisfied` — already built in a prior run of this skill; skip it.
   - `create` / `proposed` with existing `attempts/ATTEMPT_*.md` — this is a **resume**; the next attempt continues from the count of existing attempt files. Surface a brief status note: "Resuming `<item>` from attempt `<N>`."
   - `create` / `proposed` with no attempt files — fresh start for this item.
6. **Create the request branch** if it does not already exist: `git branch --list request/<slug>`; if absent, `git branch request/<slug>` from current HEAD. This branch is never auto-merged to main — it is the PR branch, exactly like Slated's scene branch.

---

## Build Loop

Process waves in the order [Sequencing Build Waves](../../agents/coordinator/actions/sequencing-build-waves.md) returned. Within a wave, every item builds independently — a wave is constructed so that no item in it still has an unmet dependency, but two items in the same wave may still write to different files, so check explicitly before parallelizing: an item may build in parallel with another only if neither's placement path overlaps the other's AND neither's `CONTRACT.md` lists the other as a dependency. Otherwise, serialize them within the wave.

For each item, run the following attempt loop (maximum **5 attempts** per item, counted across all runs of this skill on this item):

### Step 1 — Create the attempt worktree

- Derive the attempt branch: `attempt/<slug>-<cell>-<item-slug>-<NNN>` (NNN = zero-padded attempt count).
- Derive the worktree path: `.worktrees/attempt-<slug>-<cell>-<item-slug>-<NNN>` (relative to repo root).
- Create the worktree from the request branch: `git worktree add -b attempt/<slug>-<cell>-<item-slug>-<NNN> .worktrees/attempt-<slug>-<cell>-<item-slug>-<NNN> request/<slug>`.
- If the attempt branch already exists (stale from a previous run): force-delete it (`git branch -D`), remove any stale worktree (`git worktree remove --force`), then retry. If the retry fails, stop and report — do not proceed until the worktree is clean.

### Step 2 — Dispatch the cell agent

Spawn a sub-agent using the **cell-agent** agent, `--cell <concern.layer>`, instructing it to run [Executing Build Attempts](../../agents/cell-agent/actions/executing-build-attempts.md):

- The item's ratified `CONTRACT.md` output shape.
- The absolute worktree path as its working directory — all file writes for code must occur within this path, resolved against `trellis.config.json`'s placement map.
- An explicit constraint: never write `.claude/trellis/` framework artefacts (`CONTRACT.md`, `USAGE.md`, `CHANGELOG.md`, attempt files) from inside the worktree — those are written to the main tree only, in Step 5/6 below.
- If resuming, the path to the most recent `attempts/ATTEMPT_<NNN>.md` in the main tree — instruct it to read the Coordinator's Fidelity Notes and Next Attempt Instructions in full and apply every required change.

Wait for the cell agent to complete.

### Step 3 — Capture changed files

Run `git -C <worktree-path> status --short`. Parse into a structured Added/Modified/Deleted list — this is the canonical record; never infer the file list from the cell agent's own report.

### Step 4 — Reviewing Fidelity

As **coordinator**, run [Reviewing Fidelity](../../agents/coordinator/actions/reviewing-fidelity.md): compare the delivered artifact (from the worktree) against the item's ratified `CONTRACT.md` output shape and against every pattern already logged in this cell's `AGENT.md` Learned Patterns. Record the verdict and, on failure, the specific gap: what was expected — citing the contract or the specific logged pattern — what was delivered, and where.

### Step 5 — Write the attempt file

Write `.claude/trellis/<concern>/<NN-layer>/<item-slug>/attempts/ATTEMPT_<NNN>.md` in the **main tree**, from `${CLAUDE_SKILL_DIR}/templates/ATTEMPT_001.md`:

- **Files Changed** — the Step 3 list.
- **Cell Agent Summary** — factual account of what was built.
- **Coordinator's Fidelity Notes** — verdict plus, on failure, the specific shape gap.
- **Next Attempt Instructions** — omit if the attempt passed; otherwise a specific, actionable list of what must differ next time.

### Step 6 — On pass: self-document and merge

1. Spawn a sub-agent using the **cell-agent** agent, `--cell <concern.layer>`, to run [Self-Documenting](../../agents/cell-agent/actions/self-documenting.md) in the main tree: set the `CONTRACT.md` `Source` pointer(s) to the resolved host path(s), write/update `USAGE.md`, append `CHANGELOG.md`, regenerate this item's `REFERENCES.md` and the cell's `INVENTORY.md`, and log or reconcile any pattern finding into the cell's `AGENT.md` Learned Patterns. Set `CONTRACT.md` status to `satisfied`.
2. Merge into the request branch: `git checkout request/<slug> && git merge attempt/<slug>-<cell>-<item-slug>-<NNN> --no-ff -m "item(<cell>/<item-slug>): merge attempt-<NNN>"`. Return to the original branch: `git checkout -`.
3. Remove the worktree: `git worktree remove .worktrees/attempt-<slug>-<cell>-<item-slug>-<NNN>`. Delete the attempt branch: `git branch -d attempt/<slug>-<cell>-<item-slug>-<NNN>`.
4. Mark this item complete; if any other item was waiting on this one as a dependency and now has all dependencies satisfied, it becomes eligible for the current or next wave.

### Step 7 — On fail: clean up and retry

1. Force-remove the worktree: `git worktree remove --force .worktrees/attempt-<slug>-<cell>-<item-slug>-<NNN>`. Delete the attempt branch: `git branch -D attempt/<slug>-<cell>-<item-slug>-<NNN>`.
2. If the attempt count for this item is below 5: increment and return to Step 1 for this item.
3. If the attempt count has reached 5: mark this item **BLOCKED**. Any item elsewhere in the plan that depends on this one is also marked BLOCKED (dependency never satisfied) — do not attempt to build it. Continue building every other independent item in the plan; a BLOCKED item does not halt the rest of the run.

---

## Wrap Process

Executed inline once every item across every wave has either landed or been marked BLOCKED. No user confirmation required unless there are BLOCKED items (see below).

### Wrap Step 1 — Compile the summary

Gather from every item's attempt files and final status:
- Which items built successfully, and in how many attempts each.
- Which items are BLOCKED, and the persisting fidelity gap from their final attempt.
- Any escalations that occurred during scoping (from `PLAN.md`).

### Wrap Step 2 — Write SUMMARY.md

Write `.claude/trellis/requests/<slug>/SUMMARY.md` from `${CLAUDE_SKILL_DIR}/templates/SUMMARY.md`:
- **Description** — 2–4 plain-language sentences: what capability was added, grouped by cell.
- **Items** — every item built, grouped by cell, with a one-line description each (draw from each item's `USAGE.md`).
- **Blocked** — any BLOCKED items and why; omit if none.

Update `PLAN.md`'s status to `complete` if every item landed, or leave it `approved` with a note if any items are BLOCKED.

### Wrap Step 3 — Open a pull request

1. Push: `git push -u origin request/<slug>`.
2. Create the PR: `gh pr create --title "request(<slug>): complete" --body "$(cat <<'EOF'
<Description section from SUMMARY.md, verbatim>

## Items
<Items section from SUMMARY.md, verbatim>
EOF
)"`.

If `gh` is unavailable or push/PR creation fails, surface fallback instructions (push and `gh pr create --base main` manually).

### Wrap Step 4 — Report completion

```
Request <slug> — <COMPLETE | PARTIAL>

<N> item(s) built. SUMMARY.md written.
PR: <url>
```

If any items are BLOCKED, list them and use `AskUserQuestion` to pause: "Some items could not be built after 5 attempts. Review the notes above — revise the contract via `/trellis:scoping-requests` or make a manual correction, then re-run `/trellis:building-items <slug>`." Do not return until the user acknowledges.

---

## Output

- `.claude/trellis/<cell>/<item>/attempts/ATTEMPT_<NNN>.md` — attempt file per attempt executed
- `.claude/trellis/<cell>/<item>/{CONTRACT,USAGE,CHANGELOG,REFERENCES}.md` — updated on pass
- `.claude/trellis/<cell>/INVENTORY.md` — regenerated on pass
- `.claude/trellis/<cell>/AGENT.md` — Learned Patterns log appended on pass, only when a new pattern is confirmed
- `.claude/trellis/requests/<slug>/SUMMARY.md` — written on wrap
- PR opened from `request/<slug>` into `main` (or fallback instructions surfaced)

## Rules

- Never run a request whose `PLAN.md` status is not `approved`.
- Never dispatch a cell-agent without `--cell`.
- Never skip Reviewing Fidelity — an attempt is never marked passed without it.
- Never infer the changed file list from cell-agent output — always derive from `git status --short` in the worktree.
- Never write an attempt file, or any `.claude/trellis/` framework artefact, inside a worktree — always the main tree.
- Never merge a passing attempt into `main` — the request branch only.
- Always clean up the worktree before proceeding to the next attempt or item — never leave an orphaned worktree or stale attempt branch.
- Never exceed 5 attempts per item — hard stop at the limit, mark BLOCKED, and continue with independent items.
- Never create an attempt worktree directly from `main` — always from the request branch.
- If a previous attempt exists for an item, always pass its attempt file path to the cell agent — never run a subsequent attempt without it having access to prior Fidelity Notes.
- Never stop on a fully-passing run before wrap — wrap runs immediately after the last item lands, without returning control to the user.
