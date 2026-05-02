Archive a feature in Edo's Framework. The feature name is: $ARGUMENTS

Archiving moves the feature file out of the active features folder into an archive subfolder.
The history is preserved — it's just no longer cluttering the active list.

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

## Step 2 — Determine which project

Check if a project is already known in this conversation. If yes, use it as PROJECT_NAME and skip ahead.

Scan available projects:
```bash
ls "PROJECTS_PATH/" 2>&1
```

If only one project exists, use it automatically.

If multiple exist, ask:
> "Which project does **[$ARGUMENTS]** belong to?
> [numbered list]"

Wait for their answer. Store as PROJECT_NAME.

---

## Step 3 — Determine which feature to archive

If $ARGUMENTS is provided, use it as FEATURE_NAME.

If $ARGUMENTS is empty, list the features in the project:
```bash
ls "PROJECTS_PATH/[PROJECT_NAME]/features/" 2>&1
```

Ask:
> "Which feature would you like to archive?
> [numbered list of .md files, excluding the archive/ subfolder]"

Wait for their answer. Store as FEATURE_NAME (without .md extension).

---

## Step 4 — Check the feature exists

```bash
ls "PROJECTS_PATH/[PROJECT_NAME]/features/[FEATURE_NAME].md" 2>/dev/null
```

If not found:
> "No feature named **[FEATURE_NAME]** found in **[PROJECT_NAME]**."
Stop.

---

## Step 5 — Check if it's the active feature

Read `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`.

If ACTIVE FEATURE matches FEATURE_NAME, tell the user:
> "**[FEATURE_NAME]** is currently the active feature. Archiving it will clear the active feature for **[PROJECT_NAME]** — you'll need to run `/new-feature` to start tracking something new. Continue?"

Wait for confirmation.

---

## Step 6 — Create the archive folder and move the file

```bash
mkdir -p "PROJECTS_PATH/[PROJECT_NAME]/features/archive"
mv "PROJECTS_PATH/[PROJECT_NAME]/features/[FEATURE_NAME].md" \
   "PROJECTS_PATH/[PROJECT_NAME]/features/archive/[FEATURE_NAME].md" 2>&1
```

---

## Step 7 — Clear ACTIVE.md if needed

If the archived feature was the active one, write:
```
ACTIVE FEATURE: none
```
to `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`.

---

## Step 8 — Confirm

Tell the user:
> "**[FEATURE_NAME]** has been archived.
>
> It's now at: `projects/[PROJECT_NAME]/features/archive/[FEATURE_NAME].md`
>
> The history is preserved — it's just out of the active list. You can read it any time."
