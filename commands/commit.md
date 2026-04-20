Commit the current changes with a meaningful, descriptive commit message. Explain what you're doing and why as you go.

Follow these steps exactly:

## Step 1 — Check what's changed

Run:
```bash
git status 2>&1
git diff --stat 2>&1
git diff 2>&1 | head -150
```

If there is nothing to commit (working tree clean), tell the user:
> "There's nothing to commit right now — your working tree is clean. All your changes are already saved in a previous commit."
Then stop.

## Step 2 — Explain what you found

Before committing anything, tell the user exactly what changed. For example:
> "Here's what I can see has changed since your last commit:
> - `src/api/client.ts` — modified (new auth interceptor added)
> - `src/pages/LoginPage.tsx` — modified (form validation wired up)
> - `src/types/auth.ts` — new file (login request and token types)
>
> This is a solid chunk of work — the authentication flow is complete. That's a good reason to commit now: it captures this feature at a working state so you can always come back to it if something breaks later."

Adapt this to what you actually found. Always explain WHY committing now makes sense.

## Step 3 — Write a meaningful commit message

Write a commit message that:
- Starts with a short title (under 72 characters) describing WHAT was done
- Uses present tense imperative ("Add login form validation" not "Added" or "Adding")
- Is specific — names the feature or area of code ("Wire up JWT auth interceptor" not "Update files")
- If there are multiple distinct changes, the title covers the main one and the body lists the rest

Examples of good messages:
```
Add JWT auth interceptor with automatic token refresh
Wire up login form with Zod validation and error handling
Fix CORS issue by pointing FE at HTTPS backend URL
Add dashboard stats query with React Query caching
```

Examples of bad messages (never use these):
```
fix
update
changes
WIP
misc updates
```

## Step 4 — Show the user and confirm

Tell the user the commit message you've chosen:
> "I'm going to commit with this message:
> **[your message here]**
> Does that look right, or would you like to change it?"

Wait for their confirmation. If they want a different message, use theirs.

## Step 5 — Stage and commit

Stage all modified and new tracked files:
```bash
git add -A
```

Commit with the agreed message:
```bash
git commit -m "[message]"
```

## Step 6 — Confirm and nudge toward pushing

Tell the user:
> "Committed! Your snapshot is saved locally.
>
> One thing to know: this commit only exists on **your machine** right now. Until you push it, GitHub doesn't have it. Want me to push it now? (Just say 'push it' or run `/push`)"

This is the moment to teach the difference between a local commit and a remote push — do it naturally, not as a lecture.
