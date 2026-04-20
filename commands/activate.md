Set up Edo's Framework on this machine, walk the user through a project wizard, scaffold their app, and initialize everything for development.

Follow these steps exactly, in order. Do not skip steps. Do not rush — wait for user input at every question.

---

## PART A — WELCOME

Print this welcome banner in the terminal exactly as formatted:

```
╔══════════════════════════════════════════════════════════════════╗
║                                                                  ║
║              Welcome to  E D O ' S  F R A M E W O R K           ║
║                                                                  ║
║   Your guided coding internship — Claude is your senior dev.    ║
║                                                                  ║
╚══════════════════════════════════════════════════════════════════╝

  I'll walk you through everything step by step.
  There are no wrong answers — just tell me what you want to build
  and I'll handle the technical decisions and explain them as we go.

  Let's start with a few quick questions.
  ──────────────────────────────────────────────────────────────────
```

---

## PART B — ENVIRONMENT SETUP

### Step 1 — Locate or create the Workflow folder

Detect the Desktop path:
```bash
echo "$USERPROFILE/OneDrive/Desktop" 2>/dev/null || echo "$HOME/Desktop"
```

Check if a `Workflow` folder exists:
```bash
ls "$USERPROFILE/OneDrive/Desktop/Workflow" 2>/dev/null || ls "$HOME/Desktop/Workflow" 2>/dev/null
```

If found, set WORKFLOW_PATH silently and continue.

If NOT found, ask:
> "📁 I didn't find a **Workflow** folder on your Desktop. This is where the framework files will live. Should I create it there, or do you prefer a different location?"

Wait for answer. Create it:
```bash
mkdir -p "WORKFLOW_PATH"
mkdir -p "WORKFLOW_PATH/features"
```

### Step 2 — Check git is installed

```bash
git --version 2>&1
```

If not found:
> "⚠️  Git isn't installed yet. Git is the tool that saves your code history and lets you collaborate with others — it's essential. Let me open the download page for you."

```bash
start https://git-scm.com/download/win
```

Tell the user: "Install it with all the default options. Come back when it's done and tell me."
Wait for confirmation before continuing.

### Step 3 — Check git identity

```bash
git config --global user.name 2>&1
git config --global user.email 2>&1
```

If either is missing, ask:
> "Git needs a name and email for your commit history. What name and email should I use? (This is just metadata — use your real name and the email your GitHub account uses)"

Set them:
```bash
git config --global user.name "NAME"
git config --global user.email "EMAIL"
```

### Step 4 — Check GitHub SSH access

```bash
ssh -T git@github.com 2>&1 | head -3
```

If "successfully authenticated" → continue silently.

If not, explain SSH before doing anything:
> "🔑 You don't have a GitHub SSH key set up yet. Before I create one, let me explain what it actually is — because it's worth understanding.
>
> **SSH Key = a digital identity for your machine.**
> It works like a lock and key. Your computer gets a **private key** (a secret file that never leaves your machine) and a **public key** (a shareable copy you upload to GitHub). When you try to push code, GitHub checks: *"does this private key match the public key we have on file?"* If yes — you're in. No password ever needed.
>
> - **Why it's better than a password:** Passwords can be guessed, phished, or leaked. An SSH key is mathematically impossible to brute-force and never travels over the network.
> - **The private key** lives at `~/.ssh/id_ed25519_github` on your machine — never share it, never commit it to git.
> - **The public key** (the `.pub` file) is safe to share — that's what you paste into GitHub.
>
> I'll generate both now. This takes about 2 seconds."

```bash
ssh-keygen -t ed25519 -C "EMAIL" -f "$HOME/.ssh/id_ed25519_github" -N "" 2>&1
cat "$HOME/.ssh/id_ed25519_github.pub"
```

Tell the user:
> "☝️ That's your **public key** — the one that's safe to share. Copy everything from `ssh-ed25519` to the end of the line.
>
> Now go to **github.com → Settings → SSH and GPG keys → New SSH key**, give it any title (e.g. 'My Laptop'), paste it in, and save. Tell me when it's done."

Wait. Then:
```bash
eval "$(ssh-agent -s)" 2>&1
ssh-add "$HOME/.ssh/id_ed25519_github" 2>&1
ssh -T git@github.com 2>&1 | head -3
```

If successful, confirm: "✓ GitHub SSH connected. Your machine is now authenticated."

### Step 5 — Clone Edo's Framework

```bash
ls "WORKFLOW_PATH/edos-claude-framework" 2>/dev/null
```

If not found:
```bash
git clone https://github.com/EdinMhc/edos-claude-framework "WORKFLOW_PATH/edos-claude-framework"
```

If already there:
```bash
cd "WORKFLOW_PATH/edos-claude-framework" && git pull
```

Read it now — this is your operating manual for the rest of this session:
```
WORKFLOW_PATH/edos-claude-framework/EdosFramework.md
```

---

## PART C — NEW OR EXISTING PROJECT?

Print:
```
  ──────────────────────────────────────────────────────────────────
  STEP 1 OF 4 — New project or existing?
  ──────────────────────────────────────────────────────────────────
```

Ask:
> "Before we jump in — are you starting a brand new project, or do you have an **existing codebase** you'd like to work on?
>
> 1. **Starting fresh** — I'll scaffold everything from scratch
> 2. **Existing project** — I'll analyse what you have, give you a full report, and we'll plan what to do next"

Wait for answer.

- If **starting fresh** → skip to **PART C-NEW** below.
- If **existing project** → go to **PART C-EXISTING** below.

---

## PART C-EXISTING — Existing Project Analysis

### Step EX-1 — Get the project path

If the user already provided a path in their message, use it. Otherwise ask:
> "What's the path to your project? For example: `C:\Users\You\Desktop\MyProject` — you can also drag the folder into the terminal to get the path."

