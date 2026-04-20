Explain Git to the user — how it works, why it matters, and how to use it correctly in a real development workflow.

Read the current git state first so the explanation is grounded in where the user actually is:

```bash
git status 2>&1
git branch 2>&1
git log --oneline -5 2>&1
```

Then deliver this explanation, adapting it to what you find (e.g. if they're on main with no branches, emphasise branching first):

---

**What Git is and why you need it**

Git is a version control system — it tracks every change you make to your code over time. Think of it like a save system in a video game, except every save has a description and you can jump back to any previous save at any time. It also lets multiple people work on the same codebase without overwriting each other's work.

Without Git: if something breaks, you have no way back. With Git: any mistake is recoverable.

---

**The three things you do in Git**

**1. Branches — working in isolation**

A branch is your own copy of the codebase to work in. When you start a new feature, you create a branch so your changes are completely isolated from the main codebase. Nobody else sees your work-in-progress, and you can't accidentally break what's already working.

The main branch (usually called `main` or `master`) is the "official" version of the project — the one that's deployed or shared. You never work directly on it. You branch off it, build your feature, then merge it back when it's done and tested.

```
main  ──────────────────────────────── (stable, always working)
          \
           feature/my-feature ─────── (your isolated workspace)
```

To create a branch: use `/branch [feature-name]` — Claude will set it up correctly.

**2. Commits — saving your progress**

A commit is a snapshot of your work at a specific point. Every commit has a message that describes what changed and why. Good commit messages make your history readable — you can look back at any commit and understand exactly what was done.

**Rule of thumb:** commit whenever you finish a logical unit of work — a method added, a bug fixed, a feature wired up. Small, frequent commits are better than one giant commit at the end.

To commit: use `/commit` — Claude will inspect what changed and write a meaningful message.

**3. Push — sending your work to the server**

A push uploads your local commits to GitHub (the remote server). Until you push, your commits only exist on your machine. Pushing regularly means your work is backed up and your collaborators can see it.

To push: use `/push`.

---

**The full workflow for a feature**

```
1. /branch feature/my-feature    ← create your workspace
2. Write code
3. /commit                        ← save a snapshot with a good message
4. Write more code
5. /commit                        ← save again
6. /push                          ← send to GitHub
7. Open a Pull Request            ← ask for a code review before merging
8. Get reviewed, fix feedback     ← collaboration step
9. Merge into main                ← feature is officially done
10. /wrap-up [feature-name]       ← close the framework record
```

---

**Pull Requests — why code review matters**

When your branch is done, you don't merge it yourself. You open a Pull Request (PR) on GitHub — this is a formal request to merge your branch into main. The PR shows every change you made, and a collaborator (in this project: **EdinMhc**) reviews it before it goes in.

Code review exists because:
- A second pair of eyes catches bugs you missed
- It keeps the codebase style consistent
- It documents *why* a change was made for future developers

When you `/push`, Claude will prompt you to create a PR and tag EdinMhc as the reviewer.

---

**Merge vs Rebase — the two ways to combine branches**

- **Merge** keeps the full history of both branches and adds a "merge commit". Safer and easier to understand.
- **Rebase** rewrites your branch history on top of the target branch, giving a cleaner linear history. More powerful but riskier if you've already pushed.

For beginners: always use **merge**. Claude will default to this.

---

**Your available Git skills**

| Skill | What it does |
|---|---|
| `/branch [name]` | Create a new feature branch from up-to-date main/develop |
| `/commit` | Inspect changes and commit with a meaningful message |
| `/push` | Push current branch to GitHub and prompt for a PR |
| `/git` | This explanation — run it any time you need a refresher |

---

**Current state of your repo:**

Claude will now describe what it found when it read your git status and branch list at the top, and suggest the most useful next step based on where you are.
