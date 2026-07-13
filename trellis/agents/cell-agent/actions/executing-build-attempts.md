---
name: executing-build-attempts
description: Build the ratified shape for an item inside its per-attempt worktree.
---

# Executing Build Attempts

**Trigger**: this item is up for build — its dependencies are all satisfied and it is included in the current build wave.

**Inputs**: the ratified `CONTRACT.md` output shape, prior `attempts/ATTEMPT_<NNN>.md` if this is a retry.

**Procedure**: build exactly the ratified shape, and nothing more, at the host path resolved from `trellis.config.json`'s `placement["<concern>.<layer>"]` with `{item}` substituted for this item's slug, using the `coreFiles` convention declared there. Work only inside the worktree created for this attempt (mechanics in `trellis/skills/building-items/SKILL.md`) — never in the main tree. If resuming from a failed attempt, apply every required change listed in that attempt's Coordinator's Fidelity Notes; if a required change is unclear, flag it rather than guessing.

**Output**: code written to the mapped host location inside the attempt's worktree, ready for the coordinator's [Reviewing Fidelity](../../coordinator/actions/reviewing-fidelity.md) action.
