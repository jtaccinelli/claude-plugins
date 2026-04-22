---
name: scout
description: Author or update the project's shot locations document — scanning the codebase to map every meaningful structural entry point.
expected-role: dop
argument-hint: (none)
---

# Scout

Map the project. A successful run leaves `.claude/slated/locations.md` accurate and confirmed — either freshly authored from a codebase scan, or updated to reflect structural changes since the last run.

---

## Pre-work

1. Run `/slated:perform dop` to adopt the DOP role.
2. Check whether `.claude/slated/locations.md` exists.
   - If it does **not** exist: this is an **initial scan** — proceed to the Initial Scan process.
   - If it **does** exist: this is an **update scan** — proceed to the Update Scan process.

---

## Process — Initial Scan

### Step 1 — Walk the project structure

Scan the full project directory tree. For each top-level directory and every named file that represents a meaningful entry point or domain boundary, record the path and assess what it contains.

A location is any file or directory that:
- Is a named entry point (server, routes, CSS, schema, config)
- Groups domain logic or a distinct concern under a single directory
- Is a configuration file an actor would need to read or modify
- Is a key integration file that scenes will build on or reference

Exclude:
- `node_modules/`
- Build output directories (`.next/`, `dist/`, `.output/`, `build/`)
- Lock files (`package-lock.json`, `pnpm-lock.yaml`, `yarn.lock`, `bun.lockb`)
- Generated type files (`.d.ts` files in output directories)
- Cache directories (`.cache/`, `.turbo/`, `.wrangler/`)
- Files inside `.claude/` — framework artefacts are never listed as shot locations

### Step 2 — Draft the locations document

Group the identified entries by concern (e.g. "Application", "Configuration", "Database", "CMS", "Tooling"). For each entry, write:

- **Name**: kebab-case, unique (e.g. `route-manifest`, `db-schema`, `server-entry`)
- **Path**: relative to the project root — never absolute
- **Description**: one sentence stating what the file or directory contains and when an actor should reference it

Format:

```markdown
# Shot Locations

Named file and directory locations for manuscript authors. Reference these when specifying actor actions to give actors an unambiguous starting point.

| Name | Path | Description |
|---|---|---|
| `<kebab-case-name>` | `<path/relative/to/project/root>` | <what this file or directory contains and when to reference it> |
```

### Step 3 — Confirm and write

Use AskUserQuestion to surface the full draft to the user: "Here is the proposed locations list. Does this look correct?"

Do not write the file until the user confirms. If the user requests changes, apply them and confirm again before writing.

Write `.claude/slated/locations.md` once confirmed.

---

## Process — Update Scan

### Step 1 — Read the existing document

Read `.claude/slated/locations.md` in full. Record every existing entry — name, path, and description.

### Step 2 — Walk the current project structure

Scan the project directory tree using the same rules as the Initial Scan process. Record every location-worthy entry currently present in the project.

### Step 3 — Produce a diff

Compare the current structure against the existing document. Identify:

- **Entries to add**: meaningful locations present in the project but not in the document
- **Entries to remove**: entries in the document that no longer correspond to a location in the project
- **Entries to rename or update**: entries where the path has changed or the description is no longer accurate

Present the diff to the user in this format:

```
Proposed changes to locations.md:

Add:
- `<name>` | `<path>` | <description>

Remove:
- `<name>` | `<path>` | <previous description>

Update:
- `<name>`: path `<old>` → `<new>`
```

If there are no changes, report: "The locations document is up to date — no changes required." and stop.

### Step 4 — Confirm and apply

Use AskUserQuestion: "Do these changes look correct?" Do not write until confirmed.

Apply all confirmed changes to the existing document and write the updated file.

---

## Output

- `.claude/slated/locations.md` — written (initial scan) or updated (update scan)

---

## Rules

- Never write the document without user confirmation — the locations list is a user-confirmed artefact
- Never add an entry without verifying its existence in the actual project structure
- Never remove an entry without explicit user confirmation
- Never include files inside `.claude/` — framework artefacts are not project locations
- Never include implementation noise: `node_modules/`, build output, lock files, generated types, cache directories
- Never use absolute paths — all paths must be relative to the project root
- Never add entries speculatively — only map what currently exists
- Names must be unique and kebab-case across the entire document
