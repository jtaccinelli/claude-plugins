---
name: scoping-tasks
description: Break a requirement into a confirmed task definition ready to be actioned.
argument-hint: "[task-name | brief description of what needs to be done]"
---

# Scoping Tasks

Translate a requirement into a complete, confirmed `task.md`, or identify and correct gaps in an existing one. A successful run leaves the task definition written, acceptance criteria verifiable, and agent definitions confirmed adequate — ready for actioning.

---

## Pre-work

1. Discover available agent definitions: Glob `.claude/agents/*.md`. Read each in full.
2. Discover available insights: Glob `.claude/dispatch/insights/*.md`. Read each in full.
3. Use AskUserQuestion: "Are you scoping a new task or refining an existing one?"
   - If **refining**: ask which task. Read `.claude/dispatch/tasks/<name>.md` in full. Proceed to the Refine path.
   - If **creating**: use `$ARGUMENTS` as a starting point if provided. Proceed to the Create path.

---

## Process — Create Path

### Step 1 — Establish the requirement

Use `$ARGUMENTS` as a starting point. If sufficient, proceed. If vague, ask focused questions:

1. **What is being built or fixed?** — One clear sentence.
2. **What type of change is this?** — New feature, bug fix, refactor, infrastructure, content, or other.
3. **What does success look like?** — What will be true when this task is complete that is not true now? Push for verifiable outcomes, not intentions.
4. **Are there constraints or edge cases to know about up front?** — Anything that would affect how the work is sequenced or who does it.

Do not proceed to agent assignment until these are clear.

### Step 2 — Name the task

Derive a short, descriptive kebab-case name from the requirement. Use AskUserQuestion to confirm: "The task will be named `<name>`. Does that look right?"

### Step 3 — Assign agents and insights

Dispatch a manager agent to determine assignments:

- Include in the agent prompt:
  - The requirement established in Step 1
  - The full list of available agent definitions and their descriptions
  - The full list of available insights and their descriptions
  - An instruction to propose: which agent definitions are needed, which insights apply to this task
  - An instruction to flag any required agent definition or insight that does not exist in the catalogue

Wait for the manager to complete. Review the proposed assignments.

**If any required agent definition is missing**: draft it autonomously — covering identity, responsibilities, behavioural parameters, and constraints. Ask whether to save it before writing to `.claude/agents/<name>.md`.

**If any required insight is missing**: draft it autonomously — covering the relevant technical or behavioural standards. Ask whether to save it before writing to `.claude/dispatch/insights/<name>.md`.

Surface a brief note listing every agent definition or insight that was auto-created.

### Step 4 — Define acceptance criteria

Write 3–6 verifiable acceptance criteria. Each must describe a concrete, observable outcome. Avoid vague language like "the feature works" or "it is complete".

Good: `User can submit the form and see the new item appear without a page reload`
Bad: `Form submission is implemented`

### Step 5 — Sequence the steps

Break the work into ordered steps, each assigned to a specific agent. Within each step, list atomic actions — what the agent must produce, not how they produce it.

Rules for actions:

- Each action has a clear, observable output
- Steps are sequential; if a step depends on the output of a previous step, say so explicitly
- Describe outcomes, not implementation methods

### Step 6 — Enter plan mode and draft the task definition

Call `EnterPlanMode`. Write the full task draft to the plan file using the template at `${CLAUDE_SKILL_DIR}/templates/task.md`. Populate every section:

- Task name and status (`confirmed`)
- Agents table
- Insights list
- Acceptance criteria
- Steps
- Notes
- HTML comment block and stub placeholder for the Pre-Action Review and Completion Record

Once written, call `ExitPlanMode` to surface the draft to the user for approval. Do not write any files until the user approves the plan.

### Step 7 — Write the task file

Once the plan is approved:

1. Write `.claude/dispatch/tasks/<name>.md` using the template

### Step 8 — Pre-action review

Validate the task definition before it is actioned. This step runs fully automatically — no user intervention at any point.

#### Part 1 — Manager agent definition review loop

Repeat until the manager returns `PASS` or 3 iterations have completed:

1. Dispatch a manager agent. Pass:
   - The path to the task definition
   - The paths to all referenced agent definitions
   - An instruction to evaluate whether each agent definition gives the agent enough to execute the task without guessing — not whether the definition is wrong for the job, but whether it is sufficiently detailed for this specific requirement
   - An instruction to return a structured verdict: `PASS` if all definitions are adequate, or `FAIL` with specific per-definition findings describing exactly what is missing

