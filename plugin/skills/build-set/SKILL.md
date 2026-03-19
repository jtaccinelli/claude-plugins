---
name: build-set
description: Scaffold a new project from the set outline — building the complete directory structure, configuration files, entry points, and dependencies defined by the producer. Writes CLAUDE.md and the pilot on completion.
expected-role: set-designer
argument-hint: []
---

# Build Set

Scaffold the project from the set outline. The set-designer reads `.claude/slated/set/outline.md` in full, builds everything it specifies in dependency order, and writes no file that the outline does not define. The output is a complete, functional project scaffold ready for actors to begin writing scenes against.

---

## Pre-work

1. Read `.claude/slated/set/outline.md` in full — this is the sole source of truth for all scaffolding decisions; if it does not exist, stop and instruct the user to run `/slated:produce-project` first. If `.claude/slated/set/review.md` exists, delete it — the scaffold is being (re)built and the prior review is no longer valid; a fresh review will be required
2. Ensure the following project-level directories exist, creating any that are absent: `.claude/slated/set/`, `.claude/slated/scenes/`, `.claude/slated/roles/`, `.claude/slated/backgrounds/`. Do not modify existing files.
3. Read each background document listed in the outline's tech stack — check `.claude/slated/backgrounds/background-<name>.md` (project) first; if not found, use Bash (`ls $HOME/.claude/slated/backgrounds/background-<name>.md 2>/dev/null`) to check globally and read at `$HOME/.claude/slated/backgrounds/background-<name>.md` if present; use whichever is found first. Understand the conventions before touching any files.
4. Confirm the outline is complete — every section must be populated; if any section is missing or unclear, surface the gap and ask before proceeding

---

## Process

### Step 1 — Build in dependency order

Create all files in the order the outline specifies, respecting dependency constraints:

1. Root configuration files (`package.json`, `pnpm-workspace.yaml` if monorepo, `tsconfig.json`)
2. Runtime configuration (`wrangler.jsonc`)
3. CSS entry point and Tailwind configuration
4. Server entrypoint and load context declaration
5. Application entrypoint and route manifest
6. Any additional integration files (schema, drizzle config, session handler stubs)

Every file written must reflect the conventions in the relevant background document exactly. The outline defines what to build; the backgrounds define how to build it correctly. Do not deviate from either.

If a conflict exists between the outline and a background — a value in the outline that violates a background convention — surface the conflict before writing the affected file. Do not resolve it silently.

### Step 2 — Install dependencies

Run the appropriate install command for the project's package manager (e.g. `pnpm install`). Confirm the install completes without errors before proceeding. If it fails, diagnose from the output — do not proceed with a broken install.

### Step 3 — Write CLAUDE.md

Write the project's `CLAUDE.md` file. This file is loaded into every future actor's context — it must describe the project accurately and concisely so actors can orient themselves without re-reading the full scaffold.

Derive all content from the set outline and what was built:

1. **Project** — one sentence describing what this project is and what it does (from the outline)
2. **Tech stack** — the technologies in use (from the outline's tech stack section)
3. **Structure** — a brief description of the directory layout and what each top-level area contains (from what was built)
4. **Entry points** — the key files an actor needs to know about: server entrypoint, route manifest, CSS entry, schema, etc.
5. **Key decisions** — the foundational choices from the outline's Key Decisions section, stated plainly

Do not include framework meta-documentation. Do not duplicate what background documents already cover in detail — reference the pattern, not the recipe.

### Step 4 — Write the pilot

Write `.claude/slated/scenes/pilot.md`. This is a lightweight scene backlog — a starting point for the writer when planning the first scenes. It is not a manuscript and contains no cast, objectives, or actions.

Include:

- A one-sentence description of the project, restating what was established in the outline
- A suggested list of scenes: each with a short kebab-case name and a one-line description of what it would accomplish

Derive the scene suggestions from the scaffold itself — what routes exist, what the data model supports, what user flows are implied by the project's purpose. Aim for 3–6 scenes that together would make the project functional end-to-end.

Format:

```markdown
# Pilot

<one sentence describing the project>

## Suggested Scenes

- `scene-<name>` — <one-line description of what this scene would accomplish>
- `scene-<name>` — <one-line description>
```

This file is a starting point, not a plan. The writer will make their own casting and sequencing decisions when `/slated:write-scene` is run.

---

## Output

- A complete project scaffold, built from `.claude/slated/set/outline.md`
- No placeholder files — every file created must be complete enough to be functional
- `CLAUDE.md` — project context file written for future actors
- `.claude/slated/scenes/pilot.md` — suggested scene backlog for the writer

Report the full list of files created, grouped by concern, and note which background document each group was derived from. If any conflict arose between the outline and a background, document it so it can inform future updates to both.

---

## Rules

- Never build without a complete set outline — if `.claude/slated/set/outline.md` is missing or incomplete, stop
- Never introduce a file or pattern that the outline does not specify
- Never deviate from background conventions when building what the outline defines
- Never surface a conflict silently — if the outline and a background disagree, flag it before writing the affected file
- Never produce a partial scaffold — the output must be complete enough for scene work to begin
- Never skip writing `CLAUDE.md` — project context is a required output of every build
- Never skip writing `.claude/slated/scenes/pilot.md` — the scene backlog is a required output of every build
- Never make decisions the outline deferred — if the outline is ambiguous, ask rather than guess
