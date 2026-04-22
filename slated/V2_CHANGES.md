# Slated V2 Outline

I want to signficantly improve on the way we engage with Slated and how the resources around this are managed, using properly structure Skills to achieve this.

## Skills Outline

1. Perform - this is intended to be a catch all skill for determining what background and role an agent or subagent needs to be consuming, capturing both crew and cast roles.
2. Produce - this is intended to capture the initialisation process for a project, whether it's being set up fresh or brought into the framework from an existing codebase. For existing projects, the exploration phase is handled internally using the same logic as Scout before handing off to the producer.
3. Scout - this is intended to be a skill for scaffolding and maintaining an accurate document of the structure of the project, captured as 'shot locations'. This is key to the scene process to provide actors with a clear direction on where they can find the changes they need to make are. This should both capture a scaffolding process if no shot location list previously exists, or should re-evaulate structural changes after initalisation.
4. Cast - this is intended to be a catch all skill for creating new roles and refining existing ones based on provided commentary. An interview process should be optional and available if the command is called, but there should be an option to select if you want the role to just be defined on the fly. Since this is now a process which is actively refined when in use, I am less concerned with role accuracy up front and expect the framework to be able to improve upon itself without my input.
5. Brief - this is intended to capture all the background knowledge tasks - establishing and refining.
6. Write - this is intended to capture the planning phase, prior to the actionoing of a piece of work.
7. Shoot - this is intended to capture the actioning of a planned piece of work.

### Perform

Inputs:

- A role name, the skill should be able to identify if this is cast or crew.
- A list of backgrounds (optional), identified by their prefixes.

IF NO INPUTS ARE PROVIDED:

- AskUserQuestion: Do you want me to perform a cast or crew role?
- Which role would you like me to perform? (Provide list of cast or crew roles depending on previous answer)
- Any relevant backgrounds? (Provide a list of available backgrounds)

Expected behavior:

- Consume all details of provided role or background roles in their entireity.

### Produce

Inputs:

- None required — the skill determines the path through interview.

Expected behavior:

- Run perform producer
- AskUserQuestion: Are you setting up a new project or bringing in an existing one?

If new project:

1. Interview the user to understand what is being built — goals, technologies, structure, constraints.
2. Verify that all required backgrounds and cast roles exist. If any are missing, draft them autonomously using the same logic as Brief and Cast.
3. Author a shot locations document at `.claude/slated/locations.md` based on the intended structure.
4. Confirm the locations list with the user before proceeding.
5. Scaffold the project directories and configuration from the locations list.

If existing project:

1. Run the Scout exploration phase — scan the codebase, discover technologies and directory structure, and produce a candidate shot locations document at `.claude/slated/locations.md`.
2. From the scan, draft any backgrounds that are clearly warranted by the discovered stack. Surface these to the user for confirmation before writing.
3. Interview the user to identify required cast roles. Draft them autonomously, surfacing for confirmation.
4. Confirm the locations list with the user before finalising.

### Scout

Inputs:
None, this should infer based on context.

Expected behavior:
Run perform dop
Investigate if a shot list exists, if it does or doesn't, refine a long list of items.

### Cast

Inputs:

- A role name (optional)
- A brief role summary (optional)

IF NO INPUTS ARE PROVIDED:

- AskUserQuestion: Are you creating a new role or refining an existing one?
- If refining an existing one: Which role are you trying to refine? (Provide list of cast roles - crew roles should NOT be changable from this command)
- If we're creating a new role: Do you want to be interviewied, or would you like to give me a summary and I'll draft it for you?

Expected behavior:
Run perform director

If a role name is provided, immediately assume this is a refinement process. If we're refining an existing role, make sure to go through a comprehensive interview process to evaluate what changes need to be made.

If an input that isn't a role name is provided, that instead provides a summary of a role requirement, assume this is a role you want drafted without user input.

If we're creating a new role, follow the process as normal.

Any refinements that get made must be submitted to a refinement log within the refined file itself.

### Brief

Inputs:

- A background name (optional)
- A brief background summary (optional)

IF NO INPUTS ARE PROVIDED:

