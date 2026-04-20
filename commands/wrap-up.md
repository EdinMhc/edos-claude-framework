Wrap up and close a completed feature in Edo's Framework. The feature name is: $ARGUMENTS

Follow these steps exactly:

1. Read `C:/Users/Ednmh/OneDrive/Desktop/Workflow/ACTIVE.md` to confirm the active feature matches "$ARGUMENTS". If it does not match, warn the user and stop — do not close the wrong feature.

2. Read the feature file at `C:/Users/Ednmh/OneDrive/Desktop/Workflow/features/$ARGUMENTS.md` in full.

3. Update **Files Modified**: fill in every file that was touched during this feature. For each file, write a specific one-line description of what changed (not just the path — what method was added, what column was added, what endpoint was wired). Replace any "(WIP)" entries with their final descriptions.

4. Write the **Summary** section: 3–5 sentences covering what was built, why it exists, and how it is used.

5. Complete the **Decisions Made** section: add any remaining architectural notes not yet recorded.

6. Fill in the **Test Checklist** section if not already complete: how to verify this feature works end-to-end.

7. Set **Status** to `DONE` and set **Completed** to today's date.

8. Append a final bullet to **Session Notes**: `- [today's date]: Feature wrapped up and marked DONE.`

9. Write the updated feature file back.

10. Update `C:/Users/Ednmh/OneDrive/Desktop/Workflow/ACTIVE.md` to:
```
ACTIVE FEATURE: none
```

11. Update the Feature Index table in `C:/Users/Ednmh/OneDrive/Desktop/Workflow/EdosFramework.md` (Section F): set the row for "$ARGUMENTS" to status DONE and fill in the Completed date.

12. Confirm to the user: "Feature [$ARGUMENTS] wrapped up. Summary: [one sentence]. [N] files modified."
