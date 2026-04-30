Show a dashboard of all projects in Edo's Framework — what exists, what's active, and where each one left off.

Follow these steps exactly.

---

## Step 1 — Detect framework root

Run:
```bash
for candidate in \
    "$USERPROFILE/OneDrive/Desktop/Workflow/edos-claude-framework" \
    "$HOME/OneDrive/Desktop/Workflow/edos-claude-framework" \
    "$HOME/Desktop/Workflow/edos-claude-framework" \
    "$HOME/Workflow/edos-claude-framework"; do
    [ -d "$candidate/projects" ] && echo "$candidate" && break
done
```

Store as FRAMEWORK_ROOT. If nothing is returned:
> "Could not locate Edo's Framework. Make sure it's installed in your Workflow folder."
Stop.

Set PROJECTS_PATH = FRAMEWORK_ROOT/projects

---

## Step 2 — Scan all projects

```bash
ls "PROJECTS_PATH/" 2>&1
```

If no projects exist:
> "No projects found. Create your first one with `/project [name]`."
Stop.

---

## Step 3 — Read each project's state

For each project folder found, read:
1. `PROJECTS_PATH/[ProjectName]/ACTIVE.md` — to get the active feature name
2. If an active feature exists, read the last bullet from **Session Notes** in the feature file at `PROJECTS_PATH/[ProjectName]/features/[FeatureName].md`
3. Count the number of files in `PROJECTS_PATH/[ProjectName]/features/` to get total feature count (including done ones)

---

## Step 4 — Print the dashboard

Print:
```
╔══════════════════════════════════════════════════════════════════╗
║                        PROJECT STATUS                           ║
╚══════════════════════════════════════════════════════════════════╝
```

For each project, print a block like this:

```
  ┌─────────────────────────────────────────────────────────────┐
  │  [ProjectName]                                              │
  │                                                             │
  │  Active feature:  [FeatureName] — or — (none)              │
  │  Last session:    [last session note, truncated to 80 chars]│
  │  Features total:  [N] (including completed)                 │
  └─────────────────────────────────────────────────────────────┘
```

If there is no active feature, show "(none)" and omit the last session line.

After all project blocks, print:
```
  ──────────────────────────────────────────────────────────────
  Run /active to load a project and resume work.
  Run /project [name] to start a new one.
  ──────────────────────────────────────────────────────────────
```