- AskUserQuestion: Are you creating a new background or refining an existing one?
- If refining an existing one: Which background are you trying to refine? (Provide list of backgrounds)
- If we're creating a new role: Do you want to be interviewied, or would you like to give me a summary and I'll draft it for you?

Expected behavior:
Run perform director

If a background name is provided, immediately assume this is a refinement process. If we're refining an existing background, make sure to go through a comprehensive interview process to evaluate what changes need to be made.

If an input that isn't a background is provided, that instead provides a summary of a background requirement, assume this is a role you want drafted without user input.

If we're creating a new background, follow the process as normal.

Any refinements that get made must be submitted to a refinement log within the refined file itself.

### Write

Mostly the same as current, but should ask if we're creating a new scene or refining an existig one.

Refining a manuscript should just capture what was missing from it or how it was inaccurate and then go thorugh the same loop as creating.

Creating should look like:

1. Run perform writer and pre work
2. Establish context
3. Name the scene - use AskUserQuestion to confirm
4. Cast the scene - dispatch the director actor to do this
   - If missing a background or role, the director should draft these autonomously
5. Define objectives
6. Sequences the shots within the scene
7. Run director review, evaluating manuscript for inconsistencies (captured in pre-shoot review currently)
8. Run any and all refinements
9. Once confirmed, surface final draft for review from user
10. Write scene.

### Shoot

This should capture from the user whether or not they want to shoot one take or engage in the shoot / take loop with AskUserQuestion.

On init, before anything else, assess the state of the current scene — check whether a take is already in progress or a shoot was interrupted. If mid-take or mid-shoot, resume from where it left off rather than starting fresh.

The process should look like:

1. Perform as director
2. Assess scene state — resume if mid-shoot, otherwise begin fresh
3. AskUserQuestion: Do you want to shoot a single take or run the full loop?
4. Execute take loop (up to 5 times)
5. Wrap scene

The take loop should be:

1. Update status
2. Dispatch actors
3. Review changed files / output w/ git worktrees
4. Dispatch writer actor for output review
5. Conduct director review
6. If either find issues, resolve issues automatically, running role and background refinements AND manuscript adjustments.

## Background vs Role

I want to make sure we include VERY CLEAR CONSTRAINTS in the clarity between the two, as the backgrounds vs the roles feel like they keep blurring into each other.

The main purpose of a role is that this is ONLY supposed to capture the values and behavior of an actor, NOT what specific things they work on and what they know. A role is there to provide a clear identity to an agent, so that if the work it does is in conflict with the values of another agent, those inconsistencies can be surfaced and addressed. It's designed to try and create an adversarial system to encourage feedback and improve on overall outputs.

A background is almost ALWAYS meant to be project specific. There are very very few backgrounds which won't be local, due to the fact that they need to capture everything a role can't.

Those requirements and what a role serves vs what a background serves should be clearly interchangeable.

## Agent Summary (Actor)

The crux of this can stay the same, but it should adopt the usage of the Perform skill. We also should double down on the messaging around Actors not being professionals; all their knowledge and expertise on how to perform within their scene comes from the context provided in their role descriptoins and provided backgrounds. The reason this is the case is ideally, the differing roles descriptions and arising gaps from lack of provided context naturally surfaces issues in what has been provided to the agent regarding the task they have been completing.

## Crew Roles

The crew role set is significantly reduced in V2:

- **Director** — absorbs the casting director. Responsible for takes, reviews, and role/background governance.
- **Writer** — unchanged. Responsible for manuscripts and manuscript fidelity reviews.
- **DOP (Director of Photography)** — replaces the set designer. Responsible for shot locations: authoring, maintaining, and re-evaluating the locations list. This is the role Scout performs as.

Producer and visualiser are dropped entirely. Post-production QA is not a framework-level concern — it is handled as a regular scene with the appropriate actors defined for the project. Scene status history is captured per-scene in `wrap.md`; there is no need for a cross-scene tracking document.

## Removals

Init and update are no longer needed with proper skill structure; they only existed to seed the crew role files, which are now self-contained within the framework.

The storyboard is removed. Shot locations cover structural navigation, and wrap records cover scene history. There is no need for a separate cross-scene status document.

The casting director role is absorbed into the director. The set designer role is replaced by DOP.
