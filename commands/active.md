Load context for the currently active feature in Edo's Framework.

Follow these steps exactly.

---

## Step 1 — Determine which project to load

Check if a project has already been established in this conversation (e.g. the user said "load JoyRide" or a previous skill already set context). If yes, skip to Step 2 with that project name.

If no project is known yet, first detect the framework root:
```bash
for candidate in \
    "$USERPROFILE/OneDrive/Desktop/Workflow/edos-claude-framework" \
    "$HOME/OneDrive/Desktop/Workflow/edos-claude-framework" \
    "$HOME/Desktop/Workflow/edos-claude-framework" \
    "$HOME/Workflow/edos-claude-framework"; do
    [ -d "$candidate/projects" ] && echo "$candidate" && break
done
```
Store as FRAMEWORK_ROOT. Set PROJECTS_PATH = FRAMEWORK_ROOT/projects

Then list available projects:
```bash
ls "PROJECTS_PATH/"
```

If the folder is empty or doesn't exist:
> "No projects found. Start one with `/project [name]`."
Stop.

**Smart detection:** Check the current working directory Claude is opened in. Extract the folder name (e.g. if Claude is opened in `C:/Users/.../JoyRide`, the folder name is `JoyRide`). If a project folder with that exact name exists in the projects list, load it automatically without asking — just proceed silently to Step 2 with that project name.

If no match is found between the current directory and the projects list, and there is only one project in the list, load it automatically.

If no match is found and there are multiple projects, list them and ask:
> "Which project would you like to load?
> [numbered list of project folders]"

Wait for the user to pick one. Store as PROJECT_NAME.

---

## Step 2 — Read the project's ACTIVE.md

Read `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`.

If the file says `ACTIVE FEATURE: none` or is missing:
> "No active feature for **[PROJECT_NAME]**. Start one with `/new-feature [name]`."
Stop.

If About.md exists at `PROJECTS_PATH/[PROJECT_NAME]/About.md`, read it silently before continuing. Use it to understand the real-world context behind the project. Do not summarise it to the user unprompted — just let it inform your understanding.

---

## Step 3 — Read the feature file

Read the feature file at:
`PROJECTS_PATH/[PROJECT_NAME]/features/[FEATURE FILE].md`

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
