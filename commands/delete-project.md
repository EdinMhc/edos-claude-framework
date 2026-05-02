Delete a project's tracking folder from Edo's Framework. The project name is: $ARGUMENTS

This removes the project's ACTIVE.md, features folder, and About.md from the framework.
It does NOT touch your actual code — only the framework tracking files are deleted.

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

Store as FRAMEWORK_ROOT. If nothing is returned:
> "Could not locate Edo's Framework. Make sure it's installed in your Workflow folder."
Stop.

Set PROJECTS_PATH = FRAMEWORK_ROOT/projects

---

## Step 2 — Determine which project to delete

If $ARGUMENTS is provided, use it as PROJECT_NAME.

If $ARGUMENTS is empty, scan available projects:
```bash
ls "PROJECTS_PATH/" 2>&1
```

If no projects exist:
> "No projects found — nothing to delete."
Stop.

If multiple projects exist and no argument was given, ask:
> "Which project would you like to delete?
> [numbered list]"

Wait for their answer. Store as PROJECT_NAME.

---

## Step 3 — Verify the project exists

```bash
ls "PROJECTS_PATH/[PROJECT_NAME]" 2>/dev/null
```

If not found:
> "No project named **[PROJECT_NAME]** found. Check the name and try again."
Stop.

---

## Step 4 — Check if it has an active feature

Read `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`.

If ACTIVE FEATURE is not "none", warn:
> "⚠️  **[PROJECT_NAME]** has an active feature: **[FeatureName]**. Deleting the project will also delete all its tracking files including this feature's history.
>
> Your actual code is safe — this only removes the framework's memory of the project."

---

## Step 5 — Confirm with the user

Tell the user:
> "I'm about to permanently delete the tracking folder for **[PROJECT_NAME]**:
>
> - `projects/[PROJECT_NAME]/ACTIVE.md`
> - `projects/[PROJECT_NAME]/features/` (all feature files)
> - `projects/[PROJECT_NAME]/About.md` (if it exists)
>
> Your actual code is not affected — only the framework tracking files will be removed.
>
> Type the project name to confirm: **[PROJECT_NAME]**"

Wait for the user to type the project name exactly. If they type anything else, cancel:
> "Cancelled — nothing was deleted."
Stop.

---

## Step 6 — Delete the project folder

```bash
rm -rf "PROJECTS_PATH/[PROJECT_NAME]"
```

Verify:
```bash
ls "PROJECTS_PATH/[PROJECT_NAME]" 2>/dev/null && echo "STILL_EXISTS" || echo "DELETED"
```

If DELETED:
> "Done. **[PROJECT_NAME]** has been removed from the framework. Your code is untouched."

If STILL_EXISTS:
> "Something went wrong — the folder couldn't be deleted. Try removing it manually at: `PROJECTS_PATH/[PROJECT_NAME]`"
