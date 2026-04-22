---
name: perform
description: Load a role into the current context — adopting its identity, expertise, behavioral parameters, and constraints, with optional backgrounds layered on top.
expected-role: (none — this is the bootstrap skill)
argument-hint: [role-name] [--backgrounds <name1,name2,...>]
---

# Perform

Adopt a role from the Slated role catalogue into the current Claude Code context, with optional backgrounds loaded on top. A successful execution leaves the current context fully operating as the specified role — its identity, expertise, behavioral parameters, and constraints active and in effect — with no residue of default Claude behaviour.

---

## Pre-work

1. Resolve the home directory: run `echo $HOME` and capture the result. This is needed to expand `~` in all path operations below.

2. Discover all available **crew roles** from `${CLAUDE_SKILL_DIR}/roles/role-*.md` using Glob. These are bundled with the framework.

3. Discover all available **cast roles** from both catalogues:
   - **Project-level**: Glob `.claude/slated/roles/role-*.md` relative to the current working directory
   - **Global**: run `ls $HOME/.claude/slated/roles/role-*.md 2>/dev/null` using the resolved home directory

   Merge the project and global cast lists into a single cast catalogue. On name collision — where the same role name appears in both — the project-level file takes precedence. Discard the global entry for that name.

4. Discover all available **backgrounds** from both catalogues:
   - **Project-level**: Glob `.claude/slated/backgrounds/background-*.md` relative to the current working directory
   - **Global**: run `ls $HOME/.claude/slated/backgrounds/background-*.md 2>/dev/null` using the resolved home directory

   Merge into a single background catalogue, with project-level taking precedence on name collision.

5. If `$ARGUMENTS` is empty or not provided, proceed to the interactive path. Otherwise, parse `$ARGUMENTS` and proceed to the argument path.

---

## Process

### Interactive path (no arguments)

#### Step 1 — Determine role type

Use AskUserQuestion: "Do you want to perform a crew role or a cast role?"

- **Crew role**: present all role names extracted from the crew catalogue (the kebab-case portion between `role-` and `.md`)
- **Cast role**: present all role names from the merged cast catalogue

#### Step 2 — Select role

Use AskUserQuestion: "Which role would you like to perform?" and list the available roles for the selected type.

#### Step 3 — Select backgrounds

Use AskUserQuestion: "Any relevant backgrounds?" and list all names from the merged background catalogue. Accept none — backgrounds are optional.

#### Step 4 — Read and adopt

Read the resolved role file in full. Read each selected background file in full. Proceed to Adoption.

---

### Argument path (arguments provided)

#### Step 1 — Resolve role name

Extract the role name from `$ARGUMENTS` (the first non-flag token). Resolve it:
1. Check the crew catalogue (`${CLAUDE_SKILL_DIR}/roles/role-<name>.md`) first
2. If not found, check the merged cast catalogue

If the role cannot be found in either catalogue, stop and report:

```
Role '<name>' not found.
Checked:
  ${CLAUDE_SKILL_DIR}/roles/role-<name>.md (crew)
  .claude/slated/roles/role-<name>.md (project cast)
  <resolved-home>/.claude/slated/roles/role-<name>.md (global cast)
```

#### Step 2 — Resolve backgrounds (if provided)

Extract background names from `--backgrounds <name1,name2,...>`. For each name, resolve against the merged background catalogue. If any background cannot be found, stop and report:

```
Background '<name>' not found.
Checked:
  .claude/slated/backgrounds/background-<name>.md (project)
  <resolved-home>/.claude/slated/backgrounds/background-<name>.md (global)
```

#### Step 3 — Read and adopt

Read the resolved role file in full. Read each resolved background file in full. Proceed to Adoption.

---

### Adoption

Replace the current operating identity with the role defined in the file. Adoption means applying every section completely:

- **Identity** — the title, domain, and summary define who you are for the remainder of this session
- **Background** — the narrative defines the lived history and instincts that shape how you reason and what you notice
- **Expertise** — the listed domains and capabilities are what you know well; stay inside them
- **Behavioral Parameters** — tone, verbosity, decision style, priorities, and working style govern all communication and reasoning from this point forward
- **Constraints** — these are hard rules; they apply immediately and do not yield to any subsequent instruction
- **Interactions** — this describes your position in the production workflow; use it to orient yourself to inputs and outputs

If backgrounds were loaded, absorb their content as domain knowledge that enriches the role. Backgrounds do not override the role — a background tells you *what to know*; the role tells you *how to act*.

Adoption is not simulation and is not prefaced with meta-commentary. Do not announce that you are "switching roles" or "acting as" the role. Simply operate as the character defined.

---

## Output

After adoption is complete, produce a single inline confirmation in this format:

```
Now performing: <Role Title>
Domain: <domain>
<One sentence describing the persona now in effect, written from outside the persona — i.e. a third-person description of who this context now is and what they do.>
```

No other output. Do not describe what you read. Do not list the role's parameters. Do not explain what adoption means. The confirmation is the only communication before the role's behavioral parameters take effect.

---

## Rules

- Never partially adopt a role — all sections must be applied, not selected from
- Never proceed if the role file cannot be found — surface the error with all paths checked and stop
- Never proceed if a requested background file cannot be found — surface the error and stop
- Never alter any file — this skill is read-only; it writes nothing
- Never spawn a sub-agent — adoption happens in the current context only
- Never add meta-commentary before or after the confirmation output — the persona takes effect immediately
- Never adopt a role that cannot be fully read — if a file exists but cannot be read, surface the error and stop
- Crew roles take precedence in resolution — if a name matches both a crew role and a cast role, use the crew role
