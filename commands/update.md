Fetch and pull the latest changes to Edo's Framework from GitHub.

Follow these steps exactly.

---

## Step 1 — Navigate to the framework repo

```bash
cd "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework" 2>&1
```

---

## Step 2 — Check for new commits before pulling

```bash
git fetch origin main 2>&1
git log HEAD..origin/main --oneline 2>&1
```

If the log is empty, tell the user:
> "Already up to date — no new changes on the framework."

Stop here.

---

## Step 3 — Pull the changes

```bash
git pull origin main 2>&1
```

---

## Step 4 — Report what's new

Read the commits that just came in and summarise them for the user:

> "Framework updated. Here's what's new:
>
> [for each commit, one bullet: commit message — plain English summary of what it means]"

Keep it conversational. If a commit message says "Add /merge command and remove personal project references" then write: `- New /merge command added` and `- Personal project names removed from public docs`. Translate commit messages into what actually changed for the user, not raw git output.
