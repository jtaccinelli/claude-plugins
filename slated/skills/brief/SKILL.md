---
name: brief
description: Create a new background or refine an existing one — through structured interview or from a summary, with all changes logged in the background file itself.
expected-role: director
argument-hint: [background-name | background-summary]
---

# Brief

Establish or sharpen a background. A successful run leaves the background file covering Semantics, Structure, and Function — complete, confirmed, and — if changes were made to an existing background — appended with a new refinement log entry.

---

## Pre-work

1. Run `/slated:perform director` to adopt the director role.

2. Resolve the home directory: run `echo $HOME`.

3. Discover all existing backgrounds:
   - Project-level: Glob `.claude/slated/backgrounds/background-*.md`
   - Global: run `ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null`
   - Merge both, with project-level taking precedence on name collision. Read each in full to understand what is already documented and where its edges are.

4. Parse `$ARGUMENTS`:
   - If `$ARGUMENTS` matches an existing background name (after stripping `background-` prefix and `.md` extension from the catalogue) → proceed to the **Refinement path**
   - If `$ARGUMENTS` is provided but does not match an existing background name → treat it as a summary and proceed to the **Summary draft path**
   - If `$ARGUMENTS` is empty → proceed to the **Interactive path**

---

## Process — Interactive Path

### Step 1 — Determine intent

Use AskUserQuestion: "Are you creating a new background or refining an existing one?"

- If **refining**: use AskUserQuestion to list all existing background names and ask which one. Then proceed to the Refinement path.
- If **creating**: use AskUserQuestion: "Do you want to be interviewed, or would you like to give me a summary and I'll draft it from that?"
  - If **interview**: proceed to the Interview draft path
  - If **summary**: ask for the summary, then proceed to the Summary draft path

---

## Process — Refinement Path

### Step 1 — Establish what needs to change

Read the background file in full, including its `## Refinement Log` section if present.

Use `$ARGUMENTS` as a starting point if a reason was provided. Ask the user:

1. What is not working about the current background? What actor behaviour or output has revealed the gap?
2. Which sections are affected — Semantics, Structure, or Function?
3. Is this a correction (something wrong), an addition (something missing), or a removal (something that does not belong)?

Do not proceed until the scope is clear. If the answers suggest the background needs fundamental rebuilding rather than targeted adjustment, surface that and ask whether a full redefinition would be more appropriate.

### Step 2 — Check for refinement log conflicts

Read the `## Refinement Log` at the bottom of the background file. If no such section exists, treat the log as empty.

Before proposing any change, evaluate it against the log: if a proposed change **reverses a previously logged entry**, do not proceed — surface the conflict to the user and stop until resolved:

```
Conflict detected: the proposed change to <section> reverses a logged refinement from <date>.
Prior entry: <summary of logged change>
Proposed change: <what would be reversed>

How would you like to resolve this?
```

### Step 3 — Propose targeted changes

For each section identified as needing change, present the specific proposed edit:

- Show the current text and the proposed replacement side by side
- State the reason for each change in one sentence
- Do not rewrite sections that are not in scope

If a proposed change in one section conflicts with another (e.g. a new Function pattern that contradicts an existing Semantics convention), surface the conflict and resolve it before confirming.

### Step 4 — Confirm and write

Present all proposed changes together and ask for explicit confirmation. If adjustments are requested, revise and present again.

Apply only the confirmed changes to the background file at its existing location. Append a new row to the `## Refinement Log` table:

| Date | Scene / Context | Finding | Change Summary |
|---|---|---|---|
| <YYYY-MM-DD> | <scene name or "direct refinement"> | <the gap that prompted the change> | <one sentence describing what was changed> |

If the `## Refinement Log` section does not yet exist in the file, add it at the bottom with the table header before appending the first row.

---

## Process — Interview Draft Path

### Step 1 — Establish the domain

Ask the user:

1. What technology, tool, or domain does this background cover?
2. What kind of actor would load this background — what role would benefit from knowing it?
3. What makes this domain distinct enough to warrant its own background rather than an addition to an existing one?

If the user's answers reveal substantial overlap with an existing background, name the overlap and ask whether extending the existing background is preferable. Do not proceed until the domain is confirmed as genuinely distinct.

### Step 2 — Draft and confirm

Using the understanding from Step 1, draft the full background document in a single pass. Produce all sections:

1. **Opening statement** — one sentence describing what domain the background covers and why it matters to an actor filling a relevant role; specific enough that an actor immediately understands what they are about to absorb and why it is relevant
2. **Semantics** — naming conventions, terminology, typing idioms, and syntax choices. Every entry must tell an actor exactly what to write — vague guidance ("use descriptive names") is not acceptable; produce concrete patterns
3. **Structure** — file and directory layout, where things live, import paths, and how the domain organises its artefacts. Include concrete directory trees and import path examples where they add clarity; the structure must be reproducible from this section alone
4. **Function** — runtime behaviour, patterns for achieving outcomes, and recipes for common operations. Include concrete code examples where they make a convention unambiguous; every pattern must be complete enough to implement from

Present the complete draft. If the user requests changes, revise and re-present.

### Step 3 — Confirm destination and write

Derive a kebab-case name from the domain (e.g. "Tailwind CSS" → `tailwind`, "React Router 7" → `react-router`). Use AskUserQuestion: "Should this background be global (available across all projects) or project-specific?"

- **Global** → `$HOME/.claude/slated/backgrounds/background-<name>.md`
- **Project-specific** → `.claude/slated/backgrounds/background-<name>.md`

Write the file to the confirmed location. Every section must be fully populated.

---

## Process — Summary Draft Path

Draft the complete background from the provided summary without interview, following the same section structure as the Interview draft path. Present the draft to the user, then confirm destination and write as in Interview draft Step 3.

---

## Output

- Background file written at the confirmed location (new background) or updated in place (refinement)
- Refinement log appended if changes were made to an existing background

---

## Rules

- Never apply a change that reverses a logged refinement without surfacing the conflict to the user first
- Never write a partial draft — all sections must be complete before writing
- Never write vague conventions — every entry must be specific enough that an actor would know exactly what to write
- Never include domain knowledge that belongs in a role definition — backgrounds document what to know, not how to behave
- Never extend an existing background when a new one is warranted, and never create a new one when an existing one could be extended — surface the overlap and ask first
- Never write the file before confirmed — for refinements: all proposed changes confirmed; for new backgrounds: destination confirmed
- Never accept vague change requests — if the reason for a change is unclear, ask until it is specific
