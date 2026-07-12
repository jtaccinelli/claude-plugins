# Framework: Dispatch

Dispatch coordinates agents to complete complex tasks through a structured definition system. It provides measurable success criteria — pre-defined behavioural constraints validated against outputs — rather than open-ended instruction.

---

## Folder Reference

All project-level dispatch files live under `.claude/`:

| Folder                       | Purpose                                                             |
| ---------------------------- | ------------------------------------------------------------------- |
| `.claude/agents/`            | Agent definitions — role and behavioural prompts                    |
| `.claude/dispatch/insights/` | Insight documents — project knowledge and best-practice standards   |
| `.claude/dispatch/tasks/`    | Task definitions — discrete work items bridging agents and insights |

---

## Concepts

### Agents

Agent definitions provide role and tone prompts that govern how work is carried out. They are actively created and maintained by this framework's skills — not written by hand. Agent definitions also serve as evaluation criteria: after a task completes, the Worker's output is checked against the relevant agent definition to confirm behavioural constraints were respected.

### Insights

Insights are documents containing complex project knowledge. They act as evaluation points — standards a task output is checked against to confirm best practices were met. Insights can cover technical specifications, expected tool usage, or behavioural conventions specific to the project.

### Tasks

Tasks are discrete, actionable work items that bridge Agents and Insights. Each task captures what needs to be done, the acceptance criteria for success, prerequisites, and testing steps. Tasks reference the relevant Agent (governing how to behave) and Insights (governing what standards to meet).

---

## How It Works

1. **Agents** are catalogued — existing definitions are reviewed and gaps are identified or filled.
2. **Insights** are catalogued — project knowledge is gathered and structured as evaluation standards.
3. **Tasks** are scoped — requirements are broken into discrete tasks, each referencing the appropriate agent and insights.
4. The **Worker** actions tasks — executing work while adhering to the referenced agent definition and applying knowledge from the referenced insights.
5. The **Manager** evaluates outputs — checking that the Worker's output satisfies the task's acceptance criteria and that behavioural constraints were respected.
6. The **Tasker** writes and evaluates tasks — generating task definitions during scoping and assessing task outputs against acceptance criteria.

---

## Agents

Three agent types operate within this framework, each with a distinct responsibility:

| Agent     | Responsibility                                                                                                                                                                      |
| --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `worker`  | Actions tasks — executes work defined in a task, adhering to the referenced agent definition and isolating the correct insights for the task at hand                                |
| `manager` | Maintains definitions and evaluates outputs — creates and refines agent definitions used by the Worker, and checks whether behavioural constraints were respected after a task runs |
| `tasker`  | Scopes and evaluates tasks — generates task definitions from requirements, and assesses whether task outputs satisfy the stated acceptance criteria                                 |

---

## Skills Reference

| Skill                 | Purpose                                                                                                                                                      |
| --------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `cataloging-agents`   | Itemise existing agent definitions, assess coverage gaps, and create or update definitions as needed                                                         |
| `cataloging-insights` | Itemise existing insights, assess coverage gaps, and create or update insight documents as needed                                                            |
| `scoping-tasks`       | Break a requirement into discrete tasks — pulling together acceptance criteria, prerequisites, referenced agents, and insights                               |
| `actioning-tasks`     | Execute a task — loading the referenced agent definition, applying relevant insights, and producing output that can be evaluated against acceptance criteria |

---

## Naming Conventions

- Agent definitions: `.claude/agents/<name>.md`
- Insight documents: `.claude/dispatch/insights/<name>.md`
- Task definitions: `.claude/dispatch/tasks/<name>.md`