Wait for PROJECT_PATH.

If multiple projects (frontend + backend in separate folders), ask:
> "Is this the frontend, backend, or a single folder containing both?"

Collect all relevant paths. Store as PROJECT_PATH (and FRONTEND_PATH / BACKEND_PATH if separate).

### Step EX-2 — Silent project scan

**Do not ask questions yet. Silently scan the project.** Run these commands to gather information — adjust paths as needed:

```bash
# Structure overview
find "PROJECT_PATH" -maxdepth 3 -type f \( -name "*.csproj" -o -name "*.sln" -o -name "*.slnx" -o -name "package.json" -o -name "*.py" -o -name "composer.json" -o -name "go.mod" -o -name "Cargo.toml" \) 2>/dev/null | grep -v node_modules | grep -v bin | grep -v obj

# Git status
cd "PROJECT_PATH" && git log --oneline -10 2>/dev/null
cd "PROJECT_PATH" && git branch -a 2>/dev/null

# Package / dependency files
cat "PROJECT_PATH/package.json" 2>/dev/null
cat "PROJECT_PATH/*/package.json" 2>/dev/null
find "PROJECT_PATH" -name "*.csproj" -exec cat {} \; 2>/dev/null | grep -E "PackageReference|TargetFramework"

# Config / secrets surface
find "PROJECT_PATH" -maxdepth 4 -name "appsettings*.json" -o -name ".env*" -o -name "config.js" -o -name "config.ts" 2>/dev/null | grep -v node_modules
```

Also read key files: `Program.cs` / `Startup.cs` / `index.ts` / `App.tsx` / main entry points, plus any README or PLAN files at the root.

### Step EX-3 — Deep analysis (use extended thinking)

**Before writing the report, think thoroughly.** Use all available reasoning to assess:

1. **Architecture** — what pattern is used (MVC, Clean Architecture, monolith, microservices, none)? Is it consistent? Are there layer violations?
2. **Tech stack health** — is the stack modern or dated? Are packages significantly out of date? Is the language/framework the right tool for what this app does?
3. **Code quality signals** — naming consistency, separation of concerns, god classes, duplicated logic, missing abstractions
4. **Security surface** — look specifically for:
   - Secrets or credentials hardcoded in config files or source
   - JWT configuration issues (weak secrets, no expiry, symmetric keys in config)
   - CORS configured too broadly (`*` or `AllowAnyOrigin().AllowCredentials()`)
   - SQL injection risks (raw string queries)
   - Missing authentication/authorization on endpoints
   - Passwords stored or logged in plain text
   - User input passed directly to OS commands or file paths
   - Missing HTTPS enforcement
   - Sensitive data in logs
5. **Upgrade opportunities** — what would meaningfully improve this project? Is a framework migration warranted?
6. **Stack migration assessment** — if the stack is clearly wrong for the problem (e.g. plain PHP/HTML/JS for a data-heavy app, no framework at all, ancient tech with no security support), recommend a migration. Otherwise don't. Be honest and specific about the cost vs benefit.

### Step EX-4 — Present the full report

Print:
```
  ══════════════════════════════════════════════════════════════════
  PROJECT ANALYSIS REPORT
  ══════════════════════════════════════════════════════════════════
```

Then deliver the report in this exact structure:

---

**OVERVIEW**
- Project name / type
- Tech stack (language, framework, database, frontend)
- Architecture pattern detected
- Approximate size (files, lines of code estimate, number of endpoints if detectable)
- Git health (number of commits, branch structure, last activity)

---

**WHAT'S DONE WELL ✓**

List the genuine strengths. Be specific — don't just say "good structure", say what specifically is structured well and why it matters. If the project is actually well-built, say so clearly.

---

**SECURITY FINDINGS**

Rate each finding: 🔴 Critical · 🟡 Medium · 🟢 Low

For each finding:
- What it is
- Where it is (file + line if possible)
- Why it's a risk
- How to fix it (one-line fix or approach)

If there are no significant security issues, say so explicitly — don't invent problems.

---

**IMPROVEMENT OPPORTUNITIES**

Prioritize: **High** (meaningfully affects correctness, security, or maintainability) · **Medium** · **Low** (polish)

For each:
- What to improve and why
- Estimated effort (small / medium / large)

---

**STACK ASSESSMENT**

Answer directly: Is the current tech stack the right tool for this app?
- If yes: confirm it and explain why no migration is needed
- If no: recommend what to migrate to, explain the concrete benefits, and give an honest cost estimate

---

**RECOMMENDED NEXT STEPS**

Ordered list of the top 3–5 things to do, prioritized by impact. Frame as a plan the user can actually execute, not a wishlist.

---

### Step EX-5 — Ask what they want to do

After the report, ask:
> "That's the full picture of your project. What would you like to do next?
>
> Some options:
> 1. **Work on a new feature** — tell me what you want to add and we'll plan and build it
> 2. **Fix the security issues** — I'll walk you through each one
> 3. **Implement the improvements** — pick one or more from the list above
> 4. **Migrate the stack** — if the analysis recommended it, I'll plan the migration step by step
> 5. **Something else** — just tell me"

Wait for their answer. Then proceed accordingly:

- **New feature / improvements / security fixes** → go to **PART F — FRAMEWORK INITIALIZATION** to set up tracking, then start the work
- **Stack migration** → present a phased migration plan (phases, what moves first, estimated effort, risk), confirm with user, then execute phase by phase with full framework tracking
- **Something else** → handle it, then initialize the framework

**Do not scaffold a new project. Do not create new folders unless the user explicitly asks for a migration into a new structure.**

---

## PART C-NEW — New Project Wizard

### Step 6 — FE / BE / Fullstack choice

