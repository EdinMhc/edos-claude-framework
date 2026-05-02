Send a backend change report email. Follow these steps exactly in order.

---

## Step 1 — Check / create config

Check if `~/.claude/send-report-config.json` exists:
```bash
cat "$HOME/.claude/send-report-config.json" 2>/dev/null
```

If the file does NOT exist or is empty, this is first-time setup. Tell the user:
> "First-time setup for /send-report. I need two things to configure this:"

Then ask these one at a time and wait for each answer:
1. **Sender Gmail address** — the Gmail you'll send reports FROM (e.g. you@gmail.com)
2. **Gmail App Password** — go to myaccount.google.com → Security → 2-Step Verification → App Passwords → create one named "Claude Reports". It looks like: `xxxx xxxx xxxx xxxx`

Once you have both, write them to `~/.claude/send-report-config.json`:
```json
{
  "from": "<sender email>",
  "appPassword": "<app password>"
}
```

Confirm: "Sender config saved. You won't be asked again on this device."

If the file DOES exist, read it silently and continue.

---

## Step 1b — Check / create contacts

Check if `~/.claude/contacts.json` exists:
```bash
cat "$HOME/.claude/contacts.json" 2>/dev/null
```

**If the file does NOT exist or is empty:**

Tell the user:
> "I'll also need to know who to send reports to. I'll save your contacts locally so you never have to type an email address again."

Ask:
> "What's the email address of the first person you send reports to?"

Wait for their answer. Then try to extract a name from the email string (e.g. `john.doe@company.com` → "John Doe", `sarahk@example.com` → "Sarahk"). Show the user what you extracted and ask:
> "I'll save them as **[extracted name]** — does that look right, or would you like a different name for this contact?"

Wait for their answer. Use whichever name they confirm.

Then ask:
> "Is there anyone else you regularly send reports to? If so, give me their email and I'll add them now. If not, just say no."

Repeat the name extraction + confirmation for each additional person they give. Keep asking until they say no or they're done.

Once all contacts are collected, write them to `~/.claude/contacts.json`:
```json
[
  { "name": "John Doe", "email": "john.doe@company.com" },
  { "name": "Sarah", "email": "sarah@example.com" }
]
```

Tell the user:
> "Contacts saved. You can always add more people later — just tell me when sending a report."

**If the file DOES exist:** Read it silently. The contacts list is now available for this session.

---

## Step 2 — Check / install nodemailer

Run:
```bash
node -e "require('nodemailer'); console.log('ok')" 2>&1
```

If output is NOT "ok" (module not found), install it:
```bash
npm install -g nodemailer
```

Wait for completion before continuing.

---

## Step 3 — Gather git changes

Run all of these and collect the output:

```bash
git status --short 2>&1
git log --oneline -15 2>&1
git diff --stat HEAD 2>&1
git diff HEAD 2>&1 | head -200
```

Also run to understand the project context:
```bash
git log --oneline -1 2>&1
ls *.sln *.csproj package.json 2>/dev/null | head -5
```

If not in a git repo, list recently modified source files:
```bash
find . -type f \( -name "*.cs" -o -name "*.ts" -o -name "*.tsx" -o -name "*.js" \) -newer . -not -path "*/bin/*" -not -path "*/obj/*" -not -path "*/node_modules/*" 2>/dev/null | head -30
```

---

## Step 4 — Compose the email

Using everything you gathered, write a developer-focused report email:

**Subject line**: A concise title describing the work done. Derive it from recent commit messages and changed files. Example: `Backend: Invoice PDF generation + S3 upload integration ready for FE`

**HTML body** with these four sections:

1. **Summary** — 3–5 sentences. What was built, why it exists, what it enables. Written for a frontend developer, not an ops person.

2. **Files Changed** — a clean list. For each file, one line explaining what changed. Group by feature area if there are many files. Skip lock files, migrations, and generated files.

3. **New / Changed API Contracts** — if any endpoints were added or modified, list them:
   - Method + route
   - What it does
   - Key request/response fields the FE needs to know about

4. **What You Can Do Now** — concrete next steps for the FE dev. What can they wire up, what endpoints to call, what UI to build against.

---

## Step 5 — Pick recipients and show draft

**Select recipients:**

Read the contacts from `~/.claude/contacts.json`. If there is only one contact, use them automatically. If there are multiple, show the list and ask:
> "Who should this report go to? (You can pick more than one)"

List contacts by name, e.g.:
> 1. John Doe
> 2. Sarah K
> 3. Everyone

Wait for their answer. Resolve the selection to one or more email addresses.

If the user names someone not in the contacts list, ask for their email, offer to save them as a new contact, and if yes append them to `~/.claude/contacts.json`.

**Show the draft:**

Display the full draft to the user exactly like this:

```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 DRAFT REPORT EMAIL
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
 TO:      <Name (email), Name (email), ...>
 FROM:    <sender>
 SUBJECT: <subject>
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

<body as plain text>

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Then ask: **"Ready to send, or would you like to change anything before it goes out?"**

If the user wants changes: apply them, show the updated draft again, ask again. Repeat until they say yes / send it / looks good.

---

## Step 6 — Send the email

Read the config from `~/.claude/send-report-config.json`.

The TO field is a comma-separated string of all selected recipient email addresses.

Write this script to `$HOME/.claude/send-report-tmp.js`, substituting the actual values for FROM, APP_PASSWORD, TO (comma-separated emails), SUBJECT, and BODY (escape backticks and `$` in the body):

```js
const nodemailer = require('nodemailer');

const transporter = nodemailer.createTransport({
  service: 'gmail',
  auth: {
    user: 'FROM_PLACEHOLDER',
    pass: 'PASS_PLACEHOLDER'
  }
});

transporter.sendMail({
  from: '"Dev Reports" <FROM_PLACEHOLDER>',
  to: 'TO_PLACEHOLDER',
  subject: 'SUBJECT_PLACEHOLDER',
  html: `BODY_PLACEHOLDER`
})
.then(() => { console.log('SUCCESS'); process.exit(0); })
.catch(e => { console.error('ERROR: ' + e.message); process.exit(1); });
```

Run it:
```bash
node "$HOME/.claude/send-report-tmp.js"
```

Then delete it:
```bash
rm "$HOME/.claude/send-report-tmp.js"
```

If output is `SUCCESS`: tell the user "Report sent to [list recipient names]."
If output contains `ERROR`: show the error message and suggest checking the App Password or Gmail settings.

---

## Notes for new devices

On a new machine where nodemailer is not installed and config does not exist, Steps 1 and 2 handle everything automatically. The user only needs to provide credentials once per device.
