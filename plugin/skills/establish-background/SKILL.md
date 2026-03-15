---
name: establish-background
description: Define a new background document through a structured interview — covering semantics, structure, and function for a specific technology or domain. Nothing is written until every section is confirmed.
expected-role: casting-director
argument-hint: [technology or domain to document]
---

# Establish Background

Define a new background document from scratch through a structured interview. A background is domain knowledge — the conventions, patterns, and rules an actor needs to operate effectively in a given technology or domain. Every section is established through dialogue before anything is committed to file. The output is a complete, immediately usable background document.

---

## Pre-work

1. Use Bash (`ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null`) to list global backgrounds, and Glob `.claude/slated/backgrounds/background-*.md` for project-level ones — merge both results, with project-level taking precedence; read each in full to understand what is already documented and where its edges are
2. If `$ARGUMENTS` describes a domain that an existing background already covers, surface this to the user — ask whether extending the existing background is preferable to creating a new one

Only proceed with a new background once the domain is confirmed as genuinely uncovered or sufficiently distinct.

---

## Process

### Step 1 — Establish the domain

Use `$ARGUMENTS` as a starting point. Ask the user:

1. What technology, tool, or domain does this background cover?
2. What kind of actor would load this background — what role would benefit from knowing it?
3. What makes this domain distinct enough to warrant its own background rather than an addition to an existing one?

Do not proceed until the domain is clear and confirmed as distinct.

### Step 2 — Opening statement

Establish the one-sentence opening that appears at the top of the background — a description of what domain the background covers and why it matters to an actor filling a relevant role.

Present it for confirmation before proceeding. It should be specific enough that an actor reading it immediately understands what they are about to absorb and why it is relevant to their work.

### Step 3 — Semantics

The Semantics section covers naming conventions, terminology, typing idioms, and syntax choices specific to this domain. It answers: what are things called, how are they typed, and what does correct syntax look like?

Interview the user through each relevant area:

1. What are the core naming conventions — files, variables, types, functions?
2. Are there typing patterns or idioms specific to this domain? What does a correctly typed artefact look like?
3. Are there syntax choices that are correct versus incorrect — things an actor might get wrong if they didn't know the convention?
4. Are there terms or concepts that have specific meanings in this domain that differ from general usage?

For each area, ask follow-up questions until the answer is specific enough to be actionable. Vague guidance ("use descriptive names") is not acceptable — push for the concrete pattern.

Draft the Semantics section and present it for review. Revise until every entry would tell an actor exactly what to write, not just what to think about.

### Step 4 — Structure

The Structure section covers file and directory layout, where things live, import paths, and how the domain organises its artefacts. It answers: where does this code go, and how does it relate to other files?

Interview the user:

1. What is the directory structure for this domain? Where do files live within the project?
2. What are the import conventions — aliased paths, relative paths, barrel files?
3. Are there layers or separation-of-concerns conventions specific to this domain (e.g. fragments vs queries, schema vs handlers)?
4. What is the relationship between files in this domain — what depends on what?

Include concrete directory trees and import path examples where they add clarity. Present the section for review and revise until the structure could be reproduced from this document alone.

### Step 5 — Function

The Function section covers runtime behaviour, patterns for achieving outcomes, and recipes for common operations. It answers: how does this domain behave when running, and how do you make it do things correctly?

Interview the user:

1. What are the most common operations an actor will need to perform in this domain?
2. What patterns or recipes cover those operations — what does correct implementation look like?
3. Are there runtime constraints, gotchas, or failure modes an actor must know about?
4. Are there architectural decisions — things that must be done a certain way for the system to work correctly — that an actor needs to understand before writing any code?

For each pattern identified, ask for a concrete code example if one would make the convention unambiguous. Include these in the section. Present the section for review and revise until every pattern is complete enough to implement from.

### Step 6 — Present the full background

Assemble and present the complete background document — all sections formatted exactly as they will appear in the file. Ask the user to confirm the document is correct and complete, or to request changes to any section.

Do not write the file until explicit confirmation is given. If changes are requested, revise and present again.

### Step 7 — Confirm the destination

Derive a kebab-case name from the domain (e.g. "Tailwind CSS" → `tailwind`, "React Router 7" → `react-router`).

Ask the user whether this background should be:

1. **Global** — reusable across all projects on this device → writes to `~/.claude/slated/backgrounds/background-<name>.md`
2. **Project-specific** — only available in this project → writes to `.claude/slated/backgrounds/background-<name>.md`

Present the full file path for confirmation before writing.

### Step 8 — Write the background file

Write the background file to the confirmed location. Every section must be fully populated — no placeholder text, no empty fields.

---

## Output

- `~/.claude/slated/backgrounds/background-<name>.md` or `.claude/slated/backgrounds/background-<name>.md` — complete, immediately usable background document at the confirmed location

Report the file path.

---

## Rules

- Never write the file before the user explicitly confirms the full document
- Never leave a section unpopulated — if information is missing, return to the interview
- Never extend an existing background when a new one is warranted, and never create a new one when an existing one could be extended — surface the overlap and ask first
- Never accept vague conventions — every entry must be specific enough that an actor would know exactly what to write
- Never skip a section of the interview — Semantics, Structure, and Function must all be established through dialogue, not assumed
- Never include domain knowledge that belongs in a role definition — backgrounds document what to know, not how to behave
- Never write without confirming the destination (global vs project) first
