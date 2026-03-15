---
name: cast-role
description: Define a new role through a structured interview — covering identity, background, expertise, behavioral parameters, constraints, and interactions. Nothing is written until every section is confirmed.
expected-role: casting-director
argument-hint: [brief description of the role needed]
---

# Cast Role

Define a new role from scratch through a structured interview. Every section of the role definition is established through dialogue before anything is committed to file. The output is a complete, production-ready role definition — no placeholders, no assumptions left by the casting director.

---

## Pre-work

1. Glob both `~/.claude/slated/roles/role-*.md` (global) and `.claude/slated/roles/role-*.md` (project) — merge, with project-level taking precedence; read each in full to understand what roles already exist, what they cover, and where their edges are
2. If `$ARGUMENTS` describes a need that an existing role could plausibly cover, surface this to the user before proceeding — ask whether refining the existing role via `/refine-role` is preferable to defining a new one

Only proceed with a new definition once the user has confirmed the need is genuinely distinct from existing roles.

---

## Process

### Step 1 — Establish the need

Use `$ARGUMENTS` as a starting point. Ask the user:

1. What is this role responsible for? What problem does it solve in the production?
2. What kind of work will actors in this role be doing?
3. What makes this role distinct from any existing role in the catalogue?

Do not proceed until the need is clearly distinct from what already exists. If the user's answers reveal overlap with an existing role, name the overlap and ask directly whether a new role is still warranted.

### Step 2 — Identity

Establish the three identity fields one at a time:

1. **Title** — what is this role's job title or character label?
2. **Domain** — in one or two words, what is their primary area of responsibility?
3. **Summary** — in one sentence, who is this person and what do they do?

Present the proposed identity back to the user and confirm before moving on. If the summary is vague or reads as a job posting rather than a character description, push for revision.

### Step 3 — Background narrative

The background is a 2–3 paragraph professional history that shapes how this role thinks. It is not a skills list — it is a narrative that explains the role's perspective, instincts, and accumulated experience. Ask:

1. What career arc or formative experience shaped this role's instincts? What have they built, shipped, or broken that defines how they think?
2. What is the hard-won lesson at the core of their discipline — what have they been burned by that others haven't?
3. What do they notice or prioritise that a generalist would miss?

Draft the narrative from the answers and present it for review. Do not move on until it reads as a convincing, lived-in character rather than a job description. Revise as many times as needed.

### Step 4 — Expertise

Establish a focused list of specific domains, capabilities, or areas this role owns. Ask:

1. What specific technologies, tools, systems, or methodologies does this role work with?
2. What patterns or artefacts does this role own that no other role in the catalogue produces?
3. What would an actor in this role be expected to do that an actor in any other role could not?

Present the proposed list and push back on anything too generic. "Problem-solving" and "communication" are not expertise items. Every item must be specific enough that it would not appear on another role's list. Revise until this is true of every entry.

### Step 5 — Behavioral Parameters

Establish each of the five parameters through direct questions. Do not move to the next until the current one is specific enough to meaningfully constrain actor behaviour.

1. **Tone** — how does this role sound when it communicates? What adjectives describe its register?
2. **Verbosity** — concise or thorough by default? Does this shift depending on context, and if so, how?
3. **Decision style** — how does this role reason through problems? First principles, pattern-matching, data, consensus — what is the default mode?
4. **Priorities** — what does this role optimise for? List in priority order — what wins when there is a trade-off?
5. **Working style** — how does this role approach ambiguity, collaboration, and blockers? What is its default posture when something is unclear or missing?

Present all five together for confirmation. If any reads as generic enough to describe any capable person, return to it and sharpen.

### Step 6 — Constraints

Establish the hard limits on what this role does and does not do. These are rules, not preferences. Ask:

1. What will this role never do, regardless of what it is asked? What is explicitly outside its remit?
2. Where is the line it does not cross before escalating or handing off?
3. Are there specific shortcuts, anti-patterns, or tempting deviations this role must never take?

For each proposed constraint, test it: would an actor know exactly when to stop based on this rule alone? If the answer is "maybe", it is not specific enough. Push for precision on every item.

For **cast roles**: scope boundaries must be stated in terms of the work itself, not in terms of which other role owns it. For example: "does not write server-side logic" is correct; "does not write server-side logic — that belongs to the systems engineer" is not. Cast roles do not know who their co-stars will be until they read the manuscript.

For **crew roles**: constraints may reference other roles by name. Crew operate within the meta-narrative with full production awareness — they govern, review, and build the production structure itself, and need to reason about other roles explicitly.

Present the constraints list for confirmation. For cast roles, revise until every entry is unambiguous and free of peer role references.

### Step 7 — Interactions

Establish how this role fits into the production workflow. Ask:

1. What does this role typically receive as input, and in what form?
2. What does this role produce as output — what file, report, or artefact?
3. At what point in the scene lifecycle does this role appear?

For **cast roles**: the interactions section describes the role's position in abstract terms. It does not name specific co-star roles — a cast role does not know who it will be working alongside until it reads the manuscript. Describe handoffs in terms of what is passed ("the completed data layer", "the server-side handlers"), not who passes it.

For **crew roles**: other roles may be named directly. Crew operate at the meta-narrative level with full awareness of the production structure and routinely need to reason about specific roles — who directs, who reviews, who builds.

Present the interactions summary and confirm. For cast roles, revise any language that names a specific peer role.

### Step 8 — Present the full definition

Assemble and present the complete role definition — all sections formatted exactly as they will appear in the file. Ask the user to confirm the definition is correct and complete, or to request changes to any section.

Do not write the file until explicit confirmation is given. If changes are requested, revise and present again.

### Step 9 — Confirm the destination

Derive a kebab-case name from the role title (e.g. "Design Engineer" → `design-engineer`).

Ask the user whether this role should be:

1. **Global** — available across all projects on this device → writes to `~/.claude/slated/roles/role-<name>.md`
2. **Project-specific** — only available in this project → writes to `.claude/slated/roles/role-<name>.md`

Present the full file path for confirmation before writing.

### Step 10 — Write the role file

Write the role file to the confirmed location. Every section must be fully populated — no placeholder text, no empty fields.

---

## Output

- `~/.claude/slated/roles/role-<name>.md` or `.claude/slated/roles/role-<name>.md` — complete, production-ready role definition at the confirmed location

Report the file path. If the role's effective deployment will require one or more background documents that do not yet exist, name them specifically so they can be defined before any scene casts this role.

---

## Rules

- Never write the file before the user explicitly confirms the full definition
- Never leave a section unpopulated — if information is missing, return to the interview before proceeding
- Never embed domain knowledge in the role definition — behavioral parameters only; knowledge belongs in backgrounds
- Never create a new role if an existing one can be adjusted — surface the overlap and ask first
- Never accept vague constraints — every constraint must name exactly what the actor will not do
- Never accept generic expertise items — every item must be specific enough that it would not appear on another role's list
- Never skip a section of the interview — every section of the role template must be established through dialogue, not assumed
- Never write without confirming the destination (global vs project) first
- For cast roles: never name a specific peer role in a constraints or interactions section — cast roles discover co-stars from the manuscript at execution time
- For crew roles: peer role references are acceptable in constraints and interactions — crew operate within the meta-narrative with full production awareness
