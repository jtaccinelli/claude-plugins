---
name: shoot
description: Execute a scene — running takes until one passes, then wrapping inline, with auto-resolution of cast role, background, and manuscript issues along the way.
expected-role: director
argument-hint: "<scene-name>"
---

# Shoot

Run a scene to completion. The director executes takes against the manuscript, reviews output, auto-resolves any cast role or background gaps found, and wraps the scene inline when a take passes. A successful execution leaves the scene wrapped, `wrap.md` written, and a PR opened — without returning control to the user between steps.

---

## Pre-work

1. Run `/slated:perform director` to adopt the director role.

2. Resolve the scene from `$ARGUMENTS` — find `.claude/slated/scenes/scene-<name>/manuscript.md`. If not found, stop and report that the scene does not exist.

3. Read the manuscript in full — cast, resources, objectives, and all shots.

4. Resolve role file paths for each cast member: check `.claude/slated/roles/role-<name>.md` (project) first, then `~/.claude/slated/roles/role-<name>.md` (global). If neither exists, stop and report the missing role.

5. Resolve background file paths for each cast member: check `.claude/slated/backgrounds/background-<name>.md` (project) first, then `~/.claude/slated/backgrounds/background-<name>.md` (global). If any cannot be found, stop and report the missing background.

6. Read each resolved role and background file in full.

7. **Assess scene state**: Glob `.claude/slated/scenes/scene-<name>/takes/take-*.md`. If take files exist, read them all in full.
   - If the most recent take has status `pass` and `wrap.md` does not exist: skip the take loop — proceed directly to the Wrap process.
   - If the most recent take has status `pass` and `wrap.md` already exists: report that the scene is already wrapped and stop.
   - If take files exist and the most recent has status `fail`: this is a **resume** — continue the take loop from where it left off. Surface a brief status note: "Resuming `scene-<name>` from take <N>."
   - If no take files exist: this is a **fresh start**.

8. Confirm the manuscript status is `confirmed` or `in-progress` — if neither, stop and report that the scene is not ready to shoot.

9. **If fresh start**: use AskUserQuestion: "Do you want to shoot a single take or run the full loop (up to 5 takes)?"
   - **Single take**: run one take and return the verdict to the user. Stop after the verdict regardless of outcome (unless it passes, in which case proceed to Wrap).
   - **Full loop**: run takes automatically until one passes or the take limit is reached.

10. Determine take number — count existing take files; next take is count + 1, zero-padded to three digits (`001`, `002`, etc.).

11. **Create the git worktree for this take**:
    - Derive the scene branch: `scene/<scene-name>`
    - Check if the scene branch exists: `git branch --list scene/<scene-name>`
    - If it does not exist, create it from current HEAD: `git branch scene/<scene-name>`
    - Derive the take branch: `take/<scene-name>-<NNN>`
    - Derive the worktree path: `.worktrees/take-<scene-name>-<NNN>` (relative to repo root)
    - Create the worktree: `git worktree add -b take/<scene-name>-<NNN> .worktrees/take-<scene-name>-<NNN> scene/<scene-name>`
    - If the take branch already exists (stale from a previous attempt): force-delete it (`git branch -D take/<scene-name>-<NNN>`), remove any stale worktree (`git worktree remove --force .worktrees/take-<scene-name>-<NNN>`), then retry. If the retry fails, stop and report the error — do not proceed until the worktree is clean.
    - Record the absolute worktree path.

---

## Take Loop

The loop runs for a maximum of **5 takes** total across all runs of this skill on this scene. Count from existing take files.

### Step 1 — Update manuscript status

Set the manuscript `**Status**` field to `in-progress` if it is not already.

### Step 2 — Dispatch actors

For each actor in the Cast table, spawn a sub-agent using the actor agent:

