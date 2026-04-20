Sync and push all changes to the edos-claude-framework GitHub repo.

The repo at `C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework` IS the working directory.
`EdosFramework.md`, `projects/`, and `commands/` are edited directly inside it — no separate source files to copy.

Follow these steps exactly.

---

## Step 1 — Sync commands from .claude

Only the commands need to be copied in (they live in `.claude/commands/`, not in the repo):

```bash
cp "C:/Users/Ednmh/.claude/commands/"*.md \
   "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/commands/"
```

## Step 2 — Check what changed

```bash
cd "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework" && git diff --stat && git status --short
```

If there are no changes (working tree clean after sync), print:
> "Nothing to commit — repo is already up to date."
And stop here.

## Step 3 — Commit

Build a commit message from the actual diff. Check which files changed:
- If only `EdosFramework.md` changed → "Update EdosFramework.md"
- If only `projects/` changed → "Update projects: [list changed project names]"
- If only skill files changed → "Update commands: [list the changed files without path or extension, comma-separated]"
- If multiple areas changed → combine: "Update EdosFramework.md, projects: [...], commands: [...]"
- Add a one-line body describing what was added/changed if it's obvious from the diff

```bash
cd "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework" && \
git add EdosFramework.md commands/ projects/ && \
git commit -m "$(cat <<'EOF'
[COMMIT MESSAGE HERE]

Co-Authored-By: Claude Sonnet 4.6 <noreply@anthropic.com>
EOF
)"
```

## Step 4 — Push (with SSH agent)

```bash
eval "$(ssh-agent -s)" 2>/dev/null
ssh-add "C:/Users/Ednmh/.ssh/id_ed25519_github" 2>/dev/null
cd "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework" && git push origin main 2>&1
```

## Step 5 — Confirm

Print a one-line summary:
> "Pushed to edos-claude-framework: [commit message first line]"
