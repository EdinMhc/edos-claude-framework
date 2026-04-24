Analyze the codebase and generate a detailed implementation plan for a feature.

This command runs a deep, high-effort analysis — it reads requirements, scans relevant code, reasons carefully about architecture, and produces a step-by-step plan before a single line is written. Follow these steps exactly.

---

## Step 1 — Ask permission for high-effort mode

Before doing anything, tell the user:

> "This command runs a **high-effort analysis** — I'm going to read your requirements carefully, scan the relevant parts of the codebase, and think through the full implementation before presenting a plan. This takes more time than a normal response but produces a much more accurate, grounded plan.
>
> This is worth doing when:
> - The feature touches multiple files or layers
> - You're not sure where to start
> - You want to avoid going down the wrong path mid-build
>
> Ready to start the analysis? (yes / no)"

Wait for confirmation. If no, stop.

---

## Step 2 — Establish project and feature context

Check if a project and feature are already loaded in this conversation. If yes, skip to Step 3.

If not, scan for available projects:

```bash
ls "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/" 2>&1
```

If no projects exist:
> "No projects found in the framework yet. Start a project first with `/new-feature [name]`, then run `/analyze`."
Stop.

If one or more projects exist, ask:
> "Which project is this analysis for?
> [numbered list]"

Wait for the user to pick one. Store as PROJECT_NAME.

Read `C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/[PROJECT_NAME]/ACTIVE.md`.

If ACTIVE FEATURE is set, read that feature file fully and store the requirements. If ACTIVE FEATURE is "none", ask:
> "No active feature found for **[PROJECT_NAME]**. What feature do you want to analyze? Describe what you want to build and I'll work from that."

Wait for their description. Store it as the feature requirements.

---

## Step 3 — Clarify scope before analyzing

Before touching the codebase, ask any questions that would change the direction of the plan. Keep this tight — one round, maximum 3 questions. Only ask what you genuinely can't infer.

Examples of good clarifying questions:
- "Is this a new endpoint or an extension of an existing one?"
- "Should this be visible to all users or only admins?"
- "Does this replace the existing [X] or run alongside it?"

Tell the user:
> "Before I scan the code, a couple of quick questions to make sure the plan is aimed at the right thing:"

Wait for answers, then proceed.

---

## Step 4 — Scan the codebase

Read the files listed in the active feature's **Files Modified** section first (if any — these are already in context).

Then scan the broader codebase for areas relevant to the feature. Use your judgment on what to read based on the requirements. Typical areas to check:

- Entry points (Program.cs, main.ts, App.tsx, index.ts, routes)
- Existing services or controllers related to the feature domain
- Data models and entities related to what's being built
- Existing patterns in the codebase (how are services wired up? how are components structured?)
- Configuration files that might constrain the approach (auth setup, DB config, routing)
- Any existing partial implementation of the feature

Do NOT read every file in the project. Read only what is directly relevant to the feature being analyzed.

As you read, take note of:
- The existing patterns you'll follow (naming, structure, dependency injection style, etc.)
- What already exists that the feature can build on
- Any constraints, risks, or gotchas (tight coupling, missing abstractions, security implications)
- What's missing that will need to be created

Tell the user while scanning:
> "Scanning the codebase — reading [area] to understand [what you're looking for]..."

Keep these updates short and specific. One line per area scanned.

---

## Step 5 — Build the implementation plan

Think through the full implementation before writing anything. Consider:

- What is the correct order of steps? (e.g. data layer before service layer before controller before UI)
- What dependencies does each step have?
- Are there any steps that could break existing functionality?
- What's the simplest correct path — no over-engineering, no skipped steps?
- Where are the likely friction points or decisions the user will face mid-build?

Structure the plan as numbered phases. Each phase should be a complete, testable unit of work — not a single line, not an entire day.

---

## Step 6 — Present the plan

Present the plan in this format:

```
---
## Implementation Plan: [Feature Name]

**What we're building:**
[2-3 sentences. What the feature does, why it needs to exist, and how it fits into the existing system.]

**Approach:**
[1-2 sentences on the architectural strategy — e.g. "We'll add a new service layer method, expose it via a new controller endpoint, and consume it from the frontend using React Query."]

---

### Phase 1 — [Name]
**What:** [What will be built in this phase]
**Why:** [Why this comes first / what it enables]
**Steps:**
1. [Specific action — file, method, what it does]
2. [Specific action]
3. ...
**Done when:** [How you'll know this phase is complete]

### Phase 2 — [Name]
...

### Phase N — [Name]
...

---

**Risks / things to watch:**
- [Any gotcha, constraint, or decision point that could stall the build]
- [Any existing code that needs to change carefully]

**Estimated phases:** [N]
---
```

After presenting the plan, tell the user:

> "That's the full plan. Before we start: does anything look wrong, missing, or different from what you had in mind? This is the best moment to adjust — once we start implementing, changes get more expensive."

Wait for their feedback. Adjust the plan if needed.

---

## Step 7 — Prompt to begin implementation

Once the user is happy with the plan, say:

> "Ready to build. I'll go phase by phase — finishing and confirming each one before moving to the next. That way if anything needs adjusting mid-way, we catch it early before it cascades.
>
> Starting with **Phase 1: [Name]** — [one sentence reminder of what this phase does].
>
> Shall I begin?"

Wait for confirmation, then implement Phase 1 fully before asking about Phase 2.

After each phase completes:
> "Phase [N] done. Here's what was built: [brief bullet summary]. Ready for Phase [N+1]: [name]?"

Do not move to the next phase without confirmation. If something unexpected comes up mid-phase (missing file, wrong pattern, ambiguous requirement), stop and surface it immediately rather than making assumptions.
