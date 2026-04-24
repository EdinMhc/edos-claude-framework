Merge the current feature branch into the main branch (main, master, or develop).

This is a significant, semi-destructive action — explain every step before executing it, and always get confirmation before merging.

---

## Step 1 — Establish context

Run:
```bash
git branch --show-current 2>&1
git status 2>&1
```

If the user is already on `main`, `master`, or `develop`, stop and tell them:
> "You're already on **[branch]** — there's nothing to merge. Switch to a feature branch first, then run `/merge`."

Store the feature branch name as FEATURE_BRANCH for the rest of this skill.

Detect the target branch (in order of preference):
```bash
git ls-remote --heads origin main 2>/dev/null && echo "main"
git ls-remote --heads origin master 2>/dev/null && echo "master"
git ls-remote --heads origin develop 2>/dev/null && echo "develop"
```

Use the first one that exists. Store it as TARGET_BRANCH.

---

## Step 2 — Check for uncommitted changes

If `git status` shows any modified, staged, or untracked files, stop and tell the user:

> "Before we merge, you have uncommitted changes on **[FEATURE_BRANCH]**. These need to be committed first — otherwise they won't be included in the merge and could get lost or cause conflicts.
>
> A merge packages everything that's been committed on your branch and brings it into **[TARGET_BRANCH]**. Anything not committed stays behind.
>
> Want me to run `/commit` now so everything is saved before we merge?"

Wait for their answer. If yes, invoke the commit skill. If no, stop — do not proceed with uncommitted changes.

---

## Step 3 — Prompt to push the branch first

Check if the feature branch is ahead of its remote:
```bash
git log --oneline origin/$(git branch --show-current)..HEAD 2>/dev/null
```

If there are unpushed commits, tell the user:
> "Your branch has commits that haven't been pushed to GitHub yet. It's good practice to push before merging — it backs everything up and means the remote copy of your branch matches what you're about to merge.
>
> Want me to push **[FEATURE_BRANCH]** now before we continue?"

Wait for their answer. If yes, run:
```bash
git push 2>&1
```

If the branch has no remote tracking yet:
```bash
git push -u origin $(git branch --show-current) 2>&1
```

If the user says no, continue — this is their choice and the merge can still proceed.

---

## Step 4 — Explain what is about to happen

Before touching anything, give the user a clear picture:

> "Here's what's about to happen — I want you to understand each step before we run it:
>
> 1. **Switch to [TARGET_BRANCH]** — we need to be on the branch we're merging *into*
> 2. **Fetch latest from GitHub** — this pulls down any commits that teammates may have added to [TARGET_BRANCH] while you were working, so your local copy is current
> 3. **Merge [FEATURE_BRANCH] into [TARGET_BRANCH]** — your feature's commits get applied on top of the latest [TARGET_BRANCH]
> 4. **Push [TARGET_BRANCH]** — the merged result goes to GitHub so everyone has it
>
> One thing worth knowing: this is a **direct merge**. In most professional teams, this step goes through a Pull Request so a reviewer can approve the changes first. Direct merging is fine when you have full access to your own repo and don't need a review — but if you're working with a team, `/push` + opening a PR is usually the right path. Run `/git` for more on that workflow.
>
> Ready to merge **[FEATURE_BRANCH]** → **[TARGET_BRANCH]**?"

Wait for confirmation before proceeding.

---

## Step 5 — Switch to target branch and fetch latest

```bash
git checkout [TARGET_BRANCH] 2>&1
git fetch origin 2>&1
git merge origin/[TARGET_BRANCH] 2>&1
```

Tell the user:
> "Switched to **[TARGET_BRANCH]** and pulled the latest from GitHub. [TARGET_BRANCH] is now fully up to date — any commits your teammates pushed are now on your machine too."

---

## Step 6 — Merge the feature branch

```bash
git merge [FEATURE_BRANCH] 2>&1
```

---

## Step 7a — Merge succeeded (no conflicts)

If the merge exits cleanly, tell the user:
> "Merged! **[FEATURE_BRANCH]** is now part of **[TARGET_BRANCH]**. Your feature's commits are woven into the main codebase.
>
> Pushing [TARGET_BRANCH] to GitHub now so everyone gets the updated version..."

```bash
git push 2>&1
```

Confirm:
> "Done. **[TARGET_BRANCH]** is pushed. Anyone who pulls from GitHub will now have your feature included.
>
> Next steps:
> - Run `/wrap-up [feature name]` to close the framework record for this feature
> - Run `/send-report` if you made backend changes that other developers depend on
> - You can delete the feature branch if you no longer need it — want me to do that?"

If they say yes to deleting the branch:
```bash
git branch -d [FEATURE_BRANCH] 2>&1
git push origin --delete [FEATURE_BRANCH] 2>&1
```

Explain:
> "Deleted **[FEATURE_BRANCH]** locally and on GitHub. The commits are still in [TARGET_BRANCH] — deleting a branch just removes the label, not the history."

---

## Step 7b — Merge conflicts detected

If the merge output contains "CONFLICT", stop immediately and explain:

> "We hit a **merge conflict** on **[FEATURE_BRANCH]** → **[TARGET_BRANCH]**. This means the same part of a file was changed differently on both branches, and Git doesn't know which version to keep — it needs a human to decide.
>
> This is completely normal and happens in every real project. Here's what it means:
> - Git has paused the merge partway through
> - The conflicting files are marked with `<<<<<<`, `=======`, and `>>>>>>>` markers showing both versions
> - You need to edit those files, pick the correct version (or combine them), remove the markers, then commit
>
> **Your options:**
> 1. **Resolve it now** — I'll walk you through each conflicted file one by one
> 2. **Abort the merge** — go back to where you were and figure it out with more context
> 3. **Send a report** — if these conflicts involve code written by another developer, run `/send-report` to notify them and get their input before resolving
>
> What would you like to do?"

Show the conflicted files:
```bash
git diff --name-only --diff-filter=U 2>&1
```

**If the user chooses option 1 (resolve now):**

For each conflicted file, read it and show the user the conflict sections. Explain:
> "In **[filename]**, here's the conflict:
> - The `<<<<<<< HEAD` section is what's currently on **[TARGET_BRANCH]**
> - The `>>>>>>> [FEATURE_BRANCH]` section is what's on your feature branch
> - The `=======` line divides them
>
> Which version is correct — the one from [TARGET_BRANCH], the one from [FEATURE_BRANCH], or a combination of both?"

Wait for the user's decision, apply it, save the file. Repeat for each conflict.

Once all conflicts are resolved:
```bash
git add . 2>&1
git commit -m "Merge [FEATURE_BRANCH] into [TARGET_BRANCH] — resolve conflicts" 2>&1
git push 2>&1
```

**If the user chooses option 2 (abort):**
```bash
git merge --abort 2>&1
git checkout [FEATURE_BRANCH] 2>&1
```
> "Merge aborted. You're back on **[FEATURE_BRANCH]** exactly as you left it — nothing was changed. Take a look at the conflicting files, make your changes there, commit, and then run `/merge` again when you're ready."

**If the user chooses option 3 (send report):**
```bash
git merge --abort 2>&1
git checkout [FEATURE_BRANCH] 2>&1
```
> "Merge aborted so nothing is broken. Run `/send-report` to notify your collaborator about the conflicts — include which files are affected and what your changes were doing. Once you've aligned on how to resolve it, come back and run `/merge` again."
