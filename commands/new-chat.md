Start a new chat in Edo's Framework. The chat name is: $ARGUMENTS

Follow these steps exactly:

1. Read `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/` to find available projects. If multiple exist, ask the user which project this chat belongs to.

2. Read the active project's ACTIVE.md at:
   `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/ACTIVE.md`
   If ACTIVE FEATURE is not "none", warn the user that a chat is already active and ask if they want to end it first or create anyway. Stop unless they confirm.

3. Read `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/EdosFramework.md` to load the chat file template (Section D).

4. Ask the user clarifying questions about scope, edge cases, and acceptance criteria before writing any code or creating any files. Wait for their answers.

5. Create the chat file at:
   `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/chats/$ARGUMENTS.md`
   Use the exact template from Section D of EdosFramework.md. Populate the Requirements section from the user's description and their answers to your clarifying questions. Set Status to IN_PROGRESS and Started to today's date. Write the first Session Notes entry: "[today's date]: Chat created. Requirements confirmed with user."

6. Update `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/ACTIVE.md` to:
```
ACTIVE FEATURE: $ARGUMENTS
File: chats/$ARGUMENTS.md
Status: IN_PROGRESS
Started: [today's date]
```

7. Confirm to the user: "Chat [$ARGUMENTS] started. Here are the requirements I've recorded: [list them]. Shall we begin?"
