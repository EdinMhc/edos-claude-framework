Load context for the currently active feature in Edo's Framework.

Follow these steps exactly:

1. Read `C:/Users/Ednmh/OneDrive/Desktop/Workflow/ACTIVE.md`.
   - If the status is "none" or the file says there is no active feature, respond: "No active feature. Ask me to start a new one with: I want to start a new feature and it will be called [NAME]." Then stop.

2. Read `C:/Users/Ednmh/OneDrive/Desktop/Workflow/EdosFramework.md` to load the workflow rules and keyword triggers.

3. Read the active feature file at `C:/Users/Ednmh/OneDrive/Desktop/Workflow/features/[ACTIVE FEATURE NAME].md`.

4. Read every file listed in the **Files Modified** table of the feature file so you have full context on the current code state. Do not scan the broader codebase — only read the listed files.

5. Respond with exactly this format:
   ---
   **Active feature:** [Feature Name]
   **Status:** [Status]
   **Last session:** [last bullet from Session Notes]
   **Next step:** [first pending task based on requirements and session notes]
   ---
   Ready to continue. What would you like to do?
