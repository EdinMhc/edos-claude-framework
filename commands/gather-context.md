Enter deep context-gathering mode for a feature or project. The sole goal of this command is understanding — not building. Claude will ask questions until it has a complete, real-world picture of what is being built, who uses it, and why it exists. No code is written here.

Follow these steps exactly.

---

## Step 1 — Set the stage

Tell the user:

> "Entering **context mode**. Before we write a single line of code, I want to fully understand what we're building — not just technically, but in real-life terms. The better I understand the *why*, the better every decision I make during implementation will be.
>
> Think of this like a project kickoff meeting. I'm going to ask you questions until I can picture exactly how this works in the real world — how someone opens it, what they do, why they care, and what would make it fail.
>
> Start by telling me: **what is this feature (or project), and what problem does it solve?** Don't worry about technical details yet — just tell me what it does for the person using it."

Wait for their answer. Store everything they say.

---

## Step 2 — First round of questions

Based on their answer, ask targeted follow-up questions. You are a detective. Every question should try to uncover something concrete that you don't yet know. Ask in a conversational way — not as a numbered list — to keep it feeling like a real dialogue.

**Core angles to probe across all rounds (not all at once — spread naturally):**

**Real-life usage**
- Walk me through exactly what happens when a user opens this. What do they see first?
- What is the user trying to accomplish when they use this feature?
- Give me a concrete example — a real scenario with a real type of person using it. What do they do step by step?

**The "why"**
- Why would someone choose to use this instead of doing it another way?
- What frustration or problem does this fix for them?
- What happens if this feature doesn't exist — how do they manage today?

**Boundaries and edge cases**
- Who uses this — everyone, or a specific type of user? (admin vs regular, logged in vs not, etc.)
- What can the user NOT do here? What's outside the scope?
- What would make this feel broken or wrong to the user?

**Design and feel**
- How do you picture it looking? Even roughly — one screen, a modal, a sidebar?
- When does this feature appear — always visible, triggered by an action, only in certain states?
- What's the most important thing the user should see or be able to do immediately?

**Success**
- How will you know this feature is working correctly from the user's perspective?
- What does "done" look like for you?

Pick the 2-3 most important unanswered questions from this list given what the user told you. Ask only those — don't dump the full list. Keep the conversation moving.

After asking, end with:
> "Take your time — the more detail you give me here, the more accurate the plan will be."

Wait for their answer.

---

## Step 3 — Write a running summary and check understanding

After each round of answers, write a summary of what you currently understand. Format it like this:

> "Here's what I understand so far:
>
> **What it is:** [one sentence on what the feature/project is]
> **Who uses it:** [who the users are and in what context]
> **The problem it solves:** [the real-world frustration or need]
> **How it works in practice:** [the concrete flow — what the user does, step by step, in plain language]
> **Why it matters:** [what breaks or gets worse without this]
> **Open questions I still have:** [anything you're still not sure about]
>
> Have I understood this correctly, or is there anything I've missed, misread, or left out?"

Wait for their answer.

- If they say you've missed something or got something wrong: correct your understanding, ask one more round of targeted questions (Step 2 again), then repeat the summary check.
- If they confirm you've got it right: proceed to Step 4.

Keep looping — question round → summary check — until the user confirms the understanding is complete. Do not move forward until you get explicit confirmation.

---

## Step 4 — Write the final context summary

Once the user confirms your understanding is complete, write the full context document. This is the permanent record.

Format:

```
# Context: [Feature or Project Name]
**Date:** [today's date]
**Project:** [PROJECT_NAME]
**Type:** [Feature / New Project / Enhancement]

---

## What This Is
[2-3 sentences. What it is, what it does, why it exists in this product.]

## Who Uses It and When
[Concrete description of the user. Not "users" — describe the actual person: their role, their context, when they reach for this feature and why.]

## The Problem It Solves
[The real-world frustration, gap, or need. What happens without this feature? How did they manage before?]

## How It Works in Real Life
[A narrative walkthrough. Not a spec — a story. "A user opens the app. They want to do X. They click Y. They see Z. They can do A and B. They cannot do C. When they're done, they expect D to have happened."]

## Why It's Built This Way
[The reasoning behind key design choices discussed. Why a modal vs a page. Why only admins. Why it's triggered by this action and not another.]

## What Good Looks Like
[How the user knows the feature is working correctly. The acceptance criteria in human terms, not technical terms.]

## Edge Cases and Boundaries
[What the feature does NOT do. Who cannot access it. What happens in error states. Any important limitations discussed.]

## Open Questions / Decisions Still Needed
[Anything not yet decided that will affect implementation. Flag these so they surface during /analyze.]
```

After writing the summary, tell the user:
> "Context captured. This is now my working understanding of what we're building and why. If anything changes or you think of something later, just tell me and I'll update it."

---

## Step 5 — Run a checkpoint

Run the /checkpoint skill to save session notes reflecting that context was gathered and confirmed.

Tell the user before doing it:
> "Saving a checkpoint now so this context is permanently recorded in the feature file."

---

## Step 6 — Offer to create or update the About file

Tell the user:

> "One more option: I can create (or update) an **About file** for this project at `projects/[PROJECT_NAME]/About.md`.
>
> This file is a growing knowledge base about your project — not code, not feature specs, just a living document that explains what the project is, how it works in real life, and how everything fits together. Every time we gather context on a new feature, it gets woven into this file so I always have the full picture.
>
> A few things worth knowing:
> - It lives locally in your `projects/` folder — **it is not tracked by Git and will never be pushed to GitHub.** It's strictly your machine's memory.
> - The more features we add context for, the more complete the picture becomes. Over time it becomes one of the most useful things I can read at the start of a session.
>
> Want me to create it (or update it if it already exists)?"

Wait for their answer.

**If yes:**

Read `projects/[PROJECT_NAME]/About.md` if it already exists. If it does, merge the new context into it — don't overwrite existing sections, append and enrich them.

If it doesn't exist, create it using this structure:

```markdown
# About: [Project Name]
*This file is local only — not tracked by Git. Updated by /context and /checkpoint.*

---

## What This Project Is
[What the product/app is. One paragraph, plain language.]

## Who It's For
[The real users — their role, their context, why they use this product at all.]

## Core Problem It Solves
[The main reason this product exists. What life looks like without it.]

## How It's Used in Real Life
[A narrative of the typical user journey through the product. Not a feature list — a story.]

## Features and How They Connect
[As features are added via /context, they get documented here. Each entry explains what the feature does and how it connects to the rest of the product.]

| Feature | What it does | How it connects |
|---|---|---|
| [Feature name] | [plain-language description] | [how it relates to other features or the core flow] |

## Key Design Decisions
[Non-obvious choices about how the product is built — and the real-world reason behind each one.]

## What We're Still Figuring Out
[Open questions, undecided directions, or areas the user flagged as uncertain.]
```

Confirm:
> "About file created (or updated) at `projects/[PROJECT_NAME]/About.md`. This won't appear in Git — it's local only. I'll update it every time we run /context or /checkpoint on this project."

**If no:** Acknowledge and move on. Don't bring it up again this session.

---

## Step 7 — Suggest next steps

Tell the user:

> "Context is locked in. Here's what makes sense next:
> - Run `/analyze` to turn this context into a concrete implementation plan
> - Run `/new-feature [name]` if you haven't set up the feature tracking file yet
> - Or just start building — I've got everything I need to make good decisions"
