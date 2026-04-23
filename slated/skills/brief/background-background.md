A director loads this background to know exactly what a valid Slated background file must contain — the required sections, the specificity standard each section must meet, and the minimum completeness requirements — so that any background drafted or evaluated during this skill can direct actor behaviour without gaps.

## Semantics

**File naming**: `background-<name>.md` where `<name>` is kebab-case derived from the domain. Examples: `background-tailwind.md`, `background-react-router.md`, `background-drizzle-d1.md`. Derive from the technology or domain name, not from a use-case description.

**No frontmatter**: background files do not have YAML frontmatter. The first line of the file is the opening statement.

**Opening statement**: one sentence, before any section heading, describing what domain the background covers and why it matters to an actor filling a relevant role. Must be specific enough that an actor immediately understands what they are about to absorb and why it is relevant. A weak opening statement ("This background covers React") fails — it does not explain relevance to a role or task.

**Required section headings** (in order):

1. `## Semantics`
2. `## Structure`
3. `## Function`

**Specificity rule**: every entry in every section must tell an actor exactly what to produce — no vague conventions. "Use descriptive variable names" is invalid. "Name event handler props `on<Event>` (e.g. `onSubmit`, `onClose`)" is valid.

**Optional section**: `## Refinement Log` — appended at the bottom if and only if changes have been made after the initial write. Never pre-populated.

## Structure

**Global backgrounds**: `~/.claude/slated/backgrounds/background-<name>.md` — available across all projects on the device.

**Project-level backgrounds**: `.claude/slated/backgrounds/background-<name>.md` — available only within that project. Takes precedence over a global background with the same name.

**Resolution order**: project-level first, then global. On name collision, project wins.

**File structure**:

```
<Opening statement — one sentence, no heading>

## Semantics
<naming conventions, terminology, typing idioms, syntax choices>

## Structure
<file layout, directory trees, import paths, where things live>

## Function
<runtime behaviour, patterns, recipes, code examples>

## Refinement Log         ← only if refinements have been made
| Date | Scene / Context | Finding | Change Summary |
|---|---|---|---|
| YYYY-MM-DD | ... | ... | ... |
```

**Refinement Log format**: markdown table with columns `Date | Scene / Context | Finding | Change Summary`. Each row is one refinement. Date in `YYYY-MM-DD`. Scene / Context is the scene name where the gap was discovered, or "direct refinement" if refined outside a scene. Finding is the gap that prompted the change. Change Summary is one sentence describing what changed.

## Function

**What Semantics achieves**: tells an actor exactly what to write — names, idioms, syntax choices, and terminology that must be consistent across all files in the domain. An actor reads Semantics and produces output that matches the project's conventions without guessing. A Semantics entry that does not include a concrete pattern or example fails this — it informs but does not direct. Each entry must be specific enough to verify: given a file, an actor can check it against the Semantics entries and identify violations.

**What Structure achieves**: tells an actor exactly where things live — file paths, directory layout, import path aliases, and where new artefacts belong. Structure must be reproducible from the text alone; an actor must be able to derive the correct file path for any new artefact without reading the codebase. Minimum requirements: at least one directory tree where the domain has a non-trivial layout, import path examples wherever path aliases or resolution conventions exist.

**What Function achieves**: tells an actor how to accomplish outcomes in the domain — initialisation, common patterns, side-effect management, recipes for frequent operations. Each pattern must be complete enough to implement from, not merely referenced. Minimum requirements: concrete code examples wherever a pattern is ambiguous without one; every recipe written to the point of being executable. A Function entry that describes a pattern without showing it is insufficient — actors implement from examples, not from descriptions.

**Minimum completeness requirements**:

- Semantics: every naming or syntax convention must include a concrete pattern. At least one example per entry.
- Structure: must include a directory tree if the domain has more than two directories. Must include import path examples if the domain uses path aliases.
- Function: must include runnable code examples wherever the pattern involves syntax that is domain-specific or non-obvious. Pseudo-code is not acceptable.

**Common failure modes**:

- Semantics entry with no example: "Components use PascalCase" — valid as a rule but incomplete without an example showing how it applies in context.
- Structure section with only prose: a paragraph describing the directory layout is harder to apply than a directory tree. When a tree would add clarity, it is required.
- Function recipe that stops before the complete pattern: describing *that* a pattern exists without showing the full implementation leaves the actor unable to apply it. Every recipe must include the complete code needed to execute the pattern.
- Vague opening statement: "This background covers Tailwind CSS" does not tell an actor why it is relevant to their role. Add the role context: "An actor loading this background knows the Tailwind CSS 4 conventions used in this project — utility class ordering, design token names, and responsive variant patterns — enabling consistent component styling without reference to external documentation."
