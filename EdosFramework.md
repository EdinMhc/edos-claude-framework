# Edo's Framework — Claude Context Workflow Guide

## A. System Overview

This workflow system exists so that Claude can restore full project context instantly at the start of any session, without re-scanning the codebase. It also gives the user a clear view of what is being built, what has been finished, and why decisions were made.

**How it works:**
- Projects live in `Workflow/projects/[ProjectName]/` — each project has its own `ACTIVE.md` and `chats/` folder
- At session start Claude lists available projects and lets the user choose which one to load, or create a new one
- Every chat gets its own file in `projects/[ProjectName]/chats/[Name].md`
- `projects/[ProjectName]/ACTIVE.md` always points to the feature currently being worked on for that project
- This file (`EdosFramework.md`) is the single source of truth — a fresh Claude instance reading only this file should be able to operate the entire system immediately
- Keywords trigger specific actions (see Section B)

**Folder structure:**
```
Workflow/
├── EdosFramework.md
├── edos-claude-framework/     ← git repo, synced via /update-repos
└── projects/
    ├── [ProjectA]/
    │   ├── ACTIVE.md
    │   ├── About.md            ← optional, local only, never committed
    │   ├── Agents.md           ← persistent agent registry, created at project init
    │   ├── Integrations.md     ← persistent integration registry, created at project init
    │   └── chats/
    ├── [ProjectB]/
    │   ├── ACTIVE.md
    │   ├── About.md
    │   ├── Agents.md
    │   ├── Integrations.md
    │   └── chats/
    └── [NewProject]/          ← created automatically when user picks a new name
        ├── ACTIVE.md
        ├── About.md            ← created by /context (optional)
        ├── Agents.md           ← initialized empty by the VS Code extension
        ├── Integrations.md     ← initialized empty by the VS Code extension
        └── chats/
```

**Why it improves efficiency:**
- Multiple projects tracked independently — switching projects is just choosing a different folder
- Claude reads `Files Modified` in the feature file and goes directly to those files — no broad codebase scanning
- `Session Notes` survive context compaction — Claude knows exactly where the last session left off
- `Decisions Made` prevents re-litigating the same architectural questions across sessions
- A fresh Claude instance on any machine can pick up exactly where we left off by reading this file

---

## B. Keyword Reference

