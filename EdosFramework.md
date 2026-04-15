# Edo's Framework — Claude Context Workflow Guide

## A. System Overview

This workflow system exists so that Claude can restore full project context instantly at the start of any session, without re-scanning the codebase. It also gives the user a clear view of what is being built, what has been finished, and why decisions were made.

**How it works:**
- Every feature gets its own file in `features/[Name].md`
- `ACTIVE.md` always points to the feature currently being worked on — Claude reads this first at every session start
- This file (`EdosFramework.md`) is the single source of truth — a fresh Claude instance reading only this file should be able to operate the entire system immediately
- Keywords trigger specific actions (see Section B)

**Why it improves efficiency:**
- Claude reads `Files Modified` in the feature file and goes directly to those files — no broad codebase scanning
- `Session Notes` survive context compaction — Claude knows exactly where the last session left off
- `Decisions Made` prevents re-litigating the same architectural questions across sessions
- A fresh Claude instance on any machine can pick up exactly where we left off by reading this file

---

## B. Keyword Reference

| Keyword | Who triggers it | What Claude does |
|---|---|---|
| `"I want to start a new feature and it will be called [NAME]"` | User | Creates `features/[NAME].md` from the template, populates Requirements from your description, updates `ACTIVE.md` and the Feature Index below |
| `"CHECKPOINT"` | User or Claude proactively suggests it | Updates Session Notes and Files Modified (WIP) in the active feature file without marking it done. Confirms what was saved. |
| `"WRAP UP [NAME]"` | User (when testing is done and you're satisfied) | Marks feature DONE: fills Files Modified fully with descriptions, writes Summary and Decisions Made, updates `ACTIVE.md` to "none", updates the Feature Index |

**Claude will also proactively ask:** *"Should we do a CHECKPOINT now?"* after meaningful blocks of work, until the user confirms they know the keywords well enough — then Claude tapers off.

---

## C. Feature Lifecycle

```
PLANNING → IN_PROGRESS → IN_REVIEW → DONE
```

| Status | Meaning |
|---|---|
| PLANNING | Requirements written, work not yet started |
| IN_PROGRESS | Actively being built |
| IN_REVIEW | User is testing — not yet confirmed complete |
| DONE | WRAP UP received, all sections filled, feature closed |

---

## D. Feature File Template

When a new feature starts, Claude creates `features/[Name].md` using this exact template:

```markdown
# Feature: [Name]
**Status:** PLANNING
**Started:** YYYY-MM-DD
**Completed:** —

---

## Requirements
[What needs to be implemented — populated at feature start from user description]

---

## Files Modified
[Populated progressively during implementation and fully at WRAP UP]

| File | What changed |
|---|---|

---

## Decisions Made
[Non-obvious architectural choices + the reason why — populated progressively and completed at WRAP UP]

---

## Session Notes
[Append one bullet per session: date + what was done + any blockers]

---

## Test Checklist
[How to verify this feature works end-to-end — populated during or after implementation]

---

## Summary
[3–5 sentences: what was built, why it exists, how it's used — written at WRAP UP]
```

---

## E. Self-Prompts for Claude

This section contains the exact prompts Claude gives itself at each stage of the workflow. A fresh Claude instance reading this file should follow these instructions immediately.

---

### On session start — no active feature

```
1. Read ACTIVE.md.
2. If ACTIVE FEATURE is "none", greet the user and ask what to work on next.
3. If ACTIVE FEATURE points to a feature, read that feature file fully before doing anything else.
4. Note: Status, Requirements (what's pending), Session Notes (latest entry), Decisions Made.
5. Do NOT scan the codebase until you understand the current feature state from the file.
6. Confirm with the user: "I've loaded context for [feature]. Last session: [last session note]. Ready to continue?"
```

---

### On session start — resuming an IN_PROGRESS feature

```
1. Read the feature file in full.
2. Identify which Requirements are pending (not yet implemented).
3. Read the Files Modified section — open those files directly to re-establish code context.
   Do NOT read files outside of this list unless a new requirement demands it.
4. Summarize status to the user in 2–3 lines: what's done, what's pending.
5. Ask: "Ready to continue from where we left off?"
```

---

### On new feature start

```
1. Create features/[NAME].md from the template in Section D.
2. Populate Requirements from the user's description. Ask clarifying questions
   about scope, edge cases, and acceptance criteria before writing any code.
3. Update ACTIVE.md with the feature name, file path, status IN_PROGRESS, and start date.
4. Add the feature to the Feature Index table in EdosFramework.md (Section F).
5. Write the first Session Notes entry: "[date]: Feature created. Requirements confirmed with user."
6. Confirm: "Feature [NAME] created. Here are the requirements I've recorded: [list]. Shall we start?"
```

---

### During implementation

```
After each significant unit of work (a new file created, a service method added, an endpoint wired up, a bug fixed):
- Append a bullet to Session Notes in the active feature file.
- Update Files Modified with any newly touched files (mark WIP if partially done).
- Proactively ask: "Should we do a CHECKPOINT now?" once a meaningful block is complete.
  Continue asking this until the user confirms they have the keywords memorized — then taper off to once per session.
```

---

### On CHECKPOINT

```
1. Update Session Notes: append "[date]: [what was done this session]."
2. Update Files Modified with every file touched so far. Mark "WIP" for incomplete changes.
3. Note any decisions made in Decisions Made section.
4. Confirm to user: "Checkpoint saved. Recorded: [brief bullet summary of what's in the file now]."
```

---

### On WRAP UP [NAME]

```
1. Fill Files Modified fully — every file touched, with a one-line description of what specifically changed.
   (Not just the path — what method was added, what column was added, what endpoint was wired.)
2. Write the Summary section: 3–5 sentences covering what was built, why it exists, and how it's used.
3. Complete Decisions Made with any remaining architectural notes.
4. Mark Status as DONE. Fill in Completed date.
5. Update ACTIVE.md: set ACTIVE FEATURE to "none".
6. Update the Feature Index in Section F: set status to DONE and fill Completed date.
7. Confirm: "Feature [NAME] wrapped up. Summary: [one sentence]. [N] files modified."
```

---

## F. Feature Index

| Feature | Status | Started | Completed |
|---|---|---|---|
| _(none yet)_ | — | — | — |

---

## G. Notes on Efficiency

- **Read `Files Modified` first when resuming** — it's the fastest path to code context. Go directly to those files. Skip broad codebase scanning.
- **`Session Notes` survive context compaction** — they're the continuity layer between sessions. Always append, never overwrite.
- **`Decisions Made` saves architecture re-discovery** — the hardest thing to re-derive is *why* something was done a certain way. One line at the time of decision is worth 30 minutes of re-reading code later.
- **`ACTIVE.md` is always the entry point** — Claude reads it first, every session. Keep it accurate.
- **This file is the system** — a fresh Claude instance reading only `EdosFramework.md` should be able to operate the entire workflow immediately, on any machine.

---

## H. About This System

Edo's Framework was designed collaboratively between Edo and Claude on 2026-04-14, originally for the JoyRide TMS project. It has since evolved into a general-purpose framework for any developer who wants Claude to work with full context across sessions, on any project.

**What prompted it:** Context compaction mid-feature and the need to re-scan the codebase at each new session were causing wasted time. The framework fixes both by giving Claude a structured, file-based memory that persists across sessions and survives context limits.

**Design principles:**
- Keywords are explicit and hard to trigger accidentally (`WRAP UP [NAME]` requires the feature name)
- Claude asks proactively about CHECKPOINT so the user doesn't have to remember the keyword while focused on code
- Every file is plain markdown — readable by a human, parseable by Claude, no tooling required
- The system is portable — move the `Workflow/` folder to any machine, point Claude at it, and it works
- Works with any project and any tech stack — the framework is project-agnostic
