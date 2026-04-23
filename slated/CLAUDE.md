# Framework: Agentic Coordination

This project uses a film-production-inspired agentic coordination structure. All autonomous work is performed by **actors** — a single, general-purpose agent type that adopts a **role** and optionally loads **backgrounds** before operating.

---

## Folder Reference

The framework distinguishes between **global** resources (shared across all projects on a device) and **project-level** resources (specific to this project).

### Framework (`slated/skills/perform/roles/`)

Crew roles are bundled with the Perform skill and are always available regardless of project. They are immutable from within the framework — editable only by hand outside of it.

Some skills also carry **skill-local backgrounds** — background documents co-located within the skill directory and loaded automatically during that skill's pre-work. These are private to the skill that owns them; they do not appear in the general background catalogue and cannot be loaded via `perform --backgrounds`. Current skill-local backgrounds:

| File | Skill | Purpose |
|---|---|---|
| `slated/skills/cast/background-role.md` | `/slated:cast` | Schema and minimum standard for role files |
| `slated/skills/brief/background-background.md` | `/slated:brief` | Schema and minimum standard for background files |

### Global (`~/.claude/`)

| Folder | Purpose |
|---|---|
| `~/.claude/slated/roles/` | Reusable cast role definitions — available in every project |
| `~/.claude/slated/backgrounds/` | Reusable domain knowledge — technology backgrounds shared across projects |
| `~/.claude/agents/` | The `actor` agent — the only agent type in this framework |
| `~/.claude/skills/` | All framework skills — available in every project |

### Project-level (`.claude/`)

| Folder | Purpose |
|---|---|
| `.claude/slated/roles/` | Project-specific cast roles — takes precedence over global roles with the same name |
| `.claude/slated/backgrounds/` | Project-specific backgrounds — takes precedence over global backgrounds with the same name |
| `.claude/slated/scenes/` | Individual scenes — manuscripts, takes, and wrap records |
| `.claude/slated/locations.md` | Shot locations — the named structural map of the project, maintained by the DOP |

**Lookup order for roles**: crew (bundled with Perform) → project-level cast → global cast. Cast roles are checked project-level first, then global. Crew roles are always sourced from the framework bundle.

**Lookup order for backgrounds**: project-level first, then global.

---

## How It Works

1. A **role** defines *how* an actor behaves — tone, priorities, constraints, interaction style. Roles contain no domain knowledge.
2. **Backgrounds** define *what* an actor knows — domain knowledge specific to a technology or area. Backgrounds contain no behavioral parameters.
3. An **actor** is a vessel with no innate expertise. It is instantiated with a role and optional backgrounds, then directed by a scene or skill. Gaps in provided context produce gaps in output — this surfaces the framework's own deficiencies, which is the intended behaviour.
4. A **skill** is a discrete job with an expected role. The skill is loaded into context; the role governs how it runs.
5. A **scene** captures a single feature or fix — its manuscript, all takes, and a wrap record.

---

## Production Lifecycle

### Initialisation

1. The **producer** initialises the project (`/slated:produce`) — either interviewing to understand a new project or scanning an existing codebase. Establishes `locations.md`, all required backgrounds, and all required cast roles before any scene is planned. No scene work begins until this is complete.

### Scene Planning

2. The **writer** drafts a `manuscript.md` from the requirement (`/slated:write`) — casting roles, defining verifiable objectives, and sequencing shots against named locations. The director validates role sufficiency and actor assumptions automatically in a pre-shoot review loop before the manuscript is confirmed.

### Production

3. The **director** runs the scene (`/slated:shoot`) — executing takes against the manuscript, auto-resolving cast role and background gaps inline, and wrapping the scene on a passing take. Wrap produces `wrap.md` and opens a PR — no separate step required.
4. If a take **fails**: issues are auto-resolved where possible; if the loop is running, the next take starts automatically. If the take limit (5) is reached, the director surfaces a blocked summary and returns control to the user.
5. If a take **passes**: the director wraps the scene inline — Completion Record appended, `wrap.md` written, PR opened.

### Structural Maintenance

6. The **DOP** maintains `locations.md` (`/slated:scout`) — re-running after any significant structural change to keep the map accurate for future scene planning.

---

## Key Roles

Roles divide into two categories:

**Crew** operate within the meta-narrative — building the production's foundations, governing how scenes run, and reviewing what was built. Crew roles have full awareness of the production structure and may reference other roles by name. Crew roles are immutable from within the framework.

**Cast** operate through the meta-narrative — delivering actual features for the project. Cast roles discover their co-stars from the manuscript at execution time and do not name peers in their definitions. Cast roles are governed by the director and can be refined automatically during takes.

### Crew

| Role | Responsibility |
|---|---|
| `director` | Evaluates actor output against role definitions and background conventions. Governs cast role quality — drafting, refining, and resolving role gaps during takes. Most naturally operates in the main context. |
| `writer` | Creates manuscripts from requirements. Assesses manuscript fidelity during takes. Revises manuscripts when gaps are identified. |
| `producer` | Gates project initialisation — verifying backgrounds, cast roles, and confirming the locations document before any scene work begins. Does not build, does not plan scenes. |
| `dop` | Authors and maintains `locations.md` — the named structural map of the project. Deployed by Scout and by Produce on the existing-project path. |

### Cast

Cast roles are project-specific and are not bundled with the framework. They are defined using `/slated:cast` before any scene that requires them can be planned, and refined automatically by the director when gaps surface during takes.

---

## Skills Reference

| Skill | Expected Role | Purpose |
|---|---|---|
| `/slated:perform` | (none — bootstrap skill) | Load a crew or cast role into the current context, with optional backgrounds |
| `/slated:produce` | `producer` | Initialise a project — new (interview → locations → scaffold) or existing (scan → backgrounds → cast roles → locations) |
| `/slated:scout` | `dop` | Author or update `locations.md` — scan the project structure and confirm changes before writing |
| `/slated:cast` | `director` | Create a new cast role or refine an existing one — through interview, from a summary, or autonomously during a take |
| `/slated:brief` | `director` | Create a new background or refine an existing one — through interview, from a summary, or autonomously during a take |
| `/slated:write` | `writer` | Plan a new scene or refine an existing manuscript — through interview, with automatic pre-shoot review |
| `/slated:shoot` | `director` | Execute a scene — single take or full loop, with inline auto-resolution and wrap on pass |

---

## Naming Conventions

- Crew roles: `slated/skills/perform/roles/role-<name>.md`
- Global cast roles: `~/.claude/slated/roles/role-<name>.md`
- Project-specific cast roles: `.claude/slated/roles/role-<name>.md`
- Global backgrounds: `~/.claude/slated/backgrounds/background-<name>.md`
- Project-specific backgrounds: `.claude/slated/backgrounds/background-<name>.md`
- Skills: `.claude/skills/<skill-name>/SKILL.md`
- Shot locations: `.claude/slated/locations.md`
- Scenes: `.claude/slated/scenes/scene-<name>/`
- Takes: `.claude/slated/scenes/scene-<name>/takes/take-NNN.md`
- Wrap record: `.claude/slated/scenes/scene-<name>/wrap.md`
