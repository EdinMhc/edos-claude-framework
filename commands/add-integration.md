Add an integration to the current project. An integration connects an external service (Gmail, Shopify, Google Analytics, Google Sheets, etc.) to Claude so you can do your daily work through Claude rather than visiting each website manually.

Follow these steps exactly.

---

## Step 1 — Detect the project

Detect FRAMEWORK_ROOT using the standard detection block:
```bash
for candidate in \
    "$USERPROFILE/OneDrive/Desktop/Workflow/edos-claude-framework" \
    "$HOME/OneDrive/Desktop/Workflow/edos-claude-framework" \
    "$HOME/Desktop/Workflow/edos-claude-framework" \
    "$HOME/Workflow/edos-claude-framework"; do
    [ -d "$candidate/projects" ] && echo "$candidate" && break
done
```

Determine PROJECT_NAME using smart detection (current working directory folder name vs projects list). If multiple projects and no match, ask the user which project to add the integration to.

Set INTEGRATIONS_PATH = `FRAMEWORK_ROOT/projects/[PROJECT_NAME]/Integrations.md`

---

## Step 2 — Show existing integrations (if any)

Check if `INTEGRATIONS_PATH` exists.

If it does, read it and list the existing integrations by name:
> "You already have [N] integration(s) set up for **[PROJECT_NAME]**:
> [list of integration names]
>
> Would you like to add a new one, or update an existing one?"

Wait for the user's answer. If they want to update an existing one, skip to Step 5 with that integration name.

If `INTEGRATIONS_PATH` does not exist, continue to Step 3.

---

## Step 3 — Ask what the user wants to integrate

> "Which service or website would you like to connect?
>
> You can name as many as you want at once. For example:
> - Gmail
> - Shopify
> - Google Analytics
> - Google Sheets
> - Notion, Slack, Stripe, GitHub, or any other tool you use daily
>
> Just tell me the names or paste the URLs — I'll research what's possible and what you'll need to set up."

Wait for the user's answer. Store as a list of SERVICE_NAMES.

---

## Step 4 — Research each service

For each service in SERVICE_NAMES:

**Search for:**
- `[service name] API documentation`
- `[service name] OAuth setup guide`
- `[service name] developer credentials how to get`
- What actions are available (read, write, send, update, etc.)

Then present a summary to the user:

> "Here's what I found for **[Service Name]**:
>
> **What Claude can help you do:**
> [bullet list of concrete automatable actions — be specific, e.g. "Draft and send emails", "Read your inbox and summarise unread messages", "Create calendar events"]
>
> **What you'll need to set up:**
> [bullet list of credentials/access required — e.g. "A Gmail App Password", "An OAuth 2.0 token with Gmail send scope", "An API key from your Shopify admin panel"]
>
> **Setup difficulty:** [Easy / Medium / Advanced]
>
> Does this match what you're trying to do? Any specific tasks you had in mind that I haven't mentioned?"

Wait for the user to confirm or clarify before proceeding to setup.

---

## Step 5 — Guide setup step by step

For each service, walk the user through the exact setup steps based on what you found in Step 4.

**Rules for guidance:**
- Assume zero technical knowledge — explain every step as if the user has never seen an API key before
- Tell the user exactly where to click, what page to go to, and what to copy
- If a step involves going to an external website, say: "Go to [URL] and [exact action]"
- If a step generates a key or token, say: "Copy the value that appears — you won't be able to see it again after you leave this page"
- After each step, wait for the user to confirm they've completed it before moving to the next

**When credentials are collected**, confirm what was gathered:
> "Got it. Here's what I've collected for [Service]:
> [list the credential names — do NOT echo back the actual secret values]
>
> Ready to save this integration?"

---

## Step 6 — Save to Integrations.md

Once the user confirms, write or update `INTEGRATIONS_PATH`.

**File format:**

```markdown
# Integrations — [PROJECT_NAME]
*This file is local only — never committed to Git. Contains credentials and integration context for this project.*

---

## [Service Name]
**Added:** [date]
**Used for:** [one sentence — what this integration does for this specific project]

### Credentials
[credential name]: [value]
[credential name]: [value]

### What Claude can help with
- [specific action 1]
- [specific action 2]
- [specific action 3]

### Usage notes
[Any specific notes about how to use this integration, quirks, rate limits, etc.]

---
```

If the file already exists, append the new integration section after the last `---` separator. Do not overwrite existing integrations.

After writing, confirm:
> "Integration saved. **[Service Name]** is now set up for **[PROJECT_NAME]**.
>
> Next time you load this project with `/active`, I'll have this context loaded and ready — you can ask me to help you do things in [Service Name] without explaining the setup again.
>
> [If more services remain in the list]: Ready to set up **[next service]**? Or would you like to stop here and continue later with `/add-integration`?"

---

## Step 7 — Suggest first use

After all integrations are saved, offer an immediate example of what Claude can now do:

> "Now that [Service Name] is connected, here are some things you can ask me to do right now:
> [2–3 concrete, specific examples based on what was set up]
>
> Just describe what you want in plain language — you don't need to know any commands."
