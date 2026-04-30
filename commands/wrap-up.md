Wrap up and close a completed feature in Edo's Framework. The feature name is: $ARGUMENTS

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
> "Could not locate Edo's Framework. Make sure it's installed in your Workflow folder."
Stop.

Set PROJECTS_PATH = FRAMEWORK_ROOT/projects

---

## Step 2 — Determine which project this feature belongs to

Check if a project is already known in this conversation. If yes, use it as PROJECT_NAME and skip to Step 3.

Scan available projects:
```bash
ls "PROJECTS_PATH/"
```

If only one project exists, use it automatically.

If multiple exist, ask:
> "Which project does **[$ARGUMENTS]** belong to?
> [numbered list]"

Wait for their answer. Store as PROJECT_NAME.

---

## Step 3 — Verify the feature is the active one

Read `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`.

If ACTIVE FEATURE does not match "$ARGUMENTS", warn the user:
> "The active feature for **[PROJECT_NAME]** is **[current feature]**, not **[$ARGUMENTS]**. Are you sure you want to wrap up **[$ARGUMENTS]**?"

Wait for confirmation before continuing.

---

## Step 4 — Read the feature file in full

Read `PROJECTS_PATH/[PROJECT_NAME]/features/$ARGUMENTS.md`.

---

## Step 5 — Fill in all sections

Update the feature file with the following:

1. **Files Modified** — fill in every file touched during this feature. For each file, write a specific one-line description of what changed (not just the path — what method was added, what endpoint was wired, what column was created). Replace any "(WIP)" entries with their final descriptions.

2. **Summary** — write 3–5 sentences: what was built, why it exists, and how it is used.

3. **Decisions Made** — complete with any remaining architectural notes not yet recorded.

4. **Test Checklist** — fill in how to verify the feature works end-to-end, if not already done.

5. **Status** → set to `DONE`

6. **Completed** → set to today's date

7. **Session Notes** → append: `- [today's date]: Feature wrapped up and marked DONE.`

Write the updated file back.

---

## Step 6 — Clear ACTIVE.md

Write to `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`:
```
ACTIVE FEATURE: none
```

---

## Step 7 — Prompt for git

Check current branch and uncommitted changes:
```bash
git branch --show-current 2>&1
git status --short 2>&1
```

If there are uncommitted changes:
> "There are uncommitted changes. Want me to commit everything before we close out?"

Wait for their answer. If yes, run `/commit`.

Then ask:
> "Want me to push to GitHub as well?"

Wait for their answer and act accordingly.

---

## Step 8 — Confirm

Tell the user:
> "Feature **[$ARGUMENTS]** wrapped up.
> Summary: [one sentence from the Summary section you wrote]
> [N] files modified."