- Pass `--role <role-name> --backgrounds <bg1,bg2,...>` as arguments
- Include in the agent prompt:
  - The full manuscript objectives
  - The specific shot(s) assigned to this actor, verbatim from the manuscript
  - The absolute worktree path as the actor's working directory — all file reads and writes must occur within this path
  - The path to the scene directory within the worktree (`.claude/slated/scenes/scene-<name>/`) so the actor can read scene-relative files
  - An explicit constraint: actors must never write to `.claude/slated/` inside the worktree or in the main tree — framework artefacts are never produced in a worktree context
  - If a previous take exists, the path to the most recent take file in the main tree — instruct the actor to read it in full, paying particular attention to their own Director's Notes and the Next Take Instructions

Dispatch shots sequentially by default. A shot is independent only if none of its actions reference the output of another shot in this take as a prerequisite AND no two shots would modify the same files — check both conditions explicitly against the manuscript before deciding. If a shot is independent, it may be dispatched in parallel with others. If any dependency or file conflict exists, the dependent shot must wait.

Wait for all actors to complete before proceeding.

### Step 3 — Capture changed files

Run the following in the worktree:

```
git -C <worktree-path> status --short
```

Parse the output into a structured list of file paths with their status (A = added, M = modified, D = deleted). This is the canonical record — do not rely on actor output to infer the file list.

### Step 4 — Commission writer review

Spawn a writer actor:

- Pass `--role writer` as arguments
- Instruct the writer to:
  1. Read the manuscript
  2. Read all output produced by the actors in this take
  3. Assess whether each action in each shot was followed, skipped, or deviated from
  4. Identify any part of the manuscript that proved unclear, incorrect, or missing information that caused deviation
  5. Return a structured Writer's Notes report:

```
**Manuscript fidelity**:
- <action ref>: followed | deviated | skipped — <brief note if not followed>

**Manuscript issues**:
- <issue> → <proposed correction>
```

Wait for the writer to complete.

### Step 5 — Review output as director

Review all actor output against:

1. **Objectives** — is each objective in the manuscript met? Treat each as pass or fail.
2. **Role definitions** — did each actor operate within their role's behavioral parameters and constraints?
3. **Background conventions** — did each actor apply the conventions from their loaded backgrounds correctly?

For every failure found, record:
- The specific finding (what was wrong)
- The rule violated (with source — role file or background file)
- The location (file path, section, or line)
- The required change (exact, specific, actionable)

Review the full output before writing any notes. Do not flag issues mid-review.

### Step 6 — Determine take verdict

The take **passes** only if all of the following are true:
- All manuscript objectives are met
- No actor violated their role's constraints
- No actor materially violated background document conventions
- The writer found no significant manuscript fidelity failures

If any condition is not met, the take **fails**.

### Step 7 — Auto-resolve issues (if any)

If the Director's Notes from Step 5 contain cast role violations or background convention failures, resolve them before writing the take file.

For each affected cast role or background file, spawn a director actor:

- Pass `--role director` as arguments
- Include in the agent prompt:
  - The path to the cast role or background file (project-level location if applicable, otherwise global)
  - The specific findings for that file — the violation, the rule violated, and the required change
  - Instructions to:
    1. Read the current file in full
    2. Read the `## Refinement Log` section — if it does not exist, treat the log as empty
    3. Evaluate each proposed change against the log: if a proposed change **reverses a logged entry**, do not apply — return a conflict description instead
    4. If no conflicts: apply only the changes identified in the Director's Notes; append a new row to `## Refinement Log` recording today's date, the scene and take number, the finding, and a one-sentence summary of what changed; create the section if it does not exist
    5. Return a structured report: changes applied or skipped (with the conflict reason)

Multiple affected files may be dispatched in parallel — there are no cross-file dependencies.

Wait for all director actors to complete.

If any returned a conflict, use AskUserQuestion to surface the conflict and pause — do not proceed to Step 8 until the user resolves it.

