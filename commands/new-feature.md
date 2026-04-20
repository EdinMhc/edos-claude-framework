Start a new feature in Edo's Framework. The feature name is: $ARGUMENTS

Follow these steps exactly:

1. Read `C:/Users/Ednmh/OneDrive/Desktop/Workflow/ACTIVE.md` to confirm no other feature is currently active. If one is active, warn the user and stop — do not overwrite it.

2. Read `C:/Users/Ednmh/OneDrive/Desktop/Workflow/EdosFramework.md` to load the feature file template (Section D).

3. Ask the user clarifying questions about scope, edge cases, and acceptance criteria before writing any code or creating any files. Wait for their answers.

4. Create `C:/Users/Ednmh/OneDrive/Desktop/Workflow/features/$ARGUMENTS.md` using the exact template from Section D of EdosFramework.md. Populate the Requirements section from the user's description and their answers to your clarifying questions. Set Status to IN_PROGRESS and Started to today's date. Write the first Session Notes entry: "[today's date]: Feature created. Requirements confirmed with user."

5. Update `C:/Users/Ednmh/OneDrive/Desktop/Workflow/ACTIVE.md` to:
```
ACTIVE FEATURE: $ARGUMENTS
```

6. Update the Feature Index table in `C:/Users/Ednmh/OneDrive/Desktop/Workflow/EdosFramework.md` (Section F) — add a new row with the feature name, status IN_PROGRESS, started date, and "—" for completed.

7. Confirm to the user: "Feature [$ARGUMENTS] created. Here are the requirements I've recorded: [list them]. Shall we start?"
