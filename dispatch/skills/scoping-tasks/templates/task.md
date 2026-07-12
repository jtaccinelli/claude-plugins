# Task: <Task Name>

**Status**: confirmed | in-progress | complete
**Requirement**: <one-line description of what needs to be done>

---

## Agents

The agent definitions required to action this task.

| Agent | Definition |
|---|---|
| agent-1 | `<definition-name>` |
| agent-2 | `<definition-name>` |

---

## Insights

The insight documents that apply to this task — standards the output will be evaluated against.

- `<insight-name>` — <why it applies to this task>

---

## Acceptance Criteria

What a successful run of this task must produce. Written as verifiable outcomes, not intentions. The manager uses these to evaluate each run.

- [ ] <specific, verifiable outcome>
- [ ] <specific, verifiable outcome>

---

## Steps

The ordered sequence of work this task requires. Each step is assigned to an agent and describes the expected output, not the implementation method. Steps are ordered — earlier outputs may be inputs to later steps.

### Step 1: <step title>

**Agent**: `agent-1` (`<definition-name>`)

1. <atomic action — what to do and what the output should be>
2. <atomic action — what to do and what the output should be>

### Step 2: <step title>

**Agent**: `agent-2` (`<definition-name>`)

1. <atomic action — depends on output of Step 1>
2. <atomic action>

---

## Notes

<Any context that does not fit the structure above — constraints discovered during scoping, edge cases to watch for, or decisions made and why.>

---

<!--
  The tasker replaces this comment block and the stub below with pre-action review feedback
  if the review loop flags unresolved AGENT GAP issues. TASK GAP findings are fixed directly
  in the Steps/Acceptance Criteria sections above and are never documented here.
  If the review returned clean (or all gaps were resolved), remove this section entirely.
-->

## Pre-Action Review

> No review feedback recorded. Remove this section if the pre-action review returned clean.

### Manager — Agent Definition Assessment

| Agent Definition | Finding |
|---|---|
| `<definition-name>` | <specific gap in the definition> |

### Worker Assumptions

| Agent | Step | Assumption | Category |
|---|---|---|---|
| `agent-1` | Step 1 | <assumption text> | AGENT GAP |

---

<!--
  The manager replaces this entire comment block and the stub below with the completed
  Completion Record when the task is marked complete. Do not populate it manually during scoping.
-->

## Completion Record

> Replace this stub and the comment above with the completed record when finalising.

**Completed**: <YYYY-MM-DD>
**Runs required**: <N>

### Run Log

| Run | Status | Primary Failure Reason |
|---|---|---|
| 001 | fail | <brief reason> |
| 002 | pass | — |

### Agent Definition Improvement Notes

Patterns of repeated error across runs that suggest a gap in an agent definition or insight document. Omit entirely if no meaningful patterns emerged.

#### <definition-name>

- <specific gap in the definition that caused repeated errors>
- <insight document that was missing or incomplete>

### Task Improvement Notes

Structural issues in this task that caused agent confusion or deviation — ambiguous action wording, missing sequencing constraints, underspecified acceptance criteria. Omit entirely if none emerged.

- <structural issue that led to ambiguity>