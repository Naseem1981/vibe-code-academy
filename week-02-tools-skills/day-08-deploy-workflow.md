# Day 8: Ship It — Deploy Your App + Build Your Workflow Card

## Outcome artifact
By the end of this module your finished app will be live at a real public URL — something like `your-app.netlify.app` — accessible from any device and shareable with anyone.

<!-- ASSET: diagram | Two-step flow: GitHub box (your code is stored here) with arrow to Netlify box (serves your app to the world), with a URL badge at the end showing "your-app.netlify.app" -->

## The core idea

"Deployed" means your app is running on a server somewhere in the world, not just on your computer. When you open `index.html` in your browser right now, it only works because you are on the machine where the file lives. Nobody else can see it. Deploying takes that file and puts it somewhere with a public address — a real URL anyone can visit on any device.

You have two tasks: push your project to GitHub, then connect GitHub to Netlify. GitHub stores your code. Netlify takes that code, serves it as a website, and gives you a URL. Every time you push new code to GitHub, Netlify automatically updates the live site. The entire setup takes about 10 minutes. Both services are free.

**GitHub** is where developers store code. Think of it as a cloud backup for your project — but better, because it also tracks every version. Once your project is on GitHub, you have a permanent off-computer copy. If your laptop dies, your code is safe.

**Netlify** is a hosting service — it takes your HTML, CSS, and JavaScript files and serves them as a live website. The free tier is generous: up to 100 sites, 100 GB bandwidth per month, automatic HTTPS. You will not hit any of these limits with a personal project.

The connection between GitHub and Netlify means you never have to manually re-upload files. You make changes locally, commit, push to GitHub, and Netlify automatically detects the change and updates the live site within a minute.

## Step-by-step walkthrough

### Create a GitHub account and repository

**1.** Go to `https://github.com`. If you created an account in Module 4 for the GitHub plugin, sign in. If not, click **Sign up** and create a free account.

**2.** Once signed in, click the **+** icon in the top-right corner → **New repository**.

**3.** Fill in the repository form:
- **Repository name:** `my-first-app` (or whatever you want to call it)
- **Description:** "My first vibe-coded app"
- **Visibility:** Public (Netlify's free tier works with public repos)
- Leave all other options as defaults
- Click **Create repository**

**4.** GitHub shows you a page with setup commands. You want the section labelled **"…or push an existing repository from the command line"**. It shows three commands like:
```
git remote add origin https://github.com/YourUsername/my-first-app.git
git branch -M main
git push -u origin main
```

Copy these — you will use them in the next step.

### Push your project to GitHub

**5.** Open VS Code. Press `` Ctrl+` `` to open the terminal. Navigate to your project folder if you are not already there:
```
cd my-first-app
```

**6.** Run the three commands from GitHub one at a time. Paste the first command and press Enter:
```
git remote add origin https://github.com/YourUsername/my-first-app.git
```

Then the second:
```
git branch -M main
```

Then the third:
```
git push -u origin main
```

When you run `git push`, GitHub may ask for your username and password. Enter your GitHub username, and for the password, use a personal access token (the one you created in Module 4). GitHub no longer accepts plain passwords for command-line pushes.

If you did not create a token in Module 4, create one now: GitHub → Settings → Developer settings → Personal access tokens → Tokens (classic) → Generate new token. Tick `repo` scope. Copy and use the token as your password.

**7.** After the push completes, refresh your GitHub repository page in the browser. You should see your `index.html`, `styles.css`, `script.js`, `CLAUDE.md`, `feature-log.md`, and `fix-log.md` files listed. Your code is now on GitHub.

### Deploy to Netlify

**8.** Go to `https://netlify.com`. Click **Sign up** → choose **Sign up with GitHub**. Authorise Netlify to access your GitHub account. This is what allows Netlify to pull your code automatically.

**9.** After signing in, you will see the Netlify dashboard. Click **Add new site** → **Import an existing project**.

**10.** Click **GitHub** as your source. Netlify will show you a list of your repositories. Find `my-first-app` and click it.

**11.** Netlify shows a deployment configuration screen:
- **Branch to deploy:** `main` — leave as is
- **Build command:** leave this empty (your project has no build step — it is plain HTML/CSS/JS)
- **Publish directory:** leave as `.` or leave empty — Netlify will find `index.html` automatically

Click **Deploy site**.

**12.** Netlify deploys your site. This takes 10–30 seconds. You will see a progress indicator, then a green **Published** status.

**13.** Netlify generates a random URL for your site — something like `peaceful-hamilton-3f2a1b.netlify.app`. Click the URL to open your site in a new tab.

Your app is live. Open it on your phone. Share the link with someone. It works for everyone.

### Get a better URL

**14.** The random URL works fine but is hard to share. Give it a cleaner name:
- In the Netlify dashboard, click **Site configuration** → **Change site name**
- Type a name — e.g., `alexs-landing-page`
- Click **Save** — your site is now at `alexs-landing-page.netlify.app`

### Test the auto-deploy pipeline

**15.** Make a small change to your project locally — change one word in the bio text in `index.html`. Save it.

**16.** In VS Code terminal:
```
git add .
git commit -m "test: update bio text"
git push
```

**17.** Go back to Netlify dashboard. Within 60 seconds you will see a new "Deploy in progress" and then "Published" status. Refresh your live URL — the change is live.

This is the deploy loop: `edit → commit → push → Netlify auto-deploys`. You never need to touch Netlify manually again.

<!-- ASSET: diagram | Auto-deploy pipeline: local file change → git commit → git push → GitHub → Netlify auto-deploy trigger → live URL updated, with timeline showing ~60 seconds end-to-end -->

## Practical workflow

1. Create GitHub account (github.com) → create a new repository
2. Copy the 3 setup commands from GitHub's "push existing repository" section
3. Run all 3 commands in your project terminal
4. Go to netlify.com → sign up with GitHub → Add new site → Import from GitHub
5. Select your repository → leave Build command empty → click Deploy site
6. Wait 30 seconds → click the generated URL → confirm app is live
7. Rename site: Site configuration → Change site name → type new name → save
8. Future deploys: `git add . && git commit -m "message" && git push` → Netlify auto-deploys

## Common mistakes

**Mistake 1: Leaving the Netlify build command filled in.**
Netlify sometimes pre-fills a build command based on what it detects. For a plain HTML/CSS/JS project with no package.json, any build command will fail because there is nothing to build. Fix: delete everything from the "Build command" field before deploying. Leave it empty. Netlify will serve `index.html` directly without running any build process.

**Mistake 2: The deployed site looks broken but local looks fine.**
The most common cause: a file path that works on your computer but not on a server. For example, `<img src="C:\Users\YourName\Desktop\photo.jpg">` works locally but is invisible to anyone else. Fix: all file references must be relative paths (`./photo.jpg` or just `photo.jpg`). Claude Code only creates relative paths, so this usually happens when you manually add something. Check your `index.html` for any absolute paths (starting with `C:\` or `/Users/`).

**Mistake 3: Forgetting to push after committing.**
You commit locally (`git commit`) and check the Netlify dashboard wondering why nothing updated. Commits are local — they don't reach GitHub until you push. Fix: after committing, always run `git push`. Make it a habit: commit and push are one action, not two. The shortcut: `git add . && git commit -m "message" && git push`.

## Your turn

Go to your live Netlify URL on your phone. Your app should load — the full dark design, the buttons, the toggle, everything — exactly as it looks on your desktop browser.

If it loads correctly: you have shipped something real. The URL is yours to keep. You can share it now.

If the page loads but looks broken (layout issues on mobile), open Claude Code and ask:
```
My app looks broken on mobile — [describe the specific problem, e.g., "the buttons overflow the screen"]. 
Fix the responsive layout so it looks correct on small screens without changing the desktop design.
```

Then commit and push the fix — Netlify will auto-deploy.

## Prompt / Template / Checklist pack

### Pre-Launch Checklist

Run through this before sharing your URL with anyone.

**App behaviour:**
- [ ] Page loads at the live URL without errors
- [ ] All three buttons from Module 1 are visible and respond to hover
- [ ] Dark/light mode toggle works and saves preference
- [ ] Footer is visible with correct text
- [ ] Back to top button appears when scrolling down, disappears when scrolling up
- [ ] No placeholder text left in the bio (replace "Alex" with your real name or intended name)

**Cross-device testing:**
- [ ] Tested on desktop browser (Chrome or Firefox)
- [ ] Tested on mobile phone (open the URL on your phone)
- [ ] No obvious layout breakage on small screens

**Technical checks:**
- [ ] No console errors: press F12 → Console → no red text
- [ ] No hardcoded local file paths: no `C:\Users\...` or `/Users/...` in the HTML
- [ ] CLAUDE.md and fix-log.md are NOT linked or visible in the app (they are internal files)

**GitHub:**
- [ ] Latest version pushed to GitHub: `git push` completed successfully
- [ ] Netlify shows "Published" for the most recent deploy

**URL:**
- [ ] Site has been renamed to something readable (not the random Netlify default)
- [ ] URL copied and ready to share

---

### Deploy Troubleshooting Guide

**Problem: Netlify deploy shows "Failed" status**
Click the failed deploy → read the error log. Most common causes:
- Build command is set (and failing) — fix: clear the build command field in Site configuration → Build settings
- Missing file: `index.html` is expected in the root — confirm your root folder contains `index.html` directly (not inside a subfolder)

**Problem: Site loads but shows only a folder listing, not the app**
Netlify couldn't find `index.html`. Fix: go to Site configuration → Build settings → set Publish directory to the folder that contains `index.html`. For a flat project with no subfolders, leave it as `.` (dot).

**Problem: Site loads blank (white page)**
Press F12 → Console → look for errors. Most common: a JavaScript file failed to load because its path is wrong in the HTML. Confirm `<script src="script.js">` uses a relative path (no leading slash, no `C:\`).

**Problem: git push asks for password and rejects it**
GitHub no longer accepts account passwords for `git push`. Fix: create a personal access token (GitHub → Settings → Developer settings → Personal access tokens → Generate new token, tick `repo`) and use the token as your password when prompted.

**Problem: Changes not showing on live site after push**
1. Confirm the push succeeded: run `git push` and check for errors
2. Check Netlify dashboard: is a new deploy in progress or published?
3. Hard refresh the browser: `Ctrl+Shift+R` (Windows/Linux) or `Cmd+Shift+R` (Mac) to bypass the browser cache
# Module 7: Your Repeatable Workflow — Build Anything, Anytime

## Outcome artifact
By the end of this module you will have a completed Personal Workflow Card — a one-page document capturing your full process from idea to shipped app, customised to your setup and preferences — reusable for every future project.

<!-- ASSET: diagram | A filled-in workflow card as a visual — clean card design with six sections: Setup, First Prompt, Vibe Loop, Context, Plugins, Deploy — each with 3-4 bullet points filled in -->

## The core idea

You have built and shipped a real app. You went from no tools installed to a live URL in seven modules. That process — the exact steps you followed — is a workflow. It is repeatable. Every future project you want to build follows the same structure.

The difference between a beginner who builds one app and someone who ships ideas regularly is not talent or technical skill — it is documentation. The person who ships regularly has written down their process. They know what to open first, what to type, what to expect, and what to do when something breaks. They do not have to think about the meta-level ("what do I do now?") because it is already decided. They just execute.

Your Personal Workflow Card is that documentation. It lives in one place, covers the full arc from zero to shipped, and is written in your words — the way you actually work, not the way a tutorial thinks you work. In three months when you start a new project, you open this card. In a year, you update it as your process improves.

This module also covers the second-project mistakes — the errors people make once they have some confidence. Understanding them now means you avoid them before they cost you time.

## Step-by-step walkthrough

### Review what you built

Before filling in your workflow card, spend five minutes reviewing the modules. For each module, ask: what did I do, and what do I now have?

| Module | What I did | What I now have |
|--------|-----------|-----------------|
| 0 | Installed VS Code, Node.js, Claude Code CLI, the claude.ai extension | A working Claude environment |
| 1 | Wrote a structured first prompt, built a landing page | A working app in my browser |
| 2 | Added 3 features using the vibe coding loop, committed each one | 3 features + a full git history |
| 3 | Created CLAUDE.md with project context and rules | Sessions that start correctly every time |
| 4 | Installed GitHub, Filesystem, and Web Search plugins | Claude with access to external tools |
| 5 | Debugged a real bug using the 3-part prompt, filled in the fix log | A documented debugging process |
| 6 | Pushed to GitHub, deployed to Netlify, tested the live URL | A real app at a real URL |

This is your process. You have done it once. Writing the workflow card locks it in.

### Fill in your Personal Workflow Card

**1.** Open VS Code. Create a new file in your project folder: `workflow-card.md`.

**2.** Copy the template from the end of this module and paste it in. Then fill in every section based on what actually worked for you — not what the tutorial said, but what you actually did.

The card has six sections. Here is what goes in each one:

**Setup (one-time):** The exact steps to go from zero to a ready Claude environment. This is Module 0 in your own words.

**Starting a new project:** What you do at the beginning of every new project — creating the folder, writing CLAUDE.md, initialising git.

**The vibe coding loop:** Your version of the prompt → review → commit cycle. Include the rules that worked for you (one feature at a time, test before committing, etc.).

**Debugging:** The steps you follow when something breaks. Your 3-part prompt, the git revert command, the fix log habit.

**Plugins:** Which plugins you have installed and what you use them for.

**Deploy:** The exact commands and steps to go from local to live.

**3.** Write this in imperative voice — "do this, then do that" — as if you are writing instructions for a version of yourself who forgot everything. Be specific. Write the actual commands, not just "do the git thing."

### The second-project mistakes

Read these before you start your next project.

**Mistake: Skipping CLAUDE.md because "this project is small."**
Every project without CLAUDE.md becomes frustrating after session two. The project you thought was small usually isn't. Write CLAUDE.md at the start of every project, not when it starts feeling necessary.

**Mistake: Writing a big first prompt instead of a small one.**
After Module 1, you have some confidence. You write a big first prompt: "Build me a full portfolio site with a hero, projects gallery, skills section, testimonials, contact form, and dark mode." Claude produces 500 lines of code. Something is wrong. You don't know where. Small first prompt — get the scaffolding working — then add features one at a time.

**Mistake: Not committing until the end.**
"I'll commit once everything is working." Then something breaks, you can't find the last working state, and you spend an hour debugging what should have been a one-command revert. Commit after every working feature, even on small projects.

**Mistake: Using frameworks before you're ready.**
You want to try React because you heard it's powerful. You start a React project. The setup takes an hour. You hit errors you don't understand. You abandon the project. Fix: stay with plain HTML/CSS/JS until you have shipped at least five projects. Frameworks add complexity that is genuinely unnecessary until your projects outgrow plain HTML.

**Mistake: Treating Claude's first output as final.**
Claude's first attempt is a starting point, not a finished product. The vibe coding loop exists because revision is normal. Use it on every feature without feeling like something went wrong.

### What to build next

Three project ideas that work well as second projects. Each has a ready-made starter prompt below.

**Idea 1 — A portfolio page with a real projects section**
You now have a project to show. Build a proper portfolio page that lists it.

**Idea 2 — A daily habit tracker**
A simple app where you tick off daily habits and see a streak counter. Uses localStorage for persistence.

**Idea 3 — A link-in-bio page**
A single page with your name, a photo placeholder, and a list of links — for social media, your projects, your contact. More useful than a general landing page once you have things to link to.

<!-- ASSET: diagram | Three project card thumbnails side by side: Portfolio Page, Habit Tracker, Link-in-Bio — each with a starter prompt button below -->

## Practical workflow

1. Open `workflow-card.md` → review before starting any new project
2. New project: create folder → write CLAUDE.md → `git init` → initial commit → run `claude`
3. First prompt: keep it small (scaffolding only) → confirm it works → commit → add features one at a time
4. Vibe coding loop: one feature → prompt → review diff → accept → test → commit
5. When something breaks: copy full error → 3-part debug prompt → targeted fix → test → commit
6. When project is done: `git push` → Netlify auto-deploys → run pre-launch checklist → share URL
7. After completing project: update workflow card with anything that worked differently than expected

## Common mistakes

**Mistake 1: Building the workflow card as a reference and never updating it.**
The card is useful from day one, but it gets more valuable as you update it. After every project, spend five minutes noting what worked differently and updating the card. Version 5 of your workflow card — after five shipped projects — will be dramatically better than version 1.

**Mistake 2: Assuming the process works identically for bigger projects.**
The vibe coding loop works for any size project. But bigger projects need more planning before the first prompt. Before starting a complex app, write a short list of all the features you want, then break them into the order you would add them using the vibe coding loop. Do not start with the hard features — build the scaffolding first, then layer complexity.

**Mistake 3: Not sharing what you built.**
You have a live URL. Share it. Post it somewhere. The feedback — even just "this is cool" — cements the habit of finishing projects. Unshared projects feel half-real. Shared projects are real.

## Your turn

Fill in the Personal Workflow Card template. Every blank must be completed before you call this module done.

The test: give the completed card to someone who has never heard of vibe coding. Could they follow it and ship a project? If yes: your card is specific enough. If no: find the vague parts and make them concrete.

## Prompt / Template / Checklist pack

### Personal Workflow Card Template

Copy this into `workflow-card.md` in your project folder. Fill in every blank.

```markdown
# My Vibe Coding Workflow Card

Last updated: [date]
Projects shipped with this workflow: [number]

---

## 1. One-time setup (do once, never again)

- [ ] VS Code installed: [version — check with Help → About]
- [ ] Node.js installed: run `node --version` → shows [v18 or higher]
- [ ] Claude Code CLI installed: run `claude --version` → shows [version]
- [ ] Claude account: [Pro / Max] — logged in via `claude login`
- [ ] claude.ai VS Code extension: installed, publisher = Anthropic
- [ ] MCP plugins installed: `claude mcp list` shows:
  - [plugin 1] (connected)
  - [plugin 2] (connected)
  - [plugin 3] (connected)

---

## 2. Starting a new project

1. Create folder: `mkdir [project-name] && cd [project-name]`
2. Initialise git: ask Claude → "Initialise a git repo and make an initial commit"
3. Create CLAUDE.md: [describe what I include — project description, tech stack, rules]
4. Commit CLAUDE.md: `git add CLAUDE.md && git commit -m "docs: add CLAUDE.md"`
5. Run claude: `claude`

My first prompt pattern:
```
Create a [type of app] using plain HTML, CSS, and JavaScript — no frameworks.
[describe the minimal first version — just the scaffolding, not all features]
Save as index.html in the current folder, openable in any browser.
```

---

## 3. The vibe coding loop

My one-feature prompt checklist (before sending any prompt):
- [ ] Am I asking for ONE feature only?
- [ ] Did I specify: what it does + where it appears + how it looks?
- [ ] Did I specify: any data/state it needs (localStorage? Calculated?)?

Loop steps:
1. Write one-feature prompt → press Enter
2. Read Claude's explanation
3. Review diff: does it match what I asked?
4. If yes: type `y` → test in browser
5. If no: type `n` → revise prompt with more specific instructions
6. Once working: ask Claude to commit → "Commit with message 'feat: [what I added]'"

---

## 4. Debugging

When something breaks:
1. Open browser → `F12` → Console tab
2. Copy full error (including file name and line number)
3. Start `claude` → paste this prompt:

```
Error: [paste full error]
Context: [what I last changed / what was working before]
Expected behaviour: [what it should do]
Fix only this specific error. Do not change anything else.
```

4. Review Claude's fix → test in browser
5. If fix makes things worse: `git checkout -- .` → start fresh

---

## 5. Plugins I use

| Plugin | What I use it for |
|--------|------------------|
| [plugin 1] | [specific use case] |
| [plugin 2] | [specific use case] |
| [plugin 3] | [specific use case] |

Check installed plugins: `claude mcp list`

---

## 6. Deploying

**First deploy (new project):**
1. Create GitHub repo at github.com → copy the 3 setup commands
2. Run all 3 commands in terminal
3. Go to netlify.com → Add new site → Import from GitHub → select repo
4. Leave Build command empty → click Deploy site
5. Rename site: Site configuration → Change site name

**Every deploy after that:**
```
git add .
git commit -m "[description of what changed]"
git push
```
Netlify auto-deploys within ~60 seconds.

**Pre-launch check:** run the Pre-Launch Checklist from Module 6 before sharing the URL.

---

## 7. Rules I follow (updated from experience)

[Add rules here as you discover them across projects. Start with these:]
- One feature per prompt, always
- Test in browser before committing, always
- Commit after every working feature, always
- Update CLAUDE.md after every major feature, always
- Run `git checkout -- .` when fixes stack — never stack fixes on broken code

---

## 8. Notes from my projects

[After each project, add one line: what you built, what worked, what you would do differently.]
| Date | Project | What worked | What I'd change |
|------|---------|-------------|-----------------|
| [date] | [project name] | [what worked well] | [what to improve] |
```

---

### Next Project Starter Pack

Three complete starter kits — CLAUDE.md template + first prompt for each.

---

**Project 1 — Portfolio Page**

CLAUDE.md template:
```markdown
# Project: [Your Name]'s Portfolio

## What this is
A personal portfolio page showing completed projects, skills, and contact info.
Target: [recruiters / clients / general public].

## Tech stack
- Plain HTML, CSS, JavaScript — no frameworks
- index.html opens directly in a browser
- CSS in styles.css, JavaScript in script.js

## Current state
[Fill in as you build.]

## Always do
- Keep design consistent with the dark/editorial aesthetic
- Each project card must show: project name, one-sentence description, and a link

## Never do
- Never add more than 3 external font weights
- Never use placeholder images on the live version — replace with real screenshots

## Colour palette
- Background: #0F172A
- Text: #F8FAFC
- Accent: #6EE7B7
```

First prompt:
```
Create a personal portfolio page using plain HTML, CSS, and JavaScript — no frameworks.

Page sections (in order):
1. Hero: my name, one-line tagline, and two buttons ("View My Work" scrolls down, "Contact Me" is a mailto link)
2. Projects: a grid of project cards — each card shows a placeholder image (use placehold.co/400x240), project name, one-sentence description, and a "View" button
3. Contact: a simple section with an email address and links to GitHub and LinkedIn

Start with 2 placeholder project cards. I will add more later.

Dark background (#0F172A), white text (#F8FAFC), mint green accent (#6EE7B7).
Save as index.html in the current folder, openable in any browser without a server.
```

---

**Project 2 — Daily Habit Tracker**

CLAUDE.md template:
```markdown
# Project: Daily Habit Tracker

## What this is
A personal habit tracking tool. The user adds habits, ticks them off each day, 
and sees a streak counter per habit. All data is stored in localStorage.

## Tech stack
- Plain HTML, CSS, JavaScript — no frameworks
- index.html opens in browser
- All state in localStorage

## Current state
[Fill in as you build.]

## Always do
- Show streak number prominently on each habit card
- Auto-reset checkboxes at midnight using a date check in localStorage

## Never do
- Never require a login or user account
- Never add a backend or external API

## Colour palette
- Background: #1E293B
- Text: #F1F5F9
- Accent: #34D399
- Completed state: #6EE7B7 with 20% opacity background
```

First prompt:
```
Create a daily habit tracker using plain HTML, CSS, and JavaScript — no frameworks.

Features for the first version:
1. A text input and "Add Habit" button to add a new habit to the list
2. Each habit shows a checkbox, the habit name, and a streak count (number of consecutive days checked)
3. Checking a checkbox marks it as done for today
4. Streak increments by 1 each day the habit is checked, resets to 0 if a day is missed
5. All data saved to localStorage (habits and their streaks persist after page refresh)
6. A "Clear All" button at the bottom to reset everything

Dark background (#1E293B), off-white text (#F1F5F9), green accent (#34D399).
Save as index.html in the current folder, openable in any browser.
```

---

**Project 3 — Link-in-Bio Page**

CLAUDE.md template:
```markdown
# Project: Link-in-Bio Page

## What this is
A single-page link hub — my name, a short bio, a profile photo placeholder, 
and a list of links to my profiles and projects.

## Tech stack
- Plain HTML, CSS, JavaScript — no frameworks
- index.html opens directly in a browser

## Current state
[Fill in as you build.]

## Always do
- Keep the page vertically centred on mobile — this is primarily a mobile-first page
- Each link button must have a clear label and a hover state

## Never do
- Never add more than 6 links (beyond 6, visitors don't click)

## Colour palette
- Background: [your choice]
- Text: [your choice]
- Link button background: [your choice]
- Link button hover: [your choice]
```

First prompt:
```
Create a link-in-bio page using plain HTML, CSS, and JavaScript — no frameworks.

Layout (mobile-first, centred):
1. A circular profile photo placeholder (use placehold.co/120x120)
2. Name in large text
3. One-line bio (2–3 sentences)
4. A vertical stack of 4 link buttons, full-width on mobile, max 480px on desktop

Link buttons (placeholder text):
- "My Portfolio" → # (placeholder href)
- "GitHub" → #
- "LinkedIn" → #
- "Email Me" → mailto:you@email.com

Style: [describe the aesthetic you want — e.g., "minimalist with a warm cream background and dark brown text" or "bold dark background with bright green accent buttons"]

Save as index.html in the current folder. Mobile-first responsive layout.
```
