---
name: produce-project
description: Gate a new project for production readiness — interviewing to understand what is being built, verifying all required backgrounds and roles exist, and authoring the set outline. No scaffolding begins until all required backgrounds and roles exist and the set outline is confirmed.
expected-role: producer
argument-hint: [brief description of the project]
---

# Produce Project

Gate a new project for production readiness. The producer interviews the user to understand what is being built, maps every required technology to an available background document, verifies all required roles exist, and authors a set outline for the set-designer to build from. The producer does not scaffold — they define what must be built and confirm everything needed to build it correctly is in place.

---

## Pre-work

1. Glob both `~/.claude/slated/backgrounds/background-*.md` (global) and `.claude/slated/backgrounds/background-*.md` (project) — read each in full. Merge the two sets, with project-level files taking precedence over global ones with the same name. These are the only source of truth for scaffold decisions; know what is available before the interview begins.
2. Glob both `~/.claude/slated/roles/role-*.md` (global) and `.claude/slated/roles/role-*.md` (project) — merge, with project-level files taking precedence. Note which roles are defined.
3. Use `$ARGUMENTS` as a starting point if provided

---

## Process

### Step 1 — Establish what is being built

Interview the user to understand the nature of the project. Ask:

1. What is this project? What is its primary purpose and who uses it?
2. Is this a micro-app (single-purpose, few routes, minimal state) or a full application (multiple domains, complex data, multi-user)?
3. Is this a standalone app or part of a monorepo?
4. What integrations are required — database, authentication, CMS, third-party APIs?
5. Are there any known constraints — runtime, deployment target, existing tooling — that must be respected?

Do not proceed to verification until the project's nature and scope are clear.

### Step 2 — Scaffold project directories

Ensure the following project-level directories exist, creating any that are absent:

- `.claude/slated/set/` — for the set outline and set review
- `.claude/slated/scenes/` — for manuscripts, takes, and wrap records
- `.claude/slated/roles/` — for project-specific role overrides
- `.claude/slated/backgrounds/` — for project-specific background overrides

Do not modify or delete any files inside directories that already exist.

### Step 3 — Verify backgrounds

From the interview, compile a complete list of every technology, runtime, and integration the project requires. For each, identify whether a background document exists that covers it.

Present the mapping to the user in this format:

```
Required technologies:
- <technology> → background-<name>.md ✓
- <technology> → background-<name>.md ✓
- <technology> → NO BACKGROUND DEFINED ✗
```

**If any technology has no corresponding background**: stop here. Surface the gaps clearly:

```
The following technologies require background documents before production can proceed:
- <technology>

Run /establish-background for each before continuing.
```

Do not proceed until every required technology has a background document.

### Step 4 — Verify crew roles

From the production workflow, compile the crew roles required. Crew roles govern the production itself and must be present before any work begins. At minimum: `writer`, `director`, `visualiser`, `casting-director`, `set-designer`, `producer`. Check which are defined.

Cast roles — `design-engineer`, `systems-engineer`, or any other scene-delivery role — are not verified here. They are checked at scene-planning time against the manuscript cast when `/write-scene` is run.

Present the crew role audit:

```
Required crew roles:
- writer → role-writer.md ✓
- set-designer → NO ROLE DEFINED ✗
```

**If any crew role is missing**: stop here and surface the gaps:

```
The following crew roles must be defined before production can proceed:
- <role>

Run /cast-role for each before continuing.
```

Do not proceed until every required crew role is defined.

### Step 5 — Author the set outline

Read each required background document in full. From the conventions documented there and the project understanding established in Step 1, derive and write `.claude/slated/set/outline.md`.

The outline must include:

1. **Project** — a one-sentence description of what is being built and who it is for
2. **Tech stack** — the confirmed technologies, each mapped to its background document
3. **Directory structure** — the full directory tree the project requires, based on the Structure sections of each background
4. **Configuration files** — every config file required (`wrangler.jsonc`, `tsconfig.json`, `package.json`, etc.) with the values to use, drawn from background conventions
5. **Entry points** — the key files the project needs (server entrypoint, route manifest, CSS entry, schema, etc.)
6. **Dependencies** — packages required, derived from what the backgrounds reference
7. **Bindings and integrations** — runtime bindings (D1, KV, R2) and external integrations, derived from the relevant backgrounds and the decisions made in Step 1
8. **Key decisions** — any foundational choices made during the interview (storage approach, auth pattern, monorepo vs standalone, micro-app vs full app) that the set-designer must know before building

Do not introduce any pattern, value, or configuration that cannot be traced back to a background document. If a decision cannot be grounded in an available background, flag it to the user and ask rather than guessing.

Present the full outline to the user before writing the file. Ask for confirmation or adjustments. Do not write the file until confirmed.

---

## Output

- `.claude/slated/set/outline.md` — the complete set outline, ready for the set-designer to build from

Report the file path and confirm that all required backgrounds and roles are in place. If any decisions were made where backgrounds were ambiguous, document them in the Key Decisions section of the outline so they can inform future background updates.

---

## Rules

- Never scaffold a technology without a corresponding background document — stop and flag the gap instead
- Never proceed if any required role is missing — flag and stop
- Never write the set outline before the user confirms it
- Never introduce a pattern that cannot be traced to a background document
- Never build anything — the producer defines what must be built; the set-designer builds it
- Never make foundational decisions (storage approach, auth pattern, package structure) without surfacing them to the user for confirmation
