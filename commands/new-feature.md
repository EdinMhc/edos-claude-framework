Start a new feature in Edo's Framework. The feature name is: $ARGUMENTS

Follow these steps exactly.

---

## Step 1 — Detect framework root

Run:
```bash
for candidate in \
    "$USERPROFILE/OneDrive/Desktop/Workflow/edos-claude-framework" \
    "$HOME/OneDrive/Desktop/Workflow/edos-claude-framework" \
    "$HOME/Desktop/Workflow/edos-claude-framework" \
    "$HOME/Workflow/edos-claude-framework"; do
    [ -d "$candidate/projects" ] && echo "$candidate" && break
done
```

Store the result as FRAMEWORK_ROOT. If nothing is returned:
> "Could not locate Edo's Framework. Make sure it's installed in your Workflow folder and try again."
Stop.

Set PROJECTS_PATH = FRAMEWORK_ROOT/projects

---

## Step 2 — Determine which project to add the feature to

Check if a project is already established in this conversation. If yes, use it as PROJECT_NAME and skip to Step 3.

Scan available projects:
```bash
ls "PROJECTS_PATH/"
```

If no projects exist:
> "No projects found. Create one first with `/project [name]`."
Stop.

If only one project exists, use it automatically and proceed.

If multiple projects exist, ask:
> "Which project is this feature for?
> [numbered list]"

Wait for the user to pick one. Store as PROJECT_NAME.

---

## Step 3 — Check nothing is currently active

Read `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`.

If ACTIVE FEATURE is anything other than "none", warn the user:
> "**[PROJECT_NAME]** already has an active feature: **[current feature]**. You should wrap that one up before starting a new one — or let me know if you want to switch anyway."

Wait for their answer. If they want to proceed, continue. Otherwise stop.

---

## Step 4 — Ask clarifying questions

Before creating any files, ask 2–3 focused clarifying questions about scope, edge cases, and acceptance criteria. Wait for their answers before proceeding.

---

## Step 5 — Create the feature file

Create `PROJECTS_PATH/[PROJECT_NAME]/features/[ARGUMENTS].md` using this exact template:

```markdown
# Feature: [Name]
**Status:** IN_PROGRESS
**Started:** [today's date]
**Completed:** —

---

## Requirements
[Populated from user's description and answers to clarifying questions]

---

## Files Modified

| File | What changed |
|---|---|

---

## Decisions Made

---

## Session Notes
- [today's date]: Feature created. Requirements confirmed with user.

---

## Test Checklist

---

## Summary
—
```

---

## Step 6 — Update ACTIVE.md

Write to `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`:
```
ACTIVE FEATURE: [ARGUMENTS]
```

---

## Step 7 — Confirm

Tell the user:
> "Feature **[ARGUMENTS]** created for project **[PROJECT_NAME]**.
>
> Here are the requirements I've recorded:
> [list them]
>
> Shall we start?"