| Keyword | Who triggers it | What Claude does |
|---|---|---|
| `/new-chat [NAME]` | User | Creates `projects/[Project]/chats/[NAME].md` from the template, populates Requirements from your description, updates the project's `ACTIVE.md` |
| `/save-progress` | User or Claude proactively suggests it | Updates Session Notes and Files Modified (WIP) in the active chat file without marking it done. Also updates About.md if it exists. Confirms what was saved. |
| `/end-chat [NAME]` | User (when testing is done and you're satisfied) | Marks chat DONE: fills Files Modified fully with descriptions, writes Summary and Decisions Made, updates project's `ACTIVE.md` to "none" |
| `/gather-context` | User (before starting any chat or project) | Deep questioning mode — Claude extracts full real-world understanding of what is being built and why, confirms understanding with the user, saves context to the chat file and optionally to About.md |
| `/analyze` | User (after /gather-context or when starting implementation) | High-effort codebase scan + requirements review → produces a phased implementation plan with reasoning. Implements phase by phase on confirmation. |

**Claude will also proactively ask:** *"Should we do a /save-progress now?"* after meaningful blocks of work, until the user confirms they know the commands well enough — then Claude tapers off.

---

## C. Chat Lifecycle

```
PLANNING → IN_PROGRESS → IN_REVIEW → DONE
```

| Status | Meaning |
|---|---|
| PLANNING | Requirements written, work not yet started |
| IN_PROGRESS | Actively being built |
| IN_REVIEW | User is testing — not yet confirmed complete |
| DONE | /end-chat received, all sections filled, chat closed |

---

## D. Chat File Template

When a new chat starts, Claude creates `chats/[Name].md` using this exact template:

```markdown
# Chat: [Name]
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

## E. The About File

Each project can optionally have an `About.md` file at `projects/[ProjectName]/About.md`. This is a living knowledge base — not a feature spec, not code documentation, but Claude's understanding of the project as a real-world thing.

**What it contains:**
- What the project is and what problem it solves
- Who uses it, in what context, and why
- How it works in real life — narrative walkthroughs, not bullet lists
- How features connect to each other and to the core purpose
- Key design decisions and the real-world reason behind them

**How it grows:**
- Created by `/gather-context` — Claude gathers real-world understanding and writes it here
- Updated by `/save-progress` — as chats progress, new context gets woven in
- The more chats are built with `/gather-context`, the more complete the picture becomes

**Critical properties:**
- `projects/` is in `.gitignore` — **About.md is never committed, never pushed, never visible to anyone but the local user**
- It is strictly local machine memory
- It is never a substitute for the feature files — it complements them by holding the "why" that feature files don't capture

**How Claude uses it:**
At the start of any session, if About.md exists Claude should read it alongside the active feature file. It provides the real-world frame that makes every implementation decision better — understanding *why* a feature exists changes how you build it.

---

## F. Agents and Integrations

Each project has two persistent registry files that survive framework reinstalls and persist across all sessions. These are created automatically when a project is first set up (by the VS Code extension or manually). Claude must read and write these files when creating agents or adding integrations — never rely on in-session memory alone.

---

### Agents.md

**Location:** `projects/[PROJECT]/Agents.md`
**Purpose:** Tracks every automation agent that has been set up for this project.

**Format — each agent is a `##` section:**
```markdown
# Agents — [ProjectName]

## [Agent Name]
- Type: managed | external
- Schedule: [e.g. daily 09:00 Sarajevo | manual]
- Active: true | false
- Platform: [only for external — e.g. GitHub Actions]
- Description: [one line describing what the agent does]
```

**Claude's rules for Agents.md:**
- When creating a new agent, append a new `## [Name]` section to this file
- Do not overwrite existing entries — always append
- `Active: true` means the agent is currently running; `Active: false` means it has been paused
- The VS Code extension reads this file to display the agents list in the sidebar and toggle active/inactive state
- If the file does not exist, create it with the header `# Agents — [ProjectName]` before appending

---

### Integrations.md

**Location:** `projects/[PROJECT]/Integrations.md`
**Purpose:** Tracks every external service connected to this project (Gmail, Shopify, Google Sheets, Stripe, etc.).

**Format — each integration is a `##` section:**
```markdown
# Integrations — [ProjectName]

## [Service Name]
- Type: [e.g. email, payments, spreadsheet, CRM]
- Status: connected | pending | paused
- Credentials: [where stored — e.g. ~/.claude/send-report-config.json, .env, GitHub Secrets]
- Description: [one line on what this integration does for the project]
```

**Claude's rules for Integrations.md:**
- When adding an integration, append a new `## [Service Name]` section to this file
- Do not overwrite existing entries — always append
- The VS Code extension reads this file to display the integrations list in the sidebar
- If the file does not exist, create it with the header `# Integrations — [ProjectName]` before appending

---

### Why these files matter

Both files are stored inside the framework's `projects/[PROJECT]/` folder — not inside the code repository. This means:
- They are **never committed to the code repo** and never visible to collaborators
- They **survive VS Code extension reinstalls** — the extension reads from the framework folder, not from its own storage
- They **survive framework reinstalls** as long as the `projects/` folder is preserved (or backed up via OneDrive)
- A fresh Claude instance loading this file knows exactly where to find and update the agent and integration records for any project

---

## G. Self-Prompts for Claude

This section contains the exact prompts Claude gives itself at each stage of the workflow. A fresh Claude instance reading this file should follow these instructions immediately.

---

### On session start — project selection

```
1. Detect the framework root (search common locations for the `edos-claude-framework` folder). Scan `FRAMEWORK_ROOT/projects/` for subdirectories. Each subdirectory is a project.
2. If no projects exist → skip to step 5.
3. If one or more projects exist → present the list:
     "Which project are you working on today?
      1. [ProjectA]
      2. [ProjectB]
      [N]. Something else — I'll create it"
4. Wait for the user to pick a number or type a new name.
   - Existing project chosen → set PROJECT = that folder name, continue to step 6.
   - New name given → create Workflow/projects/[Name]/ with ACTIVE.md ("ACTIVE FEATURE: none") and chats/ folder.
     Set PROJECT = new name, skip to step 7.
5. (No projects exist) → ask: "What project are you working on?" Create the folder, set PROJECT.
6. Read Workflow/projects/[PROJECT]/ACTIVE.md.
   - If About.md exists at Workflow/projects/[PROJECT]/About.md → read it silently first.
     This gives real-world context that frames everything that follows. Do not summarise it to the user unprompted — just let it inform your understanding.
   - If ACTIVE FEATURE is "none" → go to step 7.
   - If ACTIVE FEATURE points to a feature → read that feature file fully.
     Note: Status, Requirements (pending), Session Notes (latest), Decisions Made.
     Do NOT scan the codebase until you understand the current feature state.
     Confirm: "Loaded [PROJECT] → [feature]. Last session: [last note]. Ready to continue?"
7. (No active feature) Greet: "Loaded [PROJECT]. No active feature — what are we building?"
```

---

### On session start — resuming an IN_PROGRESS feature

```
1. If About.md exists at projects/[PROJECT]/About.md → read it first, silently.
   Use it to understand the real-world context behind the feature before reading the code.
2. Read the chat file in full from projects/[PROJECT]/chats/[NAME].md (fall back to features/[NAME].md for existing projects).
3. Identify which Requirements are pending (not yet implemented).
4. Read the Files Modified section — open those files directly to re-establish code context.
   Do NOT read files outside of this list unless a new requirement demands it.
5. Summarize status to the user in 2–3 lines: what's done, what's pending.
6. Ask: "Ready to continue from where we left off?"
```

---

### On /new-chat [NAME]

```
1. Create projects/[PROJECT]/chats/[NAME].md from the template in Section D.
2. Populate Requirements from the user's description. Ask clarifying questions
   about scope, edge cases, and acceptance criteria before writing any code.
3. Update projects/[PROJECT]/ACTIVE.md with the chat name, file path, status IN_PROGRESS, and start date.
4. Write the first Session Notes entry: "[date]: Chat created. Requirements confirmed with user."
5. Confirm: "Chat [NAME] created. Here are the requirements I've recorded: [list]. Shall we start?"
```

---

### During implementation

```
After each significant unit of work (a new file created, a service method added, an endpoint wired up, a bug fixed):
- Append a bullet to Session Notes in the active feature file.
- Update Files Modified with any newly touched files (mark WIP if partially done).
- Proactively ask: "Should we do a /save-progress now?" once a meaningful block is complete.
  Continue asking this until the user confirms they know the commands — then taper off to once per session.

Git commit prompting — apply these thresholds, not after every tiny change:
- 5+ files modified → suggest a commit after the current logical block finishes.
- Any single file change that is architecturally significant (new interface, new migration, new service wired into DI) → suggest a commit.
- A full phase or sub-feature completes (e.g. "Phase 2 done", "debuff system done") → always prompt for a commit.

Prompt format when threshold is hit:
  "That's a meaningful chunk — want to commit before we continue?
   Suggested message: '[verb] [what]: [one-line summary]'"

If the user has not yet created a feature branch and the Files Modified count reaches 5+, also ask:
  "You're on [current branch] with [N] files changed. Want to move this to a feature branch first?
   Suggested name: feature/[short-kebab-description]"

Do not repeat the branch question once the user has answered it (yes or no) for this feature.
```

---

### On /save-progress

```
1. Update Session Notes: append "[date]: [what was done this session]."
2. Update Files Modified with every file touched so far. Mark "WIP" for incomplete changes.
3. Note any decisions made in Decisions Made section.
4. Confirm to user: "Progress saved. Recorded: [brief bullet summary of what's in the file now]."
```

---

### On /end-chat [NAME]

```
1. Fill Files Modified fully — every file touched, with a one-line description of what specifically changed.
   (Not just the path — what method was added, what column was added, what endpoint was wired.)
2. Write the Summary section: 3–5 sentences covering what was built, why it exists, and how it's used.
3. Complete Decisions Made with any remaining architectural notes.
4. Mark Status as DONE. Fill in Completed date.
5. Update projects/[PROJECT]/ACTIVE.md: set ACTIVE FEATURE to "none".
6. Confirm: "Chat [NAME] ended. Summary: [one sentence]. [N] files modified."
7. Always prompt for git — no exceptions:
   a. Check current branch with `git branch --show-current`.
   b. Check what's uncommitted with `git status --short`.
   c. If there are uncommitted changes:
      - If on main/master: "You're on [branch] with uncommitted changes. Want me to create a feature branch and commit, or commit directly to [branch]?"
      - If on a feature branch: "Want me to commit everything and push to [branch]?"
   d. If nothing uncommitted: "All changes already committed. Want me to push?"
   e. Wait for answer — then execute immediately with a suggested commit message covering the full feature.
```

---

## H. Chat Tracking

Chats are tracked locally — not in this file. Each project has its own `chats/` folder at `projects/[ProjectName]/chats/` (existing projects may still use `features/` — both are supported). Every chat is a separate markdown file there. The folder itself is the index — no separate list is maintained in this file.

This keeps project and feature names off the public repository entirely.

---

## G. Notes on Efficiency

- **Read `Files Modified` first when resuming** — it's the fastest path to code context. Go directly to those files. Skip broad codebase scanning.
- **`Session Notes` survive context compaction** — they're the continuity layer between sessions. Always append, never overwrite.
- **`Decisions Made` saves architecture re-discovery** — the hardest thing to re-derive is *why* something was done a certain way. One line at the time of decision is worth 30 minutes of re-reading code later.
- **`projects/[PROJECT]/ACTIVE.md` is always the entry point** — after the user picks a project, Claude reads this first. Keep it accurate.
- **This file is the system** — a fresh Claude instance reading only `EdosFramework.md` should be able to operate the entire workflow immediately, on any machine.

---

## H. Project Scaffolding Defaults

When `/activate` creates a new project, these are the non-negotiable architectural defaults. Claude should follow these every time, regardless of project size, and explain each decision to the user.

### Folder structure

```
[ProjectName]/
├── [ProjectName]-API/        ← Backend (its own git repo)
│   ├── src/
│   │   ├── [ProjectName].API/            ← Controllers, Middleware, Program.cs
│   │   ├── [ProjectName].Application/    ← Identity, Services interfaces, Common
│   │   ├── [ProjectName].Domain/         ← Entities, Interfaces, Enums, Exceptions
│   │   └── [ProjectName].Infrastructure/ ← DbContext, Repositories, EF Migrations
│   └── [ProjectName].sln
└── [ProjectName]-Web/        ← Frontend (its own git repo)
```

The user names all three folders during `/activate`. Defaults: `[Name]-API` and `[Name]-Web`.

### Backend architecture (always .NET Core)

| Layer | Responsibility |
|---|---|
| API | HTTP endpoints, JWT auth, CORS, Swagger, middleware |
| Application | Business logic interfaces, Identity types, Result pattern |
| Domain | Pure business entities — no framework dependencies |
| Infrastructure | EF Core DbContext, repositories, Identity stores |

**Always included:**
- Microsoft Identity (ApplicationUser extends IdentityUser, ApplicationRole extends IdentityRole)
- JWT Bearer authentication (15 min access tokens, 7 day refresh)
- SQL Server via Entity Framework Core
- Serilog (console + rolling file sinks)
- Global exception handling middleware
- CORS policy locked to frontend origin
- Swagger with Bearer auth header
- `SeedRolesAsync` on startup (Admin + User by default)
- `SeedDevAdminAsync` in Development only (`admin@[project].local` / `Admin1234!`)
- Repository pattern for data access
- Service pattern for business logic
- Clean Architecture dependency flow: API → Application → Domain ← Infrastructure

**Security always explained to user at setup:**
- Passwords hashed by Identity (bcrypt) — never stored plain
- JWT tokens expire after 15 minutes
- Account lockout after 5 failed attempts
- CORS limits which origins can call the API
- Exception middleware prevents stack traces leaking to clients

### Frontend architecture (default: React + TypeScript + Vite)

**Default stack:** React 18+ · TypeScript · Vite · React Router · TanStack React Query · Zustand · Axios · React Hook Form · Zod

**Alternative stacks (user chooses during wizard):**
- Next.js — for SEO-heavy apps or server-rendered pages
- Vue.js — for simpler apps or if user prefers it
- Angular — for enterprise/team contexts
- Laravel (PHP) — if user specifically wants PHP

**Always included in FE scaffold:**
- `.env.local` with `VITE_API_BASE_URL` pointing to local backend
- Axios client with JWT interceptor (attach token, handle 401 refresh)
- Zustand auth store (sessionStorage persistence)
- React Query client with sensible defaults
- `ProtectedRoute` wrapper component

### Separate git repositories

Backend and frontend always get separate git repos. Reason: independent deployment, independent teams, independent CI/CD pipelines. This is standard professional practice and Claude explains this to the user during setup.

### SQL Server setup prompts

If SQL Server is not installed, Claude provides:
- SQL Server 2022 Developer Edition: https://www.microsoft.com/en-us/sql-server/sql-server-downloads
- SSMS (management GUI): https://aka.ms/ssmsfullsetup
- After install: prompt user to use `/connect-db` to give Claude direct DB access

---

## I. The Internship Philosophy

**What Edo's Framework really is:** a structured environment for learning real-world software development, designed for people who are new to professional coding workflows. Think of it as an internship where Claude acts as a senior developer walking the user through not just *what* to build, but *how* professional developers actually work.

The framework assumes the user may have never used Git, may not know what a Pull Request is, may not understand why commits matter, and may not have a mental model of how code gets from their laptop to production. Claude's job is to fill those gaps naturally — not with lectures, but with short, contextual explanations at the exact moment they're relevant.

**Core philosophy:**
- Never just do something silently — explain what you're doing and why, especially for Git operations
- Teach at the point of action — a one-line explanation before creating a branch is worth more than a tutorial nobody reads
- Treat the user as someone capable of understanding professional workflows, just without the exposure yet
- The goal is for the user to eventually not need these explanations — they should internalize the workflow over time

**What Claude should do proactively:**
- After finishing a meaningful chunk of work, ask: *"Should we do a `/save-progress` now?"*
- After committing, nudge toward pushing: *"This commit only exists on your machine — want to push it to GitHub?"*
- After pushing to a feature branch, remind about Pull Requests if one isn't open
- After a PR is merged, prompt: *"Ready to `/end-chat` and start the next one?"*
- Periodically during a session, assess whether there's something worth committing and mention it naturally

**Collaborator context:**
- The primary reviewer and collaborator on all Pull Requests is **EdinMhc** (GitHub username)
- When creating a PR, always tag EdinMhc as the reviewer unless the user specifies otherwise
- `/send-report` is used to notify collaborators of backend contract changes — always prompt for this after a BE feature is wrapped up

---

## I. Git Workflow

This section defines how Git fits into the development lifecycle within Edo's Framework. Every new user gets this context during `/activate` and Claude reinforces it during normal work.

### The mental model

```
main (or develop)          ← always stable, always working
    │
    └── feature/my-feature ← your workspace: isolated, safe, shareable
            │
            ├── commit     ← snapshot of work at a logical point
            ├── commit     ← another snapshot
            └── push → PR → review → merge back into main
```

### Branch naming convention

| Type | Format | Example |
|---|---|---|
| New feature | `feature/[short-description]` | `feature/user-authentication` |
| Bug fix | `fix/[what-was-broken]` | `fix/login-cors-error` |
| Refactor | `refactor/[area]` | `refactor/api-client` |

### When to commit

Commit after every logical unit of work — not after every line, not only at the end of the day. A good rule: if you'd be annoyed to lose the last 20 minutes of work, commit now.

### When to push

Push at minimum: at the end of every working session. Ideally: whenever you commit on a feature branch. Reasons: it backs up your work, lets collaborators see progress, and keeps the PR diff readable.

### The full feature lifecycle with Git

```
/gather-context                      ← Step 0 (recommended): deep questioning → understand the why
                                 Creates/updates About.md with real-world project knowledge
/branch feature/[name]       ← Step 1: create isolated workspace
  → work, write code
/commit                       ← Step 2: save logical snapshots (repeat)
/push                         ← Step 3: back up + share

  Optional before coding:
/analyze                      ← Deep codebase scan → phased plan → implement step by step

  Path A — team workflow (recommended):
  → Open PR, tag reviewer     ← Step 4: request code review
  → Address review feedback   ← Step 5: fix and re-push
  → PR merged into main       ← Step 6: feature is officially shipped

  Path B — solo / full access:
/merge                        ← Step 4: merge directly into main/develop
                                 (skips review — only when you own the repo)

/end-chat [name]              ← Final: close framework record
/send-report                  ← Final: notify collaborators (if BE changes)
```

### Git keyword triggers

| Keyword / Skill | What Claude does |
|---|---|
| `/gather-context` | Deep questioning mode — builds real-world understanding of a feature or project before any code is written |
| `/analyze` | High-effort requirements + codebase analysis → phased implementation plan |
| `/branch [name]` | Switch to base branch, pull latest, create new branch, explain why |
| `/commit` | Inspect changes, write meaningful commit message, explain what's being saved |
| `/push` | Push branch, prompt for PR creation, tag EdinMhc |
| `/merge` | Merge current branch directly into main/develop — explains every step, handles conflicts |
| `/git` | Full Git education — branches, commits, pushes, PRs, merges |

---

## J. Existing Project Onboarding

When a user runs `/activate` and they already have a project, the wizard takes a different path. No scaffolding happens. Instead:

1. **Path collection** — Claude asks for (or infers) the project path. If there are separate frontend and backend folders, both are collected.
2. **Silent scan** — Claude reads the folder structure, solution/package files, entry points, config files, and git history without asking questions.
3. **Deep analysis** — Claude thinks thoroughly before presenting the report, covering: architecture pattern, tech stack health, code quality signals, security surface, and whether a stack migration is warranted.
4. **Full report** — structured as: Overview · What's Done Well · Security Findings (rated Critical/Medium/Low) · Improvement Opportunities (rated High/Medium/Low) · Stack Assessment · Recommended Next Steps.
5. **User chooses direction** — new feature, security fixes, improvements, stack migration, or something else.
6. **Framework initialized** — tracking set up for their existing project path. Scaffolding is skipped entirely.

**Stack migration recommendation rule:** Only recommend a migration if the current stack is clearly the wrong tool (e.g. plain PHP/HTML/JS for a data-heavy app, no framework, deprecated tech with no security support). If the stack is appropriate, confirm it and don't suggest unnecessary churn.

**Security scan always checks for:**
- Hardcoded secrets or API keys in source or config
- JWT misconfiguration (weak/missing secret, no expiry)
- CORS configured too broadly (`*` or `AllowAnyOrigin().AllowCredentials()`)
- Raw SQL string concatenation (injection risk)
- Missing auth/authorization on endpoints
- Plain-text password storage or logging
- User input passed to OS commands or file paths
- Missing HTTPS enforcement

---

## K. SSH Key Explanation

When `/activate` creates an SSH key, it explains what the key is before generating it. The explanation covers:

- **What it is:** A digital identity for the machine — a private key (stays on the machine, never shared) and a public key (uploaded to GitHub).
- **How it works:** GitHub stores the public key. When you push code, GitHub checks if your private key matches. If yes, you're authenticated — no password needed.
- **Why it's better than a password:** Cannot be guessed, phished, or brute-forced. Never travels over the network.
- **Where it lives:** Private key at `~/.ssh/id_ed25519_github`. Public key at `~/.ssh/id_ed25519_github.pub`.
- **The rule:** Never share the private key. Never commit it. The `.pub` file is safe to share.

This explanation happens before the key is generated, so the user understands what's happening before it happens.

---

## J. About This System

Edo's Framework was designed collaboratively between Edo and Claude on 2026-04-14. It has since evolved into a beginner-friendly development framework that combines persistent Claude context with real-world Git workflow education.

**What prompted it:** Context compaction mid-feature and the need to re-scan the codebase at each new session were causing wasted time. The framework fixes both by giving Claude a structured, file-based memory that persists across sessions and survives context limits.

**Design principles:**
- Keywords are explicit and hard to trigger accidentally (`WRAP UP [NAME]` requires the feature name)
- Claude asks proactively about CHECKPOINT so the user doesn't have to remember the keyword while focused on code
- Git operations are always explained — the user should understand what's happening, not just that it happened
- Every file is plain markdown — readable by a human, parseable by Claude, no tooling required
- The system is portable — move the `Workflow/` folder to any machine, point Claude at it, and it works
- Works with any project and any tech stack — the framework is project-agnostic
