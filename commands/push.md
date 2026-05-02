Push the current branch to GitHub and guide the user through opening a Pull Request.

Follow these steps exactly:

## Step 1 — Check state before pushing

Run:
```bash
git status 2>&1
git branch --show-current 2>&1
git log --oneline origin/$(git branch --show-current 2>/dev/null)..HEAD 2>/dev/null || git log --oneline -5 2>&1
```

If there are uncommitted changes, tell the user:
> "You have uncommitted changes. I'd recommend committing them first before pushing — otherwise they won't be included. Want me to run `/commit` first?"

Wait for their answer. If yes, run the commit skill first. If no, continue.

## Step 2 — Explain what pushing means (first-time context)

Before pushing, give a one-line explanation:
> "Pushing sends your local commits to GitHub. Right now your commits only exist on this machine — after this, they'll be on the server and backed up."

Keep it short. Don't over-explain if they've heard it before.

## Step 3 — Check if this branch exists on remote

```bash
git ls-remote --heads origin $(git branch --show-current) 2>&1
```

If the branch does NOT exist on remote yet, push with upstream tracking:
```bash
git push -u origin $(git branch --show-current) 2>&1
```

If it already exists:
```bash
git push 2>&1
```

If push fails due to diverged history, tell the user:
> "The remote branch has changes that aren't on your machine. This usually means someone else pushed to this branch, or you pushed from a different machine. Run `/git` for an explanation of how to handle this, or tell me what happened and I'll guide you through it."

Do not force-push without explicit user instruction.

## Step 4 — Confirm push succeeded

If push succeeded, tell the user:
> "Pushed! Your commits are now on GitHub. Branch: **[branch name]**"

## Step 5 — Prompt for Pull Request

Check what branch they're on:
```bash
git branch --show-current 2>&1
```

If they're on `main` or `master` (pushed directly to main), skip the PR prompt and warn:
> "Note: you pushed directly to main. For future features, it's better to work on a feature branch and open a PR — that way your changes get reviewed before they're merged. Run `/git` to learn more about branching."

If they're on a feature branch, say:
> "Now that your branch is on GitHub, the next step in a real development workflow is to open a **Pull Request** — this is how you ask for your changes to be reviewed and approved before they go into main.
>
> A Pull Request shows every change you made, lets collaborators leave comments, and gives you a chance to catch anything before it becomes permanent in the main codebase.
>
> Would you like me to open a PR now? I'll tag **EdinMhc** as the reviewer."

Wait for their answer.

## Step 6 — Create the Pull Request (if user agrees)

Check if `gh` (GitHub CLI) is installed:
```bash
gh --version 2>&1
```

If not installed, tell the user:
> "The GitHub CLI (`gh`) isn't installed — I'll need it to create the PR from here. Installing it now..."
```bash
winget install --id GitHub.cli -e 2>&1 || choco install gh -y 2>&1
```

Check if user is authenticated with gh:
```bash
gh auth status 2>&1
```

If not authenticated:
> "You need to log in to GitHub CLI first. I'll start the login flow — follow the prompts in your browser."
```bash
gh auth login
```

Once authenticated, get recent commits for the PR description:
```bash
git log --oneline origin/main..HEAD 2>/dev/null || git log --oneline -10 2>&1
git diff --stat origin/main..HEAD 2>/dev/null | head -20
```

Create the PR:
```bash
gh pr create \
  --title "[feature title derived from commits and branch name]" \
  --body "$(cat <<'EOF'
## What this PR does
[2-3 sentences describing what was built, derived from commit messages and changed files]

## Changes
[bullet list of key changes]

## How to test
[brief steps to verify the feature works]

🤖 Opened with Edo's Framework
EOF
)" \
  --reviewer EdinMhc \
  --base main 2>&1
```

Adapt `--base` to `develop` if that branch exists:
```bash
git ls-remote --heads origin develop 2>/dev/null && echo "develop exists"
```

## Step 7 — Confirm and explain what happens next

After the PR is created:
> "Pull Request opened and **EdinMhc** has been tagged as reviewer. Here's what happens next:
>
> 1. EdinMhc will review your changes and may leave comments or request changes
> 2. If changes are requested, you fix them locally, commit, and push again — the PR updates automatically
> 3. Once approved, the branch gets merged into main and your feature is officially part of the project
>
> While you wait for review, you can start a new feature on a new branch, or do a `/wrap-up` on the current one in Edo's Framework to close out the record."
