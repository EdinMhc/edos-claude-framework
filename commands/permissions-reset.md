Restore settings.json to its original permission state, removing the broad allow rules added by /permissions-all.

Follow these steps exactly:

1. Read `C:/Users/Ednmh/.claude/settings.json`.

2. From the `allow` array, remove these entries if present:
   - `"Write(*)"` 
   - `"Edit(*)"` 
   - `"Bash(*)"` 
   - `"Read(*)"` 

3. Keep any other existing `allow` entries (e.g. specific Bash rules already there before /permissions-all was run).

4. Write the updated settings back to `C:/Users/Ednmh/.claude/settings.json`.

5. Confirm to the user: "Permissions restored. Claude will now prompt for approvals as normal."
