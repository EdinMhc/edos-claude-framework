Save a checkpoint for the currently active feature in Edo's Framework.

Follow these steps exactly.

---

## Step 1 — Establish project and active feature

Check if a project and active feature are already known in this conversation. If yes, skip to Step 2.

If not, first detect the framework root:
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

Then scan for available projects:
```bash
ls "PROJECTS_PATH/" 2>&1
```

If no projects exist, tell the user:
> "No projects found. Nothing to checkpoint."
Stop.

If projects exist and only one is present, use it automatically. If multiple exist, ask:
> "Which project are you checkpointing?"

Wait for their answer. Store as PROJECT_NAME.

Read `PROJECTS_PATH/[PROJECT_NAME]/ACTIVE.md`.

If ACTIVE FEATURE is "none" or the file is missing:
> "No active feature for **[PROJECT_NAME]** — nothing to checkpoint."
Stop.

Store the active feature name as FEATURE_NAME and the feature file path as:
`PROJECTS_PATH/[PROJECT_NAME]/features/[FEATURE_NAME].md`

---

## Step 2 — Update the feature file

Read the feature file in full.

Make the following updates:

**Session Notes** — append a new bullet:
`- [today's date]: [summary of what was done this session, including any blockers or decisions]`

**Files Modified** — add any newly touched files not already listed. For files with incomplete changes, append "(WIP)" to the description. Do not remove or overwrite existing entries.

**Decisions Made** — if any non-obvious architectural or design decisions were made this session, append them with a one-line rationale. Skip if nothing new to add.

Write the updated file back.

---

## Step 3 — Update the About file (if it exists)

Check if `PROJECTS_PATH/[PROJECT_NAME]/About.md` exists.

If it does:
- Read it in full
- Look at what was done this session (from the Session Notes you just wrote)
- If the work done this session adds new understanding about the project — a new feature taking shape, a design decision that reveals something about how the product works, a real-world implication that came up — update the relevant sections of the About file
- Specifically: update the **Features and How They Connect** table if a feature was meaningfully progressed, and **Key Design Decisions** if a new decision was recorded
- Do not rewrite the About file wholesale — make targeted additions and updates only

If the About file does not exist, skip this step silently.

---

## Step 4 — Confirm to the user

Tell the user:
> "Checkpoint saved for **[FEATURE_NAME]**.
> - Session notes updated: [one sentence on what was recorded]
> - Files modified: [count] tracked
> [If About file was updated]: About file updated with latest context."
