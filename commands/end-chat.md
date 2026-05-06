End and close a completed chat in Edo's Framework.

**Parsing $ARGUMENTS:**
- If $ARGUMENTS contains a space, the first word is the project name and everything after is the chat name. Use these directly — skip step 1.
- If $ARGUMENTS has no space, treat it as the chat name only and follow step 1 to determine the project.

Follow these steps exactly:

1. **Determine project** (skip if project was parsed from $ARGUMENTS above):
   Read `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/` to identify the active project. If multiple exist and the project isn't already known, ask the user.

2. Read the project's ACTIVE.md to confirm the active chat matches the chat name. If it does not match, warn the user and stop — do not close the wrong chat.

3. Find the chat file — check chats/ first, fall back to features/:
   - `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/chats/[CHAT_NAME].md`
   - `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/features/[CHAT_NAME].md`
   Read it in full.

4. Update **Files Modified**: fill in every file that was touched during this chat. For each file, write a specific one-line description of what changed (not just the path — what method was added, what column was added, what endpoint was wired). Replace any "(WIP)" entries with their final descriptions.

5. Write the **Summary** section: 3–5 sentences covering what was built, why it exists, and how it is used.

6. Complete the **Decisions Made** section: add any remaining architectural notes not yet recorded.

7. Fill in the **Test Checklist** section if not already complete: how to verify this feature works end-to-end.

8. Set **Status** to `DONE` and set **Completed** to today's date.

9. Append a final bullet to **Session Notes**: `- [today's date]: Chat ended and marked DONE.`

10. Write the updated chat file back.

11. Update the project's ACTIVE.md to:
```
ACTIVE FEATURE: none
```

12. Confirm to the user: "Chat [CHAT_NAME] ended. Summary: [one sentence]. [N] files modified."

13. **Test coverage check** — before prompting for git:
    a. Check if `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/Tests.md` exists.
    b. If it does NOT exist: skip — this project has no tests set up yet, no reminder needed.
    c. If it DOES exist: read it. Check if the current chat ([CHAT_NAME]) appears in the per-chat coverage records.
    d. If the current chat already has test coverage: skip — user already created tests this session.
    e. If the current chat has NO coverage recorded: say:
       > "This project has tests set up, but **[CHAT_NAME]** doesn't have test coverage yet. Would you like to run `/add-test [PROJECT]` before closing?"
    f. Wait for their answer. If yes: run `/add-test [PROJECT]`. If no: continue to git step.

14. Always prompt for git — no exceptions:
    a. Check current branch with `git branch --show-current`.
    b. Check what's uncommitted with `git status --short`.
    c. If there are uncommitted changes, offer to commit and push.
    d. If nothing uncommitted: "All changes already committed. Want me to push?"
    e. Wait for answer — then execute immediately with a suggested commit message covering the full chat.
