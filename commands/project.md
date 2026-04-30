Create a new project in Edo's Framework. Usage: /project [ProjectName] or /project [ProjectName] — [description of what you want to build].

The project name is: $ARGUMENTS

Follow these steps exactly.

---

## Step 1 — Parse the arguments

The user may provide just a name, or a name followed by a description. Parse $ARGUMENTS like this:

- If $ARGUMENTS contains " — " (em dash with spaces) or " - " (hyphen with spaces), split on the first occurrence: everything before is the PROJECT_NAME, everything after is the DESCRIPTION.
- If $ARGUMENTS is a single word or phrase with no separator, the whole thing is PROJECT_NAME and DESCRIPTION is empty.

Strip any leading/trailing whitespace from both.

---

## Step 2 — Check if the project already exists

Check:
```bash
ls "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/[PROJECT_NAME]" 2>/dev/null
```

If it already exists, tell the user:
> "A project called **[PROJECT_NAME]** already exists. To start working on it, run `/active` and pick it from the list."

Stop.

---

## Step 3 — Create the project folder structure

Run:
```bash
mkdir -p "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/[PROJECT_NAME]/features"
```

Create `ACTIVE.md`:
```
ACTIVE FEATURE: none
```

Write it to:
`C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/[PROJECT_NAME]/ACTIVE.md`

---

## Step 4 — Create About.md if a description was given

If DESCRIPTION is not empty, create `About.md` at:
`C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/projects/[PROJECT_NAME]/About.md`

Use this structure, filling in what you know from the description:

```markdown
# About: [PROJECT_NAME]
*This file is local only — not tracked by Git. Updated by /gather-context and /checkpoint.*

---

## What This Project Is
[Write 1-2 sentences based on the user's description. If the description is vague, summarise faithfully — don't invent details.]

## What We're Still Figuring Out
- Full requirements and features not yet defined — run /gather-context to flesh this out.
```

If DESCRIPTION is empty, skip this step entirely — do not create About.md.

---

## Step 5 — Confirm to the user

If DESCRIPTION was provided:
> "Project **[PROJECT_NAME]** created.
>
> 📁 `projects/[PROJECT_NAME]/`
> ├── `ACTIVE.md` — no active feature yet
> ├── `features/` — ready for feature files
> └── `About.md` — seeded with your description
>
> When you're ready to start building, run:
> - `/gather-context` to define what you want to build in detail
> - `/new-feature [name]` to create the first feature tracking file
> - `/active` to load this project in a future session"

If no description was provided:
> "Project **[PROJECT_NAME]** created.
>
> 📁 `projects/[PROJECT_NAME]/`
> ├── `ACTIVE.md` — no active feature yet
> └── `features/` — ready for feature files
>
> When you're ready to start building, run:
> - `/gather-context` to define what you want to build in detail
> - `/new-feature [name]` to create the first feature tracking file
> - `/active` to load this project in a future session"