**If the gap is in a crew role**: do not attempt to modify it. Surface it as a finding in the take file's Director's Notes and continue — crew role gaps are for the user to address outside the framework.

If manuscript issues were identified in Step 4, spawn a writer actor to revise:
- Pass `--role writer` as arguments
- Include the manuscript path, the issues verbatim, and instructions to revise only the affected sections — no user confirmation required

Wait for the writer to complete.

If no role, background, or manuscript issues were found, skip this step.

### Step 8 — Write the take file

Write the take file to the **main tree** — not the worktree. Path: `.claude/slated/scenes/scene-<name>/takes/take-<NNN>.md` (relative to the repo root).

Use the template at `${CLAUDE_SKILL_DIR}/templates/take-001.md`. Populate:

- **Files Changed** — the structured list from Step 3, grouped by status (Added, Modified, Deleted); this is the canonical record
- **Actor Summary** — factual account of what each actor completed and skipped
- **Director's Notes** — per-actor verdict with specific findings (leave empty if no findings for an actor); include any crew role gaps identified in Step 7 as findings
- **Writer's Notes** — the writer's fidelity report
- **Next Take Instructions** — if the take failed, consolidate all required changes into a prioritised, actor-attributed list; draw from Director's Notes, Writer's Notes, and any other outstanding issues — all sources must be translated into specific, actor-attributed instructions; omit this section if the take passed

### Step 9 — Resolve the worktree

**If the take passed:**
1. Merge into the scene branch: `git checkout scene/<scene-name> && git merge take/<scene-name>-<NNN> --no-ff -m "take(<scene-name>): merge take-<NNN>"`
2. Return to the original branch: `git checkout -`
3. Remove the worktree: `git worktree remove .worktrees/take-<scene-name>-<NNN>`
4. Delete the take branch: `git branch -d take/<scene-name>-<NNN>`
5. Leave the scene branch in place — it is the PR branch and is never auto-merged to main

**If the take failed:**
1. Force-remove the worktree: `git worktree remove --force .worktrees/take-<scene-name>-<NNN>`
2. Delete the take branch: `git branch -D take/<scene-name>-<NNN>`

In both cases, confirm the worktree is removed before proceeding. If any command fails, surface the error and stop.

### Step 10 — Determine next action

**If the take passed**: proceed to the Wrap process immediately. Do not return control to the user.

**If the take failed, single-take mode**: surface the verdict and stop:

```
Take <NNN> — FAIL

The following issues must be resolved before the next take:

<Next Take Instructions, numbered>

Run /slated:shoot <scene-name> to begin take <NNN+1>.
```

**If the take failed, loop mode, limit not reached**: surface a brief status update and begin the next iteration:

```
Take <NNN> — FAIL. Beginning take <NNN+1>.
```

Create a new worktree (Pre-work step 11), increment the take number, and return to Step 1.

**If the take limit (5) has been reached**: surface the blocked summary and stop:

```
Scene <name> — BLOCKED

The take limit (5) has been reached without a passing take.

Repeated failure reason(s):
<concise summary of issues that persisted across the most recent takes, drawn from Director's Notes>

Review the take files and manuscript, then either:
- Run /slated:shoot <scene-name> to attempt a single additional take after making manual corrections
- Run /slated:write <scene-name> to revise the manuscript before continuing
```

---

## Wrap Process

Executed inline after a passing take. No user confirmation required.

### Wrap Step 1 — Compile the Completion Record

Gather from the take files:

- **Completed date**: today's date (YYYY-MM-DD)
- **Takes required**: total count of take files
- **Take log**: each take's number and status; for failed takes, a brief primary failure reason drawn from Director's Notes
- **Role improvement notes**: patterns of repeated error across takes that suggest a gap in a cast role or background document; omit entirely if no meaningful patterns emerged
- **Manuscript improvement notes**: structural issues that caused actor confusion or deviation; omit entirely if none emerged

### Wrap Step 2 — Update the manuscript

