Show a quick reference of all available commands in Edo's Framework.

Print the following exactly:

```
╔══════════════════════════════════════════════════════════════════╗
║                  EDO'S FRAMEWORK — COMMAND REFERENCE            ║
╚══════════════════════════════════════════════════════════════════╝

  ── SETUP ──────────────────────────────────────────────────────────

  /activate          First-time setup. Installs everything, walks you
                     through creating your first project, and runs the
                     onboarding tutorial. Run once per machine.

  /project [Name]    Register a project. Creates the tracking folder
                     for a new or existing project. Run once per project.
                     Example: /project MyApp
                     Example: /project MyApp - A task tracker for teams

  /update            Pull the latest version of the framework from GitHub.

  ── DAILY USE ──────────────────────────────────────────────────────

  /active            Load context for the current session. Run this
                     FIRST every time you open a new chat. Claude will
                     know exactly what you were working on and where
                     you left off.

  /new-feature       Start tracking a new piece of work. Creates a
    [Name]           file that Claude uses as its memory for this
                     feature across sessions.
                     Example: /new-feature Login Page

  /gather-context    Deep questioning mode. Claude asks questions about
                     what you're building and why before any code is
                     written. Recommended before every new feature.

  /analyze           Scan the codebase and produce a phased
                     implementation plan. Run after /gather-context.

  /checkpoint        Save your progress. Updates the feature file with
                     what was done, which files were touched, and any
                     decisions made. Run after any meaningful block of
                     work.

  /status            See all your projects and what's active in each.

  ── GIT & CODE ─────────────────────────────────────────────────────

  /branch [Name]     Create a new Git branch for a feature.
                     Example: /branch feature/login-page

  /commit            Save a snapshot of your current changes with a
                     meaningful message. Claude explains what's being
                     saved.

  /push              Back up your code to GitHub. Claude will also
                     offer to open a Pull Request.

  /merge             Merge your feature branch into main. Claude
                     handles conflicts step by step.

  /git               Learn how Git works — branches, commits, pushes,
                     pull requests. Run this once if you're new to Git.

  ── CLOSING OUT ────────────────────────────────────────────────────

  /wrap-up [Name]    Close a completed feature. Fills in the full
                     record, marks it DONE, and clears the active
                     feature. Example: /wrap-up Login Page

  /send-report       Email a change report to your teammates after
                     completing a backend feature.

  /archive-feature   Move a feature file to the archive without
    [Name]           deleting it.

  /delete-project    Remove a project's tracking folder from the
    [Name]           framework. Does NOT delete your actual code.

  ── DATABASE ───────────────────────────────────────────────────────

  /connect-db        Connect Claude to your SQL Server database for
                     direct query access.

  ──────────────────────────────────────────────────────────────────
  Tip: Start every new chat with /active — it's the most important
  habit in the whole framework.
  ──────────────────────────────────────────────────────────────────
```
