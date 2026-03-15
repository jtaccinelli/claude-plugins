---
name: write-scene
description: Draft a new scene from a requirement or idea. Asks clarifying questions, produces a manuscript.md, creates the scene folder structure, and registers it in the storyboard. Use whenever a new feature or fix needs to be planned before actors run it.
expected-role: writer
argument-hint: [brief description of the feature or fix]
---

# Write Scene

Translate a requirement into a complete, confirmed `manuscript.md` and register the new scene in the storyboard. Nothing is written until the requirement is understood and the cast is validated.

---

## Pre-work

Read the available roles and backgrounds so casting decisions are grounded in what actually exists:

- Glob both `~/.claude/slated/roles/role-*.md` (global) and `.claude/slated/roles/role-*.md` (project) — merge, with project-level taking precedence; read each to understand available roles and their expertise
- Glob both `~/.claude/slated/backgrounds/background-*.md` (global) and `.claude/slated/backgrounds/background-*.md` (project) — merge, with project-level taking precedence; read each in full; background content determines relevance, not just name
- Read `.claude/slated/scenes/storyboard.md` — understand what scenes already exist to avoid duplication

---

## Process

### Step 1 — Establish the requirement

Use `$ARGUMENTS` as a starting point. If it is sufficient to understand the scope, proceed. If it is vague, ask focused questions to establish:

1. **What is being built or fixed?** — One clear sentence.
2. **What type of change is this?** — New feature, bug fix, refactor, infrastructure, content, or other.
3. **What does success look like?** — What will be true when this scene is complete that is not true now? Push for verifiable outcomes, not intentions.
4. **Are there constraints or edge cases to know about up front?** — Anything that would affect how the work is sequenced or who does it.

Do not proceed to casting until these are clear.

### Step 2 — Name the scene

Derive a short, descriptive kebab-case name from the requirement. This becomes the folder name: `scene-<name>`.

Present the name to the user for confirmation before creating anything.

### Step 3 — Cast the scene

Based on the requirement, determine:

1. **Which roles are needed?** — Match the work to roles whose expertise and constraints fit. Do not assign work to a role that its constraints prohibit.
2. **Which backgrounds should each actor load?** — Select from the available backgrounds. Only include backgrounds directly relevant to the work the actor will do.
3. **What external resources are required?** — APIs, services, databases, third-party platforms.

Present the proposed cast to the user. If a required role or background does not exist, stop immediately — do not invent one and do not continue planning. No files have been written at this point, so there is nothing to clean up. Surface the gap to the user:

```
Cannot proceed — the following are required but do not exist:

- Role: <name> → run /cast-role to define it
- Background: <name> → run /establish-background to define it

Once created, run /write-scene again to restart planning.
```

Do not proceed to Step 4 until the cast is fully satisfied by existing roles and backgrounds.

### Step 4 — Define objectives

Write 3–6 verifiable objectives. Each must describe a concrete outcome that the director can confirm or deny after a take. Avoid vague language like "the feature works" or "the UI is complete".

Good objective: `User can submit a form at /items/create and see the new item appear in the list`
Bad objective: `Form submission is implemented`

### Step 5 — Sequence the actions

Break the work into ordered shots, each assigned to a specific actor. Within each shot, list atomic actions — what the actor must produce, not how they produce it.

Rules for actions:
- Each action has a clear, observable output
- Actions within a shot are sequential; shots themselves may be sequential or parallel
- Do not describe implementation methods — describe outcomes
- If a shot depends on output from a previous shot, say so explicitly

### Step 6 — Draft and confirm

Present the full manuscript to the user before writing anything:

- Scene name
- Cast table
- Resources
- Objectives
- Actions (shots and steps)
- Any notes worth preserving

Ask for confirmation or adjustments. Do not write files until confirmed.

### Step 7 — Write the scene files

Once confirmed, create the scene directory and files:

1. Create `.claude/slated/scenes/scene-<name>/` directory
2. Create `.claude/slated/scenes/scene-<name>/takes/` directory
3. Write `.claude/slated/scenes/scene-<name>/manuscript.md` with status set to `confirmed`

Use the manuscript template structure, including the HTML comment block and stub placeholder for the Completion Record — do not populate the record data. The director replaces the comment and stub with the completed Completion Record during finalisation.

### Step 8 — Dispatch the visualiser

Spawn a sub-agent using the actor agent:

- Pass `--role visualiser` as arguments
- Include in the agent prompt:
  - The path to the completed `manuscript.md`
  - The path to `.claude/slated/scenes/storyboard.md`
  - The scene directory path
  - Instructions to:
    1. Read the manuscript to extract the scene name, status, and one-line requirement
    2. Read the current storyboard in full
    3. Add the new scene to the Pending section of `storyboard.md`:
       ```
       | `scene-<name>` | confirmed | <one-line requirement> |
       ```

Wait for the visualiser to complete before proceeding.

---

## Output

- `.claude/slated/scenes/scene-<name>/manuscript.md` — complete manuscript at `confirmed` status
- `.claude/slated/scenes/scene-<name>/takes/` — empty directory ready for take files
- `.claude/slated/scenes/storyboard.md` — updated Pending section

Report the scene folder path, the number of actors cast, and the number of objectives defined.

---

## Rules

- Never write files before the user confirms the manuscript
- Never cast a role that does not exist in the merged role catalogue — flag the gap instead
- Never assign an action to a role whose constraints prohibit it
- Never write objectives that cannot be verified by observation
- Never populate the Completion Record data in a newly written manuscript — include the HTML comment block and stub as placeholders only
