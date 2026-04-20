Sync and push all changes to the edos-claude-framework GitHub repo.

This skill keeps the repo in sync with the local working files:
- `C:/Users/Ednmh/OneDrive/Desktop/Workflow/EdosFramework.md` → repo
- `C:/Users/Ednmh/.claude/commands/*.md` → repo `commands/`

Follow these steps exactly.

---

## Step 1 — Sync local files into the repo

```bash
cp "C:/Users/Ednmh/OneDrive/Desktop/Workflow/EdosFramework.md" \
   "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework/EdosFramework.md"

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
- If only skill files changed → "Update commands: [list the changed files without path or extension, comma-separated]"
- If both changed → "Update EdosFramework.md and commands: [list]"
- Add a one-line body describing what was added/changed if it's obvious from the diff

```bash
cd "C:/Users/Ednmh/OneDrive/Desktop/Workflow/edos-claude-framework" && \
git add EdosFramework.md commands/ && \
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