2. If `FAIL`: dispatch a manager agent again. Pass:
   - The path to the task definition
   - The paths to the flagged agent definitions only
   - The manager's findings for each flagged definition, verbatim
   - An instruction to revise each flagged definition in place — adding only the missing detail identified in the findings

   Wait for the manager to complete. Use the updated definitions in the next iteration.

3. If `PASS`: exit the loop and proceed to Part 2.

If the iteration cap is reached without `PASS`: collect the manager's final findings and skip Part 2. Proceed to the Outcome step.

#### Part 2 — Worker assumption review

For each agent in the task, dispatch a worker agent. Pass:

- The path to the task definition
- The agent's assigned steps only (extracted from the Steps section)
- An instruction to outline every assumption they would need to make to complete their steps — places where the task does not give them enough to act without deciding for themselves

Collect the assumption list returned by each worker.

#### Part 3 — Manager evaluation of assumptions

Dispatch a manager agent. Pass:

- The path to the task definition
- The paths to all referenced agent definitions
- All worker assumption reports from Part 2
- An instruction to categorise each assumption as one of:
  - `AGENT GAP` — the agent definition should specify how to handle this
  - `TASK GAP` — the task action is underspecified; the tasker needs to add detail
  - `ACCEPTABLE` — a reasonable inference that does not warrant a change
- An instruction to return a structured verdict: `CLEAN` if no `AGENT GAP` or `TASK GAP` findings exist, or `FLAGGED` with each non-acceptable item categorised and attributed to agent and step

#### Outcome

**Step A — Resolve task gaps:**

If Part 3 returned `FLAGGED` with any `TASK GAP` findings, dispatch a tasker agent. Pass:

- The path to the task definition
- The `TASK GAP` findings verbatim, each attributed to agent and step
- An instruction to fix those gaps by revising only the affected Steps (and Acceptance Criteria if applicable) — eliminating the ambiguity that caused each assumption
- An instruction not to touch the Pre-Action Review section or any other sections

Wait for the tasker to complete before proceeding.

**Step B — Document unresolved agent gaps:**

If the manager review loop ended with findings (cap reached), or Part 3 returned `FLAGGED` with any `AGENT GAP` findings:

Dispatch a tasker agent. Pass:

- The path to the task definition
- All remaining findings: manager definition assessment (if the Part 1 cap was reached) and any `AGENT GAP` worker assumptions from Part 3
- An instruction to replace the `## Pre-Action Review` section in the task definition with the consolidated feedback — populating the Manager Agent Definition Assessment table and the Worker Assumptions table (`AGENT GAP` items only) as applicable, and removing the HTML comment placeholder

If all parts returned clean verdicts and all task gaps were resolved in Step A, remove the entire Pre-Action Review section from the task definition — including the HTML comment block.

---

## Process — Refine Path

### Step 1 — Identify what needs to change

Read the existing task definition in full. Use AskUserQuestion: "What was missing or inaccurate in this task definition?"

Establish:

1. Which sections need updating — agents, insights, acceptance criteria, steps, or notes
2. Whether any agent definitions or insights referenced in the task need to be corrected

### Step 2 — Enter plan mode and draft the revised task definition

Call `EnterPlanMode`. Write the full updated task to the plan file, incorporating all identified corrections. Call `ExitPlanMode` to surface the revised draft to the user for approval. Do not write any files until the plan is approved.

### Step 3 — Write and re-review

Write the updated task definition. Run the full pre-action review (Steps 8, Parts 1–3 from the Create path) against the revised task.

---

## Output

- `.claude/dispatch/tasks/<name>.md` — complete task definition at `confirmed` status
- Pre-Action Review section populated if issues were found; removed entirely if the review returned clean
- Any auto-created agent definitions or insights written to their confirmed locations

Report the task file path, the number of agents assigned, the number of acceptance criteria defined, and the pre-action review outcome (clean or flagged with a summary count of findings).

---

## Rules

- Never write files before the user approves the plan — `EnterPlanMode` and `ExitPlanMode` are the confirmation gate, not `AskUserQuestion`
- Never assign an agent definition that does not exist — draft it before proceeding
- Never write acceptance criteria that cannot be verified by observation
- Never prompt the user during the pre-action review — it runs fully automatically
- Never skip the pre-action review — it runs after every confirmed task definition
- Never document a `TASK GAP` finding in the Pre-Action Review section — fix it in the Steps/Acceptance Criteria directly; only `AGENT GAP` findings are documented
