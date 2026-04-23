A director loads this background to know exactly what a valid Slated role file must contain — every required section, the specificity standard each section must meet, and the rules that distinguish cast from crew — so that any role drafted or evaluated during this skill meets the framework's minimum standard.

## Semantics

**File naming**: `role-<name>.md` where `<name>` is kebab-case derived from the role title. Examples: `role-design-engineer.md`, `role-systems-engineer.md`. Never use spaces, capitals, or underscores.

**Frontmatter**: every role file begins with YAML frontmatter containing exactly two fields:

```yaml
---
name: <kebab-case-name>
version: <semver e.g. 1.0>
---
```

No other frontmatter fields. No frontmatter on backgrounds — frontmatter is exclusive to role files.

**Required section headings** (in order):

1. `## Identity`
2. `## Background`
3. `## Expertise`
4. `## Behavioral Parameters`
5. `## Constraints`
6. `## Interactions`

**Identity subsections** (bold labels, inline): `**Title**`, `**Domain**`, `**Summary**`

**Behavioral Parameters subsections** (bold labels, inline — all five required): `**Tone**`, `**Verbosity**`, `**Decision style**`, `**Priorities**`, `**Working style**`

**Cast vs crew distinction**: cast roles must never name a specific peer role in Constraints or Interactions. Crew roles may. Cast roles must never embed domain knowledge in any section — behavioral parameters only. Domain knowledge belongs in backgrounds.

**Optional section**: `## Refinement Log` — appended at the bottom if and only if changes have been made to the role after its initial write. Never pre-populated.

## Structure

**Crew roles** (immutable, bundled with the framework): `slated/skills/perform/roles/role-<name>.md`. Cannot be created or modified through the cast skill.

**Global cast roles**: `~/.claude/slated/roles/role-<name>.md` — available across all projects on the device.

**Project-level cast roles**: `.claude/slated/roles/role-<name>.md` — available only within that project. Takes precedence over a global role with the same name.

**Resolution order**: crew bundle → project-level cast → global cast. On name collision between project and global, project wins.

**Section order** within the file:

```
---                          ← YAML frontmatter
name: <name>
version: <semver>
---

## Identity
**Title**: ...
**Domain**: ...
**Summary**: ...

## Background
<2–3 paragraphs of narrative>

## Expertise
- <specific capability>
- <specific capability>

## Behavioral Parameters
**Tone**: ...
**Verbosity**: ...
**Decision style**: ...
**Priorities**: ...
**Working style**: ...

## Constraints
- <hard limit>
- <hard limit>

## Interactions
<description of inputs, outputs, and when this role appears>

## Refinement Log         ← only if refinements have been made
| Date | Scene / Context | Finding | Change Summary |
|---|---|---|---|
| YYYY-MM-DD | ... | ... | ... |
```

## Function

**What Identity achieves**: gives an actor immediate orientation — title and domain anchor the actor's sense of scope; summary gives the actor a one-sentence operating mandate. A weak Identity (vague domain, summary that restates the title) leaves the actor without a clear mandate.

**What Background achieves**: shapes how the actor reasons and what it notices — through narrative history, not a skills list. A Background written as a bullet list of capabilities fails; it must read as a professional history that explains *why* the role approaches problems the way it does.

**What Expertise achieves**: scopes the actor's confidence and authority. An actor stays within listed expertise items; gaps in the list create gaps in output. Each item must be specific enough that it would not appear on a different role's list — "communication" or "problem-solving" are invalid because they are universal. A valid item: "Component state management using React hooks and context"; an invalid item: "Frontend development".

**What Behavioral Parameters achieves**: constrains every output the actor produces during the session. All five must be present. Each must be specific enough to meaningfully direct actor behaviour:

- **Tone**: not just "professional" — describe *how* professional. "Direct and precise — no qualifiers, no hedging, no meta-commentary" is valid; "professional and friendly" is not.
- **Verbosity**: not just "concise" — name what gets cut. "One sentence per point; no preamble, no trailing summary" is valid; "concise" alone is not.
- **Decision style**: describes *how* the actor resolves uncertainty. "Prefer the pattern already established in the codebase over introducing a new one" is valid; "careful" is not.
- **Priorities**: an ordered list of what matters most, where ordering resolves ties. "1. Correctness 2. Consistency with existing patterns 3. Brevity" is valid; a flat list without ordering is not.
- **Working style**: describes *how* the actor structures its work. "Complete one file before moving to the next; never leave a file in a broken state" is valid; "thorough" is not.

**What Constraints achieves**: tells the actor exactly when to stop. Each constraint must be precise enough that the actor knows without ambiguity whether it applies. "Never modify files outside the component directory" is valid; "stay in scope" is not. A constraint that cannot be applied without interpretation is not a constraint — it is a suggestion.

**What Interactions achieves**: orients the actor to its position in the production workflow — what it receives, what it produces, and when it appears. For cast roles: handoffs are described in terms of what is passed, not who passes it (cast roles discover co-stars from the manuscript at runtime). A cast role that names a specific peer role in Interactions violates the discovery model.

**Minimum specificity rule**: if an actor could satisfy the section text without changing its default behaviour, the section is not specific enough. Every section must materially constrain or direct what the actor does.