Replace the HTML comment block and stub placeholder in `manuscript.md` with the completed Completion Record, following the template structure. Set the `**Status**` field to `complete`. The comment and stub must not remain in the file.

### Wrap Step 3 — Write wrap.md

Write `.claude/slated/scenes/scene-<name>/wrap.md` using the template at `${CLAUDE_SKILL_DIR}/templates/wrap.md`:

- **Description**: 2–4 sentences in plain language — what capability was added, what problem was solved, or what structure was established. No implementation detail. Written to be understood by an actor with no prior context on this scene.
- **Files**: a complete list of every file created or meaningfully modified during the scene, grouped by concern. Draw from the Files Changed sections of all take files — the `git status --short` output in each take is the authoritative record.
- **Editor Notes**: any shared or foundational artefacts this scene modified — entry points, shared handlers, configuration files, design system primitives — and any implicit dependencies introduced. If the scene only created new files with no effect on shared artefacts, omit this section.

### Wrap Step 4 — Open a pull request

Push the scene branch and open a PR:

1. Push: `git push -u origin scene/<scene-name>`
2. Create the PR: `gh pr create --title "scene(<scene-name>): complete" --body "$(cat <<'EOF'
<Description section from wrap.md, verbatim>

## Files
<Files section from wrap.md, verbatim>
EOF
)"`

If `gh` is not available or the push or PR creation fails, surface fallback instructions:

```
PR creation failed: <error>

Push and open the PR manually:
  git push -u origin scene/<scene-name>
  gh pr create --base main
```

### Wrap Step 5 — Report completion

Re-read the Completion Record from `manuscript.md` and check for Role Improvement Notes.

Surface a completion summary:

```
Scene <name> — COMPLETE

Completed in <N> take(s). wrap.md written.
PR: <url>
```

If Role Improvement Notes were present, append:

```
Role improvements identified:
- /slated:cast <role-name> — <one-line summary of improvement needed>
- /slated:brief <background-name> — <one-line summary of improvement needed>
```

Then use AskUserQuestion to pause: "Role improvements were identified during this scene. Review the notes above and confirm you have noted them before this scene is closed." Do not return until the user acknowledges.

---

## Output

- `.claude/slated/scenes/scene-<name>/takes/take-<NNN>.md` — take file per take executed
- `.claude/slated/scenes/scene-<name>/manuscript.md` — status updated; Completion Record appended on wrap
- Cast role and background files — updated by director actors if auto-resolved (Step 7); each update appends to the file's `## Refinement Log`
- `.claude/slated/scenes/scene-<name>/wrap.md` — written on pass
- PR opened from `scene/<scene-name>` into `main` (or fallback instructions surfaced)

---

## Rules

- Never run a scene whose status is not `confirmed` or `in-progress`
- Never dispatch an actor whose role file does not exist in any catalogue
- Never skip the writer review — manuscript fidelity is not optional
- Never skip auto-resolution (Step 7) when cast role or background violations are found — resolve before writing the take file
- Never allow a conflict reversal without AskUserQuestion
- Never modify crew roles — crew role gaps are surfaced in Director's Notes as findings for the user
- Always clean up the worktree (Step 9) before proceeding — never leave an orphaned worktree or stale take branch
- Never write the take file inside the worktree — always the main tree
- Never infer the changed file list from actor output — always derive from `git status --short` in the worktree
- Never merge a passing take into main — scene branch only
- Never stop on a pass before wrap — the wrap process runs immediately after a passing take, without returning control to the user
- Never wrap after a hard stop — the wrap process runs only after a passing take
- Never exceed 5 takes — hard stop at the limit; surface the blocked summary and return control to the user
- Never create a worktree directly from main — always create or reuse the scene branch first
- Never allow actors to write to `.claude/slated/` inside the worktree
- If a previous take exists, always pass the take file path to each actor — never run a subsequent take without the actor having access to their prior notes
