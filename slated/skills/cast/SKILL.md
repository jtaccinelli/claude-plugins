---
name: cast
description: Create a new cast role or refine an existing one — through structured interview or from a summary, with all changes logged in the role file itself.
expected-role: director
argument-hint: [role-name | role-summary]
---

# Cast

Define or sharpen a cast role. A successful run leaves the role file complete, accurate, and — if changes were made to an existing role — appended with a new refinement log entry. The director applies the same adversarial lens to role quality that they apply to actor output: a role that cannot direct good behaviour is a defect, not a draft.

---

## Pre-work

1. Run `/slated:perform director` to adopt the director role.

2. Resolve the home directory: run `echo $HOME`.

3. Discover all existing cast roles:
   - Project-level: Glob `.claude/slated/roles/role-*.md`
   - Global: run `ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null`
   - Merge both, with project-level taking precedence on name collision. Read each in full to understand what roles already exist, what they cover, and where their edges are.

4. Parse `$ARGUMENTS`:
   - If `$ARGUMENTS` matches an existing role name (after stripping `role-` prefix and `.md` extension from the catalogue) → proceed to the **Refinement path**
   - If `$ARGUMENTS` is provided but does not match an existing role name → treat it as a summary and proceed to the **Summary draft path**
   - If `$ARGUMENTS` is empty → proceed to the **Interactive path**

---

## Process — Interactive Path

### Step 1 — Determine intent

Use AskUserQuestion: "Are you creating a new role or refining an existing one?"

- If **refining**: use AskUserQuestion to list all existing cast role names and ask which one. Then proceed to the Refinement path.
- If **creating**: use AskUserQuestion: "Do you want to be interviewed, or would you like to give me a summary and I'll draft it from that?"
  - If **interview**: proceed to the Interview draft path
  - If **summary**: ask for the summary, then proceed to the Summary draft path

---

## Process — Refinement Path

### Step 1 — Establish what needs to change

Read the role file in full, including its `## Refinement Log` section if present.

Use `$ARGUMENTS` as a starting point if a reason was provided. Ask the user:

1. What is not working about the current role definition? What behaviour or output has revealed the gap?
2. Which sections are affected — identity, background, expertise, behavioral parameters, constraints, or interactions?
3. Is this a correction (something wrong), an addition (something missing), or a removal (something that does not belong)?

Do not proceed until the scope is clear. If the answers suggest the role needs fundamental rebuilding rather than targeted adjustment, surface that and ask whether a full redefinition would be more appropriate.

### Step 2 — Check for refinement log conflicts

Read the `## Refinement Log` at the bottom of the role file. If no such section exists, treat the log as empty.

Before proposing any change, evaluate it against the log: if a proposed change **reverses a previously logged entry** (re-adds something explicitly removed, or removes something explicitly added), do not proceed — surface the conflict to the user and stop until resolved:

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

If a proposed change in one section conflicts with another (e.g. a new constraint that contradicts an existing behavioral parameter), surface the conflict and resolve it before confirming.

### Step 4 — Confirm and write

Present all proposed changes together and ask for explicit confirmation. If adjustments are requested, revise and present again.

Apply only the confirmed changes to the role file at its existing location. Append a new row to the `## Refinement Log` table:

| Date | Scene / Context | Finding | Change Summary |
|---|---|---|---|
| <YYYY-MM-DD> | <scene name or "direct refinement"> | <the gap that prompted the change> | <one sentence describing what was changed> |

If the `## Refinement Log` section does not yet exist in the file, add it at the bottom with the table header before appending the first row.

---

## Process — Interview Draft Path

### Step 1 — Establish the need

Ask the user:

1. What is this role responsible for? What problem does it solve in the production?
2. What kind of work will actors in this role be doing?
3. What makes this role distinct from any existing role in the catalogue?

If the user's answers reveal overlap with an existing role, name the overlap and ask directly whether a new role is still warranted. Do not proceed until the need is confirmed as genuinely distinct.

### Step 2 — Draft and confirm

Using the understanding from Step 1, draft the complete role definition in a single pass — all sections:

1. **Identity** — title, domain (one or two words), one-sentence summary
2. **Background** — 2–3 paragraph professional history that shapes how this role thinks; a narrative, not a skills list
3. **Expertise** — specific domains and capabilities; every item must be distinct enough that it would not appear on another role's list
4. **Behavioral parameters** — all five: tone, verbosity, decision style, priorities (in priority order), working style; each specific enough to meaningfully constrain actor behaviour
5. **Constraints** — hard limits; each precise enough that an actor knows exactly when to stop. For cast roles: scope boundaries stated in terms of the work itself, not in terms of peer role ownership
6. **Interactions** — what this role receives, what it produces, and when it appears. For cast roles: describe handoffs in terms of what is passed, not who passes it — cast roles discover co-stars from the manuscript

Present the complete draft. If the user requests changes, revise and re-present.

### Step 3 — Confirm destination and write

Derive a kebab-case name from the role title. Use AskUserQuestion: "Should this role be global (available across all projects) or project-specific?"

- **Global** → `$HOME/.claude/slated/roles/role-<name>.md`
- **Project-specific** → `.claude/slated/roles/role-<name>.md`

Write the file to the confirmed location. Every section must be fully populated.

---

## Process — Summary Draft Path

Draft the complete role definition from the provided summary without interview, following the same section structure as the Interview draft path. Present the draft to the user, then confirm destination and write as in Interview draft Step 3.

---

## Output

- Role file written at the confirmed location (new role) or updated in place (refinement)
- Refinement log appended if changes were made to an existing role

---

## Rules

- Crew roles are immutable — they cannot be created or modified through this skill under any circumstances
- Never apply a change that reverses a logged refinement without surfacing the conflict to the user first
- Never write a partial draft — the full role must be complete before writing
- Never embed domain knowledge in the role definition — behavioral parameters only; knowledge belongs in backgrounds
- Never create a new role if an existing one could be adjusted — surface the overlap and ask first
- Never write vague constraints — every constraint must name exactly what the actor will not do
- Never write generic expertise items — every item must be specific enough that it would not appear on another role's list
- For cast roles: never name a specific peer role in a constraints or interactions section — cast roles discover co-stars from the manuscript at execution time
- Never write the file before confirmed — for refinements: all proposed changes confirmed; for new roles: destination confirmed
