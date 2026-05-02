Create a new Git branch for a feature, starting from an up-to-date main or develop branch. The branch name is: $ARGUMENTS

Follow these steps exactly:

## Step 1 — Check if a branch name was provided

If $ARGUMENTS is empty, ask the user:
> "What would you like to call this branch? Branch names should describe the feature — for example:
> - `feature/user-authentication`
> - `feature/dashboard-stats`
> - `fix/login-cors-error`
>
> The name becomes part of your Git history, so make it readable. What are you building?"

Wait for their answer and use it as the branch name. If the name they give has spaces, replace spaces with hyphens automatically.

If $ARGUMENTS is provided, use it directly (replacing spaces with hyphens).

## Step 2 — Explain branches before touching anything

Tell the user:
> "Before I create the branch, here's what's about to happen and why:
>
> A **branch** is your own isolated copy of the codebase to work in. When you create a branch, you're saying: 'I'm going to build this feature here, separate from the main codebase, and only merge it in when it's ready and reviewed.'
>
> This means:
> - You can break things without affecting anyone else
> - Your work-in-progress is invisible to collaborators until you open a Pull Request
> - If the feature goes wrong, you can throw the branch away and start fresh — main is untouched
>
> I'm going to switch to main/develop first, pull the latest changes, then create your branch from that clean starting point."

## Step 3 — Find the base branch

```bash
git branch -a 2>&1
```

Check if `develop` exists:
```bash
git ls-remote --heads origin develop 2>/dev/null && echo "develop exists"
```

If `develop` exists, use it as the base. Otherwise use `main`. If neither exists, check for `master`.

Set BASE_BRANCH accordingly.

## Step 4 — Switch to base branch and pull latest

Tell the user:
> "Switching to **BASE_BRANCH** and pulling the latest changes from GitHub — this makes sure your new branch starts from the most up-to-date version of the project."

```bash
git checkout BASE_BRANCH 2>&1
git pull origin BASE_BRANCH 2>&1
```

If there are merge conflicts or the pull fails, stop and tell the user what happened. Do not proceed with a dirty or conflicted state.

If there are uncommitted changes preventing the checkout:
> "You have uncommitted changes that would be lost when switching branches. Let me help you deal with them first."

Options to present:
1. Commit them now (`/commit`)
2. Stash them temporarily: `git stash`

Wait for their decision before continuing.

## Step 5 — Create and switch to the new branch

Tell the user:
> "Creating your branch **[branch-name]** from BASE_BRANCH. This is now your workspace — any code you write goes here, not into BASE_BRANCH."

```bash
git checkout -b [branch-name] 2>&1
```

Verify:
```bash
git branch --show-current 2>&1
```

## Step 6 — Confirm and connect to framework

Tell the user:
> "You're now on branch **[branch-name]**. Here's your current status:
> - Base: BASE_BRANCH (up to date)
> - New branch: **[branch-name]**
> - Clean working tree — ready to build
>
> **What to do from here:**
> 1. Write your code
> 2. Run `/commit` whenever you finish a meaningful chunk of work — don't wait until the end
> 3. Run `/push` when you're ready to share it or want it backed up on GitHub
> 4. When the feature is done, `/push` will prompt you to open a Pull Request for review
>
> If you haven't started a feature in Edo's Framework yet, now's a good time — run `/new-feature [feature-name]` to set up the tracking file."
