Load context for the currently active feature in Edo's Framework.

Follow these steps exactly.

---

## Step 1 — Determine which project to load

Check if a project has already been established in this conversation (e.g. the user said "load JoyRide" or a previous skill already set context). If yes, skip to Step 2 with that project name.

If no project is known yet, scan for available projects:

```bash
ls "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/"
```

If the folder is empty or doesn't exist:
> "No projects found. Start one with `/new-feature [name]`."
Stop.

If one or more projects exist, list them and ask:
> "Which project would you like to load?
> [numbered list of project folders]"

Wait for the user to pick one. Store as PROJECT_NAME.

---

## Step 2 — Read the project's ACTIVE.md

Read `C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/[PROJECT_NAME]/ACTIVE.md`.

If the file says `ACTIVE FEATURE: none` or is missing:
> "No active feature for **[PROJECT_NAME]**. Start one with `/new-feature [name]`."
Stop.

---

## Step 3 — Read the feature file

Read the feature file at:
`C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/[PROJECT_NAME]/features/[FEATURE FILE].md`

---

## Step 4 — Read modified files

Read every file listed in the **Files Modified** table of the feature file so you have full context on the current code state. Do not scan the broader codebase — only read the listed files.

---

## Step 5 — Report

Respond with exactly this format:
```
---
**Project:** [PROJECT_NAME]
**Active feature:** [Feature Name]
**Status:** [Status]
**Last session:** [last bullet from Session Notes]
**Next step:** [first pending task based on requirements and session notes]
---
Ready to continue. What would you like to do?
```
