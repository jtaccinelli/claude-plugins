---
name: shoot-scene
description: Run a scene to completion — iterating through takes automatically until one passes, then finalising the scene without returning control between steps.
expected-role: director
argument-hint: [scene name]
---

# Shoot Scene

Run a full scene from first take to completion. The director executes takes in sequence using the shoot-take process, feeding next-take instructions back automatically until a take passes, then immediately runs the wrap-scene process to close out the scene.

---

## Pre-work

1. Resolve the scene from `$ARGUMENTS` — find `.claude/slated/scenes/scene-<name>/manuscript.md`
2. Read the manuscript in full — cast, resources, objectives, and all shots
3. Confirm the manuscript status is `confirmed` or `in-progress` — if neither, stop and report that the scene is not ready to shoot
4. Glob `.claude/slated/scenes/scene-<name>/takes/take-*.md` — if any take files exist, read them all in full before proceeding; the director must enter the loop with full context on what has already been attempted and why it failed
5. If the most recent take file (if any) has status `pass`, skip the take loop entirely — proceed directly to Step 2 (wrap the scene); do not run another take
6. Load the shoot-take and wrap-scene skills — these are the two processes this skill orchestrates

---

## Process

### Step 1 — Execute the take loop

Repeat until a take passes:

1. Execute the shoot-take process in full for the current scene — with one exception: do not stop and return control to the user at the end of the take. Instead, read the take file that was written and use the verdict to determine the next step.
2. If the take **failed**: surface a brief status update to the user, then continue to the next iteration:
   ```
   Take <NNN> — FAIL. Beginning take <NNN+1>.
   ```
3. If the take **passed**: exit the loop and proceed to Step 2.

### Step 2 — Wrap the scene

Execute the wrap-scene process in full for the current scene.

### Step 3 — Report completion

Surface a clear summary:

```
Scene <name> — COMPLETE

Completed in <N> take(s).
wrap.md written. Storyboard updated.
```

---

## Output

- `.claude/slated/scenes/scene-<name>/manuscript.md` — status set to `complete`, Completion Record appended
- `.claude/slated/scenes/scene-<name>/takes/take-<NNN>.md` — one file per take executed
- `.claude/slated/scenes/scene-<name>/wrap.md` — written on completion
- `.claude/slated/scenes/storyboard.md` — scene moved from Pending to Completed
- Completion summary returned to user with take count

---

## Rules

- Never run a scene whose status is not `confirmed` or `in-progress`
- Never skip the wrap-scene process after a passing take — the scene is not done until the Completion Record, wrap.md, and storyboard are all updated
- Never return control to the user between takes — the take loop runs uninterrupted until a take passes, with two permitted pause points: the writer confirmation pause when manuscript issues are identified in shoot-take Step 7, and the wrap-scene Role Improvement Notes pause after the loop completes
- All rules defined in the shoot-take and wrap-scene skills remain in effect when those processes are executed here
