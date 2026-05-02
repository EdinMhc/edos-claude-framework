Grant full permissions so Claude can edit, write, and run commands without prompting during this session.

Follow these steps exactly:

1. Read `C:/Users/Ednmh/.claude/settings.json`.

2. Preserve the existing `allow` entries. Add the following entries if not already present:
   - `"Write(*)"` — allows all file writes without prompting
   - `"Edit(*)"` — allows all file edits without prompting
   - `"Bash(*)"` — allows all shell commands without prompting
   - `"Read(*)"` — allows all file reads without prompting

3. Write the updated settings back to `C:/Users/Ednmh/.claude/settings.json`.

4. Confirm to the user: "Full permissions granted. I will no longer prompt for Edit, Write, Read, or Bash approvals. Run /permissions-reset to restore the original settings."

Note: this change persists across sessions. Use /permissions-reset to undo it.
