Save progress for the currently active chat in Edo's Framework.

**Parsing $ARGUMENTS:**
- If $ARGUMENTS is provided (not empty), use it as PROJECT_NAME and skip Step 1's project scan/question.
- If $ARGUMENTS is empty, follow Step 1 normally.

Follow these steps exactly.

---

## Step 1 — Establish project and active chat

If PROJECT_NAME was set from $ARGUMENTS, skip to reading ACTIVE.md.

Otherwise, scan for available projects:
```bash
ls "C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/" 2>&1
```

If no projects exist, tell the user:
> "No projects found. Nothing to save."
Stop.

If projects exist and only one is present, use it automatically. If multiple exist, ask:
> "Which project are you saving progress for?"

Wait for their answer. Store as PROJECT_NAME.

Read `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT_NAME]/ACTIVE.md`.

If ACTIVE FEATURE is "none" or the file is missing:
> "No active chat for **[PROJECT_NAME]** — nothing to save."
Stop.

Store the active chat name as CHAT_NAME.

Check for the chat file — look in `chats/` first, fall back to `features/`:
- `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT_NAME]/chats/[CHAT_NAME].md`
- `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT_NAME]/features/[CHAT_NAME].md`

---

## Step 2 — Update the chat file

Read the chat file in full.

Make the following updates:

**Session Notes** — append a new bullet:
`- [today's date]: [summary of what was done this session, including any blockers or decisions]`

**Files Modified** — add any newly touched files not already listed. For files with incomplete changes, append "(WIP)" to the description. Do not remove or overwrite existing entries.

**Decisions Made** — if any non-obvious architectural or design decisions were made this session, append them with a one-line rationale. Skip if nothing new to add.

Write the updated file back.

---

## Step 3 — Update the About file (if it exists)

Check if `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT_NAME]/About.md` exists.

If it does:
- Read it in full
- Look at what was done this session (from the Session Notes you just wrote)
- If the work done this session adds new understanding about the project — a new chat taking shape, a design decision that reveals something about how the product works, a real-world implication that came up — update the relevant sections of the About file
- Specifically: update **Key Design Decisions** if a new decision was recorded
- Do not rewrite the About file wholesale — make targeted additions and updates only

If the About file does not exist, skip this step silently.

---

## Step 4 — Confirm to the user

Tell the user:
> "Progress saved for **[CHAT_NAME]** in **[PROJECT_NAME]**.
> - Session notes updated: [one sentence on what was recorded]
> - Files modified: [count] tracked
> [If About file was updated]: About file updated with latest context."