Print:
```
  ──────────────────────────────────────────────────────────────────
  STEP 1 OF 4 — What are you building?
  ──────────────────────────────────────────────────────────────────
```

Ask:
> "First, a quick vocabulary check so we're on the same page:
>
> **Frontend (FE)** — The part of an app the user sees and interacts with. Buttons, pages, forms, menus. It runs in the browser. Think of any website you've used — that visual layer is the frontend.
>
> **Backend (BE)** — The invisible engine behind the app. It stores data in a database, handles logins and security, processes business logic, and sends data to the frontend through an API. The user never sees it directly.
>
> **Full Stack** — Both. A complete application with a frontend the user sees *and* a backend that powers it. This is what most real-world apps are — for example, Instagram has a React Native frontend and a backend that stores photos, user accounts, and relationships.
>
> **What would you like to build?**
> 1. Frontend only (a website or UI, no server needed)
> 2. Backend only (an API or service, no visible interface)
> 3. Full Stack — a complete app (this is what I'd recommend if you want to build something real)
>
> Just say the number or describe what you have in mind."

Wait for answer. Store as BUILD_TYPE (frontend / backend / fullstack).

### Step 7 — IDE setup

Print:
```
  ──────────────────────────────────────────────────────────────────
  STEP 2 OF 4 — Your development tools
  ──────────────────────────────────────────────────────────────────
```

Ask:
> "Do you have a code editor (also called an IDE) installed? An IDE is the app you write code in — it's like Microsoft Word but for code. It highlights your code, warns you about mistakes, and gives you tools to navigate large projects.
>
> What do you currently use, or do you not have one yet?"

Wait for answer.

If they don't have one, recommend based on BUILD_TYPE:

If **backend** or **fullstack**:
> "For the backend (.NET / C#), I'd recommend:
>
> - **Visual Studio Community 2022** (free) — the gold standard for .NET development. Has built-in debugger, database tools, and everything you need. Download: https://visualstudio.microsoft.com/vs/community/
>   When installing, select the **'ASP.NET and web development'** workload.
>
> For the frontend, also grab:
> - **Visual Studio Code** (free, lightweight) — the most popular editor for TypeScript/React. Download: https://code.visualstudio.com/
>
> You'll use Visual Studio for the BE and VS Code for the FE. Many professional devs do exactly this."

If **frontend** only:
> "For frontend (TypeScript/React), I'd recommend:
> - **Visual Studio Code** (free) — the most popular editor in the world for frontend work. Has thousands of extensions, great TypeScript support, and a built-in terminal. Download: https://code.visualstudio.com/"

After recommending, ask: "Do you want to install one of those now, or do you have something you're already comfortable with?"
Wait for answer. If installing, tell them to come back when done. Do not proceed until they confirm they have an editor.

### Step 8 — The app idea

Print:
```
  ──────────────────────────────────────────────────────────────────
  STEP 3 OF 4 — Tell me about your idea
  ──────────────────────────────────────────────────────────────────
```

Ask:
> "Now the fun part. Tell me about the app you want to build.
>
> Don't worry about technical details — just describe it like you'd explain it to a friend:
> - What does it do?
> - Who uses it?
> - What's the main thing a user can do on it?
> - Is there anything it should NOT do (out of scope)?
>
> No idea is too simple or too ambitious. Just tell me what's in your head."

Wait for their full answer. Ask follow-up questions if needed to understand:
- The main entities (users, products, orders, posts, etc.)
- Whether there's user authentication (do different people log in?)
- Whether data needs to be stored long-term
- Whether multiple people use it simultaneously

Summarize back what you understood before proceeding:
> "So just to make sure I have this right: you want to build [summary]. The main things it should do are: [list]. Does that sound right?"

Wait for confirmation.

### Step 9 — Tech stack recommendation

Print:
```
  ──────────────────────────────────────────────────────────────────
  STEP 4 OF 4 — Recommending your tech stack
  ──────────────────────────────────────────────────────────────────
```

Based on BUILD_TYPE and the app idea, explain the recommended stack. Always frame it in plain language first:

**For Backend (always .NET Core in Edo's Framework):**
> "For your backend I'll use **.NET Core** (C#) — this is Microsoft's professional-grade framework for building APIs and web services. It's fast, secure, battle-tested, and used by companies like Stack Overflow, Bing, and thousands of enterprise apps. It's also an excellent first backend language because it forces good habits.
>
> Here's what your backend will include:
>
> **Architecture — Domain-Driven Design (DDD)**
> DDD is a way of structuring code so that the business logic (the rules of YOUR app) is the center of everything. Instead of organising code by what it does technically (database stuff here, web stuff there), you organise it by what it means in the real world (Customers, Orders, Products). This makes large apps much easier to understand and change.
>
> **Security — Microsoft Identity + JWT tokens**
> Every user will have a secure account. When they log in, they receive a JWT (JSON Web Token) — a tamper-proof digital pass that proves who they are. Every request they make carries this token. The backend validates it before doing anything. You never store passwords in plain text — they're hashed (scrambled one-way) before saving.
>
> **Database — SQL Server**
> Your data lives in SQL Server, a professional relational database. Think of it as a very structured, reliable spreadsheet that can handle millions of rows and complex relationships between things (a user has many orders; an order has many items).
>
> **Patterns — Repository + Service**
> The repository pattern gives every data type (User, Product, Order) a dedicated class responsible for reading and writing it to the database. The service pattern puts business logic in a dedicated layer. This keeps things cleanly separated and easy to test."

Then ask:
> "Does this make sense so far? Any questions before I start building?"

Wait for answer.

**For Frontend — ask first:**
> "For the frontend, the most common professional choice today is **React with TypeScript**. Here's what those mean:
>
> - **React** — a JavaScript library made by Facebook/Meta. Instead of writing a whole page at once, you build small reusable pieces called 'components' (a Button, a NavBar, a UserCard) and compose them into pages. Most modern web apps use it.
> - **TypeScript** — JavaScript with type safety. It catches mistakes while you're writing code before they become bugs in production. Every serious FE project uses it.
> - **Vite** — the build tool that compiles and serves your code in milliseconds. Much faster than older tools.
>
> Other popular options:
> - **Next.js** — React with server-side rendering. Good for SEO-heavy apps or blogs.
> - **Vue.js** — simpler syntax than React, good for smaller apps.
> - **Angular** — opinionated, more corporate, good for large enterprise teams.
> - **Laravel (PHP)** — if you specifically want PHP, this is the best framework for it.
> - **Plain HTML/CSS/JS** — no framework. Fine for a static site, not ideal for apps with data.
>
> My recommendation for what you're building: **React + TypeScript + Vite**. It's the right tool for a real app and it'll teach you patterns used across the industry.
>
> Would you like to go with that, or do you have a preference?"

Wait for their answer. Store as FE_FRAMEWORK.

### Step 10 — Folder and project naming

Ask:
> "Let's name your project. This name will become:
> - The root folder: `[ProjectName]/`
> - Your backend folder: `[ProjectName]/[BackendName]-API/`
> - Your frontend folder: `[ProjectName]/[FrontendName]/`
> - Your GitHub repositories (one for each)
>
> What would you like to call the project? (e.g. 'ShopTracker', 'MyPortfolio', 'TaskFlow')"

Wait for PROJECT_NAME.

Ask:
> "And what should the backend folder be called? I'd suggest **[PROJECT_NAME]-API** — or just say 'that works' and I'll use it."

Wait for BACKEND_NAME. Default to `[PROJECT_NAME]-API`.

Ask:
> "And the frontend folder? I'd suggest **[PROJECT_NAME]-Web** — or say 'that works'."

Wait for FRONTEND_NAME. Default to `[PROJECT_NAME]-Web`.

Ask where to create the project:
> "Where should I create the project folder? I can create it on your Desktop, or give me any path you prefer."

Wait for BASE_PATH. Default to Desktop.

---

## PART D — SCAFFOLDING

**Skip this entire part if the user is working on an existing project (PART C-EXISTING path). Go directly to PART F.**

Print:
```
  ══════════════════════════════════════════════════════════════════
  Building your project...
  ══════════════════════════════════════════════════════════════════
```

### Step 11 — Check SQL Server (if BE or fullstack)

Skip this step if BUILD_TYPE is frontend only.

```bash
sqlcmd -Q "SELECT @@VERSION" 2>&1 | head -3
```

If SQL Server is not found:
> "⚠️  SQL Server isn't installed. This is the database where your app's data will live — user accounts, [app-specific data], everything. Let me point you to the right download.
>
> Install one of these (both free for development):
> - **SQL Server 2022 Developer Edition** (full featured, for development): https://www.microsoft.com/en-us/sql-server/sql-server-downloads
> - **SQL Server 2022 Express** (lighter, limited size): same page, scroll to Express
>
> Also install **SQL Server Management Studio (SSMS)** — it's the visual tool for managing your database: https://aka.ms/ssmsfullsetup
>
> Come back when both are installed."

Wait for confirmation.

Also tell the user:
> "💡 Tip: Once SQL Server is installed, you can use the `/connect-db` skill to give me direct access to query your database — I can create test data, inspect tables, and help debug data issues right from this chat."

### Step 12 — Check .NET SDK (if BE or fullstack)

Skip if frontend only.

```bash
dotnet --version 2>&1
```

If not found:
> ".NET SDK isn't installed. This is the toolkit for building C# apps."
```bash
start https://dotnet.microsoft.com/en-us/download
```
"Install .NET 8 or later (LTS). Come back when done."

Wait for confirmation.

### Step 13 — Check Node.js (if FE or fullstack)

Skip if backend only.

```bash
node --version 2>&1
npm --version 2>&1
```

If not found:
> "Node.js isn't installed. This is needed to run the frontend build tools."
```bash
start https://nodejs.org/en/download
```
"Install the LTS version. Come back when done."

Wait for confirmation.

### Step 14 — Create folder structure

```bash
mkdir -p "BASE_PATH/PROJECT_NAME/BACKEND_NAME"
mkdir -p "BASE_PATH/PROJECT_NAME/FRONTEND_NAME"
```

Print:
```
  ✓ Created BASE_PATH/PROJECT_NAME/
  ✓ Created BASE_PATH/PROJECT_NAME/BACKEND_NAME/
  ✓ Created BASE_PATH/PROJECT_NAME/FRONTEND_NAME/
```

### Step 15 — Scaffold the Backend (if BE or fullstack)

Skip if frontend only.

Print:
```
  ──────────────────────────────────────────────────────────────────
  Scaffolding Backend — .NET Core + DDD + Identity + JWT
  ──────────────────────────────────────────────────────────────────
```

Create the solution and projects:
```bash
cd "BASE_PATH/PROJECT_NAME/BACKEND_NAME"

# Solution
dotnet new sln -n PROJECT_NAME

# Projects — Clean Architecture layers
dotnet new webapi -n "PROJECT_NAME.API" -o "src/PROJECT_NAME.API" --no-openapi
dotnet new classlib -n "PROJECT_NAME.Application" -o "src/PROJECT_NAME.Application"
dotnet new classlib -n "PROJECT_NAME.Domain" -o "src/PROJECT_NAME.Domain"
dotnet new classlib -n "PROJECT_NAME.Infrastructure" -o "src/PROJECT_NAME.Infrastructure"

# Add all to solution
dotnet sln add src/PROJECT_NAME.API/PROJECT_NAME.API.csproj
dotnet sln add src/PROJECT_NAME.Application/PROJECT_NAME.Application.csproj
dotnet sln add src/PROJECT_NAME.Domain/PROJECT_NAME.Domain.csproj
dotnet sln add src/PROJECT_NAME.Infrastructure/PROJECT_NAME.Infrastructure.csproj

# Project references (clean architecture dependency flow)
dotnet add src/PROJECT_NAME.API/PROJECT_NAME.API.csproj reference src/PROJECT_NAME.Application/PROJECT_NAME.Application.csproj
dotnet add src/PROJECT_NAME.API/PROJECT_NAME.API.csproj reference src/PROJECT_NAME.Infrastructure/PROJECT_NAME.Infrastructure.csproj
dotnet add src/PROJECT_NAME.Application/PROJECT_NAME.Application.csproj reference src/PROJECT_NAME.Domain/PROJECT_NAME.Domain.csproj
dotnet add src/PROJECT_NAME.Infrastructure/PROJECT_NAME.Infrastructure.csproj reference src/PROJECT_NAME.Application/PROJECT_NAME.Application.csproj

# NuGet packages
dotnet add src/PROJECT_NAME.API/PROJECT_NAME.API.csproj package Microsoft.AspNetCore.Authentication.JwtBearer
dotnet add src/PROJECT_NAME.Infrastructure/PROJECT_NAME.Infrastructure.csproj package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add src/PROJECT_NAME.Infrastructure/PROJECT_NAME.Infrastructure.csproj package Microsoft.EntityFrameworkCore.SqlServer
dotnet add src/PROJECT_NAME.Infrastructure/PROJECT_NAME.Infrastructure.csproj package Microsoft.EntityFrameworkCore.Tools
dotnet add src/PROJECT_NAME.API/PROJECT_NAME.API.csproj package Swashbuckle.AspNetCore
dotnet add src/PROJECT_NAME.API/PROJECT_NAME.API.csproj package Serilog.AspNetCore
dotnet add src/PROJECT_NAME.API/PROJECT_NAME.API.csproj package Serilog.Sinks.File
```

Create the core folder structure inside each project:

**Domain layer** (`src/PROJECT_NAME.Domain/`):
```bash
mkdir -p "src/PROJECT_NAME.Domain/Entities"
mkdir -p "src/PROJECT_NAME.Domain/Interfaces"
mkdir -p "src/PROJECT_NAME.Domain/Interfaces/Repositories"
mkdir -p "src/PROJECT_NAME.Domain/Enums"
mkdir -p "src/PROJECT_NAME.Domain/Exceptions"
```

**Application layer** (`src/PROJECT_NAME.Application/`):
```bash
mkdir -p "src/PROJECT_NAME.Application/Identity"
mkdir -p "src/PROJECT_NAME.Application/Services"
mkdir -p "src/PROJECT_NAME.Application/Features/Auth/Commands"
mkdir -p "src/PROJECT_NAME.Application/Common"
```

**Infrastructure layer** (`src/PROJECT_NAME.Infrastructure/`):
```bash
mkdir -p "src/PROJECT_NAME.Infrastructure/Persistence"
mkdir -p "src/PROJECT_NAME.Infrastructure/Persistence/Configurations"
mkdir -p "src/PROJECT_NAME.Infrastructure/Persistence/Repositories"
mkdir -p "src/PROJECT_NAME.Infrastructure/Identity"
mkdir -p "src/PROJECT_NAME.Infrastructure/Services"
mkdir -p "src/PROJECT_NAME.Infrastructure/Migrations"
```

**API layer** (`src/PROJECT_NAME.API/`):
```bash
mkdir -p "src/PROJECT_NAME.API/Controllers"
mkdir -p "src/PROJECT_NAME.API/Middleware"
mkdir -p "src/PROJECT_NAME.API/Properties"
```

Now write the core scaffolding files. Create these files with their content:

**`src/PROJECT_NAME.Application/Identity/ApplicationUser.cs`:**
```csharp
using Microsoft.AspNetCore.Identity;

namespace PROJECT_NAME.Application.Identity;

public class ApplicationUser : IdentityUser
{
    public string FullName { get; set; } = string.Empty;
    public bool IsActive { get; set; } = true;
    public DateTime? LastLoginAt { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
}
```

**`src/PROJECT_NAME.Application/Identity/ApplicationRole.cs`:**
```csharp
using Microsoft.AspNetCore.Identity;

namespace PROJECT_NAME.Application.Identity;

public class ApplicationRole : IdentityRole { }
```

**`src/PROJECT_NAME.Application/Common/Result.cs`:**
```csharp
namespace PROJECT_NAME.Application.Common;

public class Result<T>
{
    public bool IsSuccess { get; private set; }
    public T? Value { get; private set; }
    public string? Error { get; private set; }

    public static Result<T> Success(T value) => new() { IsSuccess = true, Value = value };
    public static Result<T> Failure(string error) => new() { IsSuccess = false, Error = error };
}
```

**`src/PROJECT_NAME.Infrastructure/Persistence/AppDbContext.cs`:**
```csharp
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;
using PROJECT_NAME.Application.Identity;

namespace PROJECT_NAME.Infrastructure.Persistence;

public class AppDbContext : IdentityDbContext<ApplicationUser, ApplicationRole, string>
{
    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }

    protected override void OnModelCreating(ModelBuilder builder)
    {
        base.OnModelCreating(builder);
        builder.ApplyConfigurationsFromAssembly(typeof(AppDbContext).Assembly);
    }
}
```

**`src/PROJECT_NAME.Infrastructure/DependencyInjection.cs`:**
```csharp
using Microsoft.AspNetCore.Identity;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using PROJECT_NAME.Application.Identity;
using PROJECT_NAME.Infrastructure.Persistence;

namespace PROJECT_NAME.Infrastructure;

public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(
        this IServiceCollection services,
        IConfiguration configuration)
    {
        services.AddDbContext<AppDbContext>(options =>
            options.UseSqlServer(
                configuration.GetConnectionString("DefaultConnection"),
                sql => sql.MigrationsAssembly(typeof(AppDbContext).Assembly.FullName)));

        services.AddIdentity<ApplicationUser, ApplicationRole>(options =>
            {
                options.Password.RequiredLength = 8;
                options.Password.RequireDigit = true;
                options.Password.RequireUppercase = true;
                options.Password.RequireNonAlphanumeric = false;
                options.User.RequireUniqueEmail = true;
                options.Lockout.MaxFailedAccessAttempts = 5;
                options.Lockout.DefaultLockoutTimeSpan = TimeSpan.FromMinutes(15);
            })
            .AddEntityFrameworkStores<AppDbContext>()
            .AddDefaultTokenProviders();

        return services;
    }

    public static async Task SeedRolesAsync(IServiceProvider services)
    {
        using var scope = services.CreateScope();
        var roleManager = scope.ServiceProvider.GetRequiredService<RoleManager<ApplicationRole>>();
        string[] roles = ["Admin", "User"];
        foreach (var role in roles)
            if (!await roleManager.RoleExistsAsync(role))
                await roleManager.CreateAsync(new ApplicationRole { Name = role });
    }

    public static async Task SeedDevAdminAsync(IServiceProvider services)
    {
        using var scope = services.CreateScope();
        var userManager = scope.ServiceProvider.GetRequiredService<UserManager<ApplicationUser>>();
        const string email = "admin@PROJECT_NAME.local";
        const string password = "Admin1234!";
        if (await userManager.FindByEmailAsync(email) is null)
        {
            var admin = new ApplicationUser { UserName = email, Email = email, FullName = "Dev Admin", EmailConfirmed = true };
            await userManager.CreateAsync(admin, password);
            await userManager.AddToRoleAsync(admin, "Admin");
        }
    }
}
```

**`src/PROJECT_NAME.API/appsettings.json`:**
```json
{
  "Logging": { "LogLevel": { "Default": "Information", "Microsoft.AspNetCore": "Warning" } },
  "AllowedHosts": "*",
  "ConnectionStrings": {
    "DefaultConnection": "Server=(localdb)\\mssqllocaldb;Database=PROJECT_NAMEDb;Trusted_Connection=True;MultipleActiveResultSets=true"
  },
  "Jwt": {
    "Secret": "REPLACE_WITH_A_STRONG_SECRET_AT_LEAST_32_CHARACTERS_LONG",
    "Issuer": "PROJECT_NAME.API",
    "Audience": "PROJECT_NAME.UI",
    "AccessTokenExpiryMinutes": "15",
    "RefreshTokenExpiryDays": "7"
  },
  "Cors": {
    "AllowedOrigins": ["http://localhost:5173", "http://localhost:3000"]
  }
}
```

**`src/PROJECT_NAME.API/Middleware/ExceptionHandlingMiddleware.cs`:**
```csharp
using System.Net;
using System.Text.Json;

namespace PROJECT_NAME.API.Middleware;

public class ExceptionHandlingMiddleware(RequestDelegate next, ILogger<ExceptionHandlingMiddleware> logger)
{
    public async Task InvokeAsync(HttpContext context)
    {
        try { await next(context); }
        catch (Exception ex)
        {
            logger.LogError(ex, "Unhandled exception on {Method} {Path}", context.Request.Method, context.Request.Path);
            context.Response.StatusCode = (int)HttpStatusCode.InternalServerError;
            context.Response.ContentType = "application/json";
            await context.Response.WriteAsync(JsonSerializer.Serialize(new { error = "An unexpected error occurred." }));
        }
    }
}
```

**`src/PROJECT_NAME.API/Controllers/BaseApiController.cs`:**
```csharp
using Microsoft.AspNetCore.Mvc;

namespace PROJECT_NAME.API.Controllers;

[ApiController]
[Route("api/v1/[controller]")]
public abstract class BaseApiController : ControllerBase { }
```

**`src/PROJECT_NAME.API/Program.cs`:**
```csharp
using System.Text;
using Microsoft.AspNetCore.Authentication.JwtBearer;
using Microsoft.IdentityModel.Tokens;
using Microsoft.OpenApi.Models;
using PROJECT_NAME.API.Middleware;
using PROJECT_NAME.Infrastructure;
using Serilog;
using Serilog.Events;

Log.Logger = new LoggerConfiguration().MinimumLevel.Override("Microsoft", LogEventLevel.Warning).WriteTo.Console().CreateBootstrapLogger();

try
{
    var builder = WebApplication.CreateBuilder(args);

    builder.Host.UseSerilog((ctx, _, config) => config
        .ReadFrom.Configuration(ctx.Configuration)
        .Enrich.FromLogContext()
        .WriteTo.Console(outputTemplate: "[{Timestamp:HH:mm:ss} {Level:u3}] {SourceContext}: {Message:lj}{NewLine}{Exception}")
        .WriteTo.File("logs/api-.log", rollingInterval: RollingInterval.Day, retainedFileCountLimit: 30)
        .MinimumLevel.Override("Microsoft.EntityFrameworkCore", LogEventLevel.Warning)
        .MinimumLevel.Override("Microsoft.AspNetCore", LogEventLevel.Warning));

    builder.Services.AddControllers();
    builder.Services.AddEndpointsApiExplorer();
    builder.Services.AddInfrastructure(builder.Configuration);

    var jwtSecret = builder.Configuration["Jwt:Secret"] ?? throw new InvalidOperationException("Jwt:Secret is required.");
    builder.Services.AddAuthentication(o => { o.DefaultAuthenticateScheme = JwtBearerDefaults.AuthenticationScheme; o.DefaultChallengeScheme = JwtBearerDefaults.AuthenticationScheme; })
        .AddJwtBearer(o => o.TokenValidationParameters = new TokenValidationParameters
        {
            ValidateIssuerSigningKey = true,
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(jwtSecret)),
            ValidateIssuer = true, ValidIssuer = builder.Configuration["Jwt:Issuer"],
            ValidateAudience = true, ValidAudience = builder.Configuration["Jwt:Audience"],
            ValidateLifetime = true, ClockSkew = TimeSpan.Zero
        });

    builder.Services.AddAuthorization();

    builder.Services.AddCors(o => o.AddPolicy("FrontendPolicy", p =>
    {
        var origins = builder.Configuration.GetSection("Cors:AllowedOrigins").Get<string[]>() ?? [];
        p.WithOrigins(origins).AllowAnyHeader().AllowAnyMethod().AllowCredentials();
    }));

    builder.Services.AddSwaggerGen(o =>
    {
        o.SwaggerDoc("v1", new OpenApiInfo { Title = "PROJECT_NAME API", Version = "v1" });
        o.AddSecurityDefinition("Bearer", new OpenApiSecurityScheme { Description = "JWT Authorization header. Enter: Bearer {token}", Name = "Authorization", In = ParameterLocation.Header, Type = SecuritySchemeType.ApiKey, Scheme = "Bearer" });
        o.AddSecurityRequirement(new OpenApiSecurityRequirement { { new OpenApiSecurityScheme { Reference = new OpenApiReference { Type = ReferenceType.SecurityScheme, Id = "Bearer" } }, [] } });
    });

    var app = builder.Build();

    app.UseMiddleware<ExceptionHandlingMiddleware>();

    if (app.Environment.IsDevelopment())
    {
        app.UseSwagger();
        app.UseSwaggerUI(c => { c.SwaggerEndpoint("/swagger/v1/swagger.json", "PROJECT_NAME API v1"); c.RoutePrefix = "swagger"; });
        app.MapGet("/", () => Results.Redirect("/swagger")).ExcludeFromDescription();
        await DependencyInjection.SeedDevAdminAsync(app.Services);
    }

    app.UseCors("FrontendPolicy");
    app.UseAuthentication();
    app.UseAuthorization();
    app.MapControllers();

    await DependencyInjection.SeedRolesAsync(app.Services);

    app.Run();
}
catch (Exception ex) when (ex is not HostAbortedException) { Log.Fatal(ex, "Application terminated unexpectedly."); }
finally { Log.CloseAndFlush(); }

public partial class Program { }
```

**`src/PROJECT_NAME.API/Properties/launchSettings.json`:**
```json
{
  "$schema": "https://json.schemastore.org/launchsettings.json",
  "profiles": {
    "http": {
      "commandName": "Project",
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "http://localhost:5000",
      "environmentVariables": { "ASPNETCORE_ENVIRONMENT": "Development" }
    },
    "https": {
      "commandName": "Project",
      "launchBrowser": true,
      "launchUrl": "swagger",
      "applicationUrl": "https://localhost:7000;http://localhost:5000",
      "environmentVariables": { "ASPNETCORE_ENVIRONMENT": "Development" }
    }
  }
}
```

Create `.gitignore` for the backend:
```bash
cat > "BASE_PATH/PROJECT_NAME/BACKEND_NAME/.gitignore" << 'EOF'
bin/
obj/
*.user
.vs/
logs/
*.env
appsettings.Development.json
EOF
```

Run initial build check:
```bash
cd "BASE_PATH/PROJECT_NAME/BACKEND_NAME" && dotnet build 2>&1 | tail -10
```

If build fails, show the error and fix it before continuing.

### Step 16 — Scaffold the Frontend (if FE or fullstack)

Skip if backend only.

Print:
```
  ──────────────────────────────────────────────────────────────────
  Scaffolding Frontend — FE_FRAMEWORK
  ──────────────────────────────────────────────────────────────────
```

If FE_FRAMEWORK is React + TypeScript + Vite (default):
```bash
cd "BASE_PATH/PROJECT_NAME"
npm create vite@latest FRONTEND_NAME -- --template react-ts
cd FRONTEND_NAME
npm install
npm install axios react-router-dom @tanstack/react-query zustand react-hook-form zod @hookform/resolvers
npm install -D @types/node
```

Create `.env.local`:
```bash
echo "VITE_API_BASE_URL=https://localhost:7000/api/v1" > "BASE_PATH/PROJECT_NAME/FRONTEND_NAME/.env.local"
```

Create `.gitignore` for frontend:
```bash
cat > "BASE_PATH/PROJECT_NAME/FRONTEND_NAME/.gitignore" << 'EOF'
node_modules/
dist/
.env.local
.env*.local
EOF
```

If FE_FRAMEWORK is Next.js:
```bash
cd "BASE_PATH/PROJECT_NAME"
npx create-next-app@latest FRONTEND_NAME --typescript --tailwind --app
```

If FE_FRAMEWORK is Vue:
```bash
cd "BASE_PATH/PROJECT_NAME"
npm create vue@latest FRONTEND_NAME
```

If FE_FRAMEWORK is Laravel (PHP):
Tell the user to install PHP + Composer first if not present, then:
```bash
composer create-project laravel/laravel "BASE_PATH/PROJECT_NAME/FRONTEND_NAME"
```

---

## PART E — GIT REPOSITORIES

**Skip this part for existing projects that already have git repos. Only do Step 17 (init) if a project folder has no `.git` directory.**

### Step 17 — Initialize local git repos

Print:
```
  ──────────────────────────────────────────────────────────────────
  Setting up Git repositories
  ──────────────────────────────────────────────────────────────────
```

Explain briefly:
> "Each part of your app (backend and frontend) gets its own Git repository. This is standard practice — it means the BE team and FE team can work independently, deploy separately, and have their own history. Let's set them up."

Initialize backend repo (if BE or fullstack):
```bash
cd "BASE_PATH/PROJECT_NAME/BACKEND_NAME"
git init
git add -A
git commit -m "Initial scaffold: .NET Core + DDD + Identity + JWT"
```

Initialize frontend repo (if FE or fullstack):
```bash
cd "BASE_PATH/PROJECT_NAME/FRONTEND_NAME"
git init
git add -A
git commit -m "Initial scaffold: FE_FRAMEWORK"
```

### Step 18 — Create GitHub repositories

Check if gh CLI is available:
```bash
gh --version 2>&1
```

If not installed:
```bash
winget install --id GitHub.cli -e 2>&1
```

Check auth:
```bash
gh auth status 2>&1
```

If not authenticated:
```bash
gh auth login
```

Ask the user:
> "Would you like to create GitHub repositories for these projects now? I'll create them and push the initial code. (Recommended — it backs everything up immediately)"

If yes, create them:
```bash
# Backend
cd "BASE_PATH/PROJECT_NAME/BACKEND_NAME"
gh repo create "BACKEND_NAME" --public --source=. --remote=origin --push --description "PROJECT_NAME backend API — .NET Core"

# Frontend
cd "BASE_PATH/PROJECT_NAME/FRONTEND_NAME"
gh repo create "FRONTEND_NAME" --public --source=. --remote=origin --push --description "PROJECT_NAME frontend — FE_FRAMEWORK"
```

If they want private repos, change `--public` to `--private`.

---

## PART F — FRAMEWORK INITIALIZATION

### Step 19 — Initialize Edo's Framework for the project

For **new projects**, ask the user which part to track first (or both):
> "Edo's Framework tracks one project at a time. Would you like to start with the backend or the frontend?"

Wait for answer.

For **existing projects**, use the primary project path they provided (or ask if ambiguous).

Check if `.claude` folder exists in the chosen project path:
```bash
ls "PROJECT_PATH/.claude" 2>/dev/null
```

If not found, run `/init` for that project.

---

## PART G — FINAL REPORT

For **new projects**, print this full report:

```
╔══════════════════════════════════════════════════════════════════╗
║              PROJECT SETUP COMPLETE                              ║
╚══════════════════════════════════════════════════════════════════╝

  Project:    PROJECT_NAME
  Location:   BASE_PATH/PROJECT_NAME/

  ┌─────────────────────────────────────────────────────────────┐
  │  BACKEND  (BACKEND_NAME)                                    │
  │                                                             │
  │  Framework:    .NET Core (C#)                               │
  │  Architecture: Clean Architecture + Domain-Driven Design    │
  │  Auth:         Microsoft Identity + JWT Bearer tokens       │
  │  Database:     SQL Server (Entity Framework Core)           │
  │  Patterns:     Repository pattern + Service layer           │
  │  API docs:     Swagger UI at https://localhost:7000/swagger  │
  │  Dev admin:    admin@PROJECT_NAME.local / Admin1234!         │
  │  Git repo:     BACKEND_REPO_URL                             │
  └─────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────┐
  │  FRONTEND  (FRONTEND_NAME)                                  │
  │                                                             │
  │  Framework:    FE_FRAMEWORK                                 │
  │  API base URL: https://localhost:7000/api/v1                │
  │  Dev server:   http://localhost:5173 (npm run dev)          │
  │  Git repo:     FRONTEND_REPO_URL                            │
  └─────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────┐
  │  HOW YOUR APP IS SECURED                                    │
  │                                                             │
  │  ✓ Passwords are hashed with bcrypt — never stored plain   │
  │  ✓ JWT tokens expire after 15 minutes                      │
  │  ✓ Account lockout after 5 failed login attempts           │
  │  ✓ CORS policy limits which URLs can call your API         │
  │  ✓ All requests go through exception handling middleware    │
  └─────────────────────────────────────────────────────────────┘

  ┌─────────────────────────────────────────────────────────────┐
  │  WHAT TO DO NEXT                                            │
  │                                                             │
  │  1. Run migrations to create your database:                │
  │     cd BACKEND_NAME/src/PROJECT_NAME.API                   │
  │     dotnet ef migrations add InitialCreate                  │
  │     dotnet ef database update                               │
  │                                                             │
  │  2. Start the backend:    dotnet run                        │
  │  3. Start the frontend:   npm run dev                       │
  │  4. Start a feature:      /new-feature [name]               │
  │  5. Create a branch:      /branch feature/[name]            │
  └─────────────────────────────────────────────────────────────┘
```

For **existing projects**, print a shorter summary:

```
╔══════════════════════════════════════════════════════════════════╗
║              EDO'S FRAMEWORK INITIALIZED                         ║
╚══════════════════════════════════════════════════════════════════╝

  Project:    PROJECT_NAME
  Location:   PROJECT_PATH

  Analysis complete. Framework tracking active.
  Use /new-feature [name] to start tracking your first piece of work.
```

Then deliver the full skills onboarding:

---

**Edo's Framework is active.** Here's your complete toolkit:

**Session tracking:** `/new-feature [name]` · `/active` · `/checkpoint` ⭐ · `/wrap-up [name]`

**Git workflow:** `/git` · `/branch [name]` · `/commit` · `/push`

**Collaboration:** `/send-report` · `/connect-db`

**The development loop:**
```
/branch feature/[name]  →  write code  →  /commit  →  /checkpoint
→  /push  →  PR reviewed by EdinMhc  →  merge  →  /wrap-up [name]
→  /send-report  (if BE changes)
```

> ⭐ **Most important habit:** Run `/checkpoint` after every meaningful chunk of work. I'll also remind you. A missed checkpoint = lost context next session.

Ready. What would you like to build first?
