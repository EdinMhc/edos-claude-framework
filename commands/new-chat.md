Start a new chat in Edo's Framework.

**Parsing $ARGUMENTS:**
- If $ARGUMENTS contains a space, the first word is the project name and everything after is the chat name. Use these directly — skip step 1.
- If $ARGUMENTS has no space, treat it as the chat name only and follow step 1 to determine the project.

Follow these steps exactly:

1. **Determine project** (skip if project was parsed from $ARGUMENTS above):
   Read `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/` to find available projects. If multiple exist, ask the user which project this chat belongs to. Wait for their answer.

2. Read the active project's ACTIVE.md at:
   `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/ACTIVE.md`
   If ACTIVE FEATURE is not "none", warn the user that a chat is already active and ask if they want to end it first or create anyway. Stop unless they confirm.

3. Read `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/EdosFramework.md` to load the chat file template (Section D).

4. Ask the user clarifying questions about scope, edge cases, and acceptance criteria before writing any code or creating any files. Wait for their answers.

5. Create the chat file at:
   `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/chats/[CHAT_NAME].md`
   Use the exact template from Section D of EdosFramework.md. Populate the Requirements section from the user's description and their answers to your clarifying questions. Set Status to IN_PROGRESS and Started to today's date. Write the first Session Notes entry: "[today's date]: Chat created. Requirements confirmed with user."

6. Update `C:/Users/Ednmh/OneDrive/Desktop/PROJECTS/edos-claude-framework/projects/[PROJECT]/ACTIVE.md` to:
```
ACTIVE FEATURE: [CHAT_NAME]
File: chats/[CHAT_NAME].md
Status: IN_PROGRESS
Started: [today's date]
```

7. Confirm to the user: "Chat [CHAT_NAME] started for project [PROJECT]. Here are the requirements I've recorded: [list them]. Shall we begin?"
