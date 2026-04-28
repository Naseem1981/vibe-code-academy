# Module 3: Context Management — Teaching Claude Your Project with CLAUDE.md

## Outcome artifact
By the end of this module you will have a completed CLAUDE.md file in your project root that Claude reads at the start of every session — confirmed by starting a new Claude Code session, seeing Claude acknowledge the file, and receiving a correct response without re-briefing.

<!-- ASSET: diagram | Split screen: left shows a CLAUDE.md file open in VS Code, right shows a Claude terminal session reading "Reading CLAUDE.md..." and responding correctly to a project question -->

## The core idea

Every time you run `claude` in a new terminal session, Claude starts fresh. It has no memory of what you built last time, what rules you gave it, or what the project is supposed to do. Without CLAUDE.md, you re-explain everything every session. With CLAUDE.md, you write it down once and Claude reads it automatically at the start of every session.

CLAUDE.md is a plain text file that lives in your project's root folder — right next to `index.html`. When Claude Code starts in that folder, it reads CLAUDE.md first, before doing anything else. This gives Claude the context it needs to work correctly without you having to paste it into every prompt.

Here is what goes into a CLAUDE.md that actually works:

**Project description.** One paragraph describing what the app is and what it does. Not what you want to build in the future — what it is right now.

**Tech stack.** The exact technologies in use. "Plain HTML, CSS, and JavaScript. No frameworks. No npm. No build tools. index.html opens directly in a browser." This stops Claude from suggesting React or installing packages.

**Rules — what Claude must always do.** Things like: "Always test changes in index.html." "Always use the existing colour variables." "Always keep the file structure simple — one index.html, one styles.css, one script.js."

**Rules — what Claude must never do.** "Never introduce a build step." "Never use inline styles — all CSS goes in styles.css." "Never change the colour scheme without being asked."

**Current state of the project.** A short description of what's already built. "The app has a dark landing page with a name, bio, three buttons, a dark/light mode toggle, a footer, and a back to top button. All features are working."

The combination of these five things gives Claude enough context to work correctly from the first prompt of any session. A session that used to start with five minutes of re-briefing now starts immediately.

One important thing to understand: CLAUDE.md is a living document. You update it when the project grows. When you add a new major feature, add it to the "current state" section. When you discover a rule Claude keeps breaking, add it to the "never do" section. Think of it as instructions you maintain for a contractor who starts fresh every day.

## Step-by-step walkthrough

### Create the CLAUDE.md file

**1.** Open VS Code. Make sure you are in your `my-first-app` project folder — you should see `index.html`, `styles.css`, and `script.js` in the Explorer panel (`Ctrl+Shift+E`).

**2.** In the Explorer panel, right-click in the empty space below your files and select **New File**. Name it exactly:
```
CLAUDE.md
```

The capitalisation matters. Claude Code looks for `CLAUDE.md` (all caps). A file named `claude.md` or `Claude.md` may not be picked up on case-sensitive systems.

**3.** Click the new `CLAUDE.md` file to open it in the editor.

### Write your CLAUDE.md

**4.** Copy and paste the template below into CLAUDE.md. Then fill in the blanks for your project. A filled-in example follows the template.

**Template:**
```markdown
# Project: [Your Project Name]

## What this is
[One paragraph: what the app is, what it does, who it's for.]

## Tech stack
- HTML, CSS, JavaScript — no frameworks, no npm, no build steps
- Single index.html file that opens directly in a browser
- Styles in styles.css
- JavaScript in script.js

## Current state
[List the features that are already built and working.]
- [Feature 1]
- [Feature 2]
- [Feature 3]

## Always do
- [Rule 1]
- [Rule 2]

## Never do
- [Rule 1]
- [Rule 2]

## Colour palette
- Background: #0F172A
- Text: #F8FAFC
- Accent: #6EE7B7
```

**5.** Here is a fully filled-in CLAUDE.md for the landing page from Module 1:

```markdown
# Project: Alex's Personal Landing Page

## What this is
A personal landing page for Alex — a dark-themed single page showing a name, 
a short bio, and three navigation buttons. Built as a first vibe-coded app to 
practice using Claude Code.

## Tech stack
- Plain HTML, CSS, and JavaScript — no frameworks, no npm, no build steps
- index.html opens directly in any browser — no server needed
- All styles in styles.css
- All JavaScript in script.js

## Current state
- Dark landing page with name "Alex" and 2-sentence bio
- Three mint-green buttons: "My Projects", "Contact Me", "About"
- Dark/light mode toggle button (top-right corner, saves preference to localStorage)
- Footer with copyright line and mint-green top border
- Back to top button (appears after 300px scroll, fades in/out)

## Always do
- Keep file structure simple: index.html, styles.css, script.js only
- Use the existing colour variables for all new elements
- Test every change by opening index.html in a browser

## Never do
- Never add a build step, package.json, or npm dependency
- Never use inline styles — all CSS goes in styles.css
- Never change the colour scheme unless explicitly asked

## Colour palette
- Background: #0F172A (deep navy)
- Text: #F8FAFC (off-white)
- Accent: #6EE7B7 (mint green)
- Light mode background: #F8FAFC
- Light mode text: #0F172A
```

**6.** Save the file: `Ctrl+S`.

### Test that Claude reads CLAUDE.md

**7.** Close any open Claude Code session. In the terminal, make sure you are in `my-first-app`:
```
pwd
```
(Mac/Linux shows the path; on Windows run `cd` with no arguments.) You should see your `my-first-app` folder path.

**8.** Start a fresh Claude session:
```
claude
```

**9.** Without explaining anything about your project, type:
```
What is this project and what tech stack does it use?
```

Press Enter.

Claude should respond with information pulled directly from your CLAUDE.md — the project description, HTML/CSS/JS stack, and no-frameworks rule. If Claude accurately describes your project without you telling it anything, CLAUDE.md is working.

**10.** Test the rules. Ask Claude to do something your CLAUDE.md says never to do:
```
Add a React component for the footer.
```

Claude should push back — something like: "This project uses plain HTML, CSS, and JavaScript with no frameworks. I'll keep the footer as a plain HTML element instead." That response means Claude read your rules and is following them.

### Commit CLAUDE.md

**11.** Ask Claude to commit the file:
```
Add CLAUDE.md to git and commit it with the message "docs: add CLAUDE.md with project context and rules"
```

CLAUDE.md is now part of your project's git history. Every future session starts from this context.

<!-- ASSET: diagram | Before/after: left side shows a long back-and-forth of re-explaining the project in chat, right side shows one "What is this project?" question with an instant correct answer — CLAUDE.md bridge connecting them -->

## Practical workflow

1. Open VS Code → `` Ctrl+` `` → navigate to project folder
2. If CLAUDE.md doesn't exist: create it in the project root (`New File` → `CLAUDE.md`)
3. Fill in: project description, tech stack, current state, always/never rules, colour palette
4. Save → start `claude` in terminal
5. Test: ask "What is this project?" without explaining → verify Claude answers correctly
6. After adding major features: update `## Current state` section in CLAUDE.md
7. After discovering a rule Claude keeps breaking: add it to `## Never do`
8. After major updates: commit CLAUDE.md → `git add CLAUDE.md && git commit -m "docs: update CLAUDE.md"`

## Common mistakes

**Mistake 1: Writing CLAUDE.md as a wish list instead of a current-state document.**
"I want to add a portfolio section, a contact form, and animations" is not useful context for Claude. It tells Claude what you want, not what exists. Fix: CLAUDE.md describes what is already built, not what you plan to build. Future features go in your prompts, not in CLAUDE.md.

**Mistake 2: Never updating CLAUDE.md after the project grows.**
You add six more features, but CLAUDE.md still describes the project as it was in Module 1. Claude reads it, thinks the project is simpler than it is, and starts making suggestions that conflict with what you've already built. Fix: treat CLAUDE.md like documentation. After every significant session, spend two minutes updating the current state section. It takes ninety seconds and saves fifteen minutes of confused Claude behaviour.

**Mistake 3: Naming the file incorrectly.**
`claude.md`, `CLAUDE.MD`, `Claude.md`, or `claude_md.txt` will not be automatically read by Claude Code. Fix: the file must be named exactly `CLAUDE.md`. Use VS Code's rename function (right-click the file → Rename) to fix it if you named it wrong.

## Your turn

Create `CLAUDE.md` in your `my-first-app` folder. Use the filled-in example above as your starting point — update it with the actual features your app has after Module 2.

Start a new Claude session and ask: **"What is this project and what rules do you follow when working on it?"**

You should get a response that accurately describes your app, names the tech stack (plain HTML/CSS/JS, no frameworks), and lists your rules. If the response is accurate and Claude didn't need you to explain anything: CLAUDE.md is working. Move on to Module 4.

If Claude says it doesn't have context, check: are you running `claude` from inside `my-first-app`? Is the file named exactly `CLAUDE.md`? Run `ls` (Mac/Linux) or `dir` (Windows) to confirm both.

## Prompt / Template / Checklist pack

### CLAUDE.md Starter Templates

Three ready-to-use templates. Copy the one that matches your project, fill in the blanks.

---

**Template 1 — Web App (plain HTML/CSS/JS)**

```markdown
# Project: [App Name]

## What this is
[What the app does in 1-2 sentences. Include who uses it.]

## Tech stack
- Plain HTML, CSS, JavaScript — no frameworks, no npm, no build steps
- Single index.html opens directly in a browser without a server
- CSS in styles.css, JavaScript in script.js

## Current state
[List working features as bullet points.]
- [Feature 1]
- [Feature 2]

## Always do
- Keep the file structure flat: index.html, styles.css, script.js only
- Use existing colour variables for all new elements
- Test every change in the browser before committing

## Never do
- Never add npm packages, build tools, or a package.json
- Never use inline styles — all CSS goes in styles.css
- Never change the colour scheme unless explicitly asked

## Colour palette
- Primary background: [hex]
- Primary text: [hex]
- Accent colour: [hex]
```

---

**Template 2 — Personal Landing Page**

```markdown
# Project: [Name]'s Personal Landing Page

## What this is
A personal landing page for [Name]. Shows [content: bio, links, projects, etc.].
Target audience: [who visits — recruiters, clients, friends, general public].

## Tech stack
- Plain HTML, CSS, JavaScript — no frameworks, no npm, no build steps
- Opens directly in a browser — no server required
- All in index.html, styles.css, script.js

## Current state
- [Hero section with name and tagline / bio text]
- [Navigation buttons / links]
- [Other completed sections]

## Always do
- Keep the design consistent with the existing aesthetic
- Make all new sections responsive (works on mobile and desktop)
- Use semantic HTML elements (<section>, <header>, <footer>, <nav>)

## Never do
- Never add more than one external font
- Never use stock hero images — use the existing placeholder or leave space for a real photo
- Never change the colour palette without being explicitly asked

## Colour palette
- Background: [hex]
- Text: [hex]
- Accent: [hex]
```

---

**Template 3 — Personal Tool (calculator, timer, tracker, etc.)**

```markdown
# Project: [Tool Name]

## What this is
[What the tool does. What problem it solves. One paragraph.]

## Tech stack
- Plain HTML, CSS, JavaScript — no frameworks
- index.html opens directly in a browser
- State managed with JavaScript variables (no external state library)

## Current state
[Describe what the tool currently does — its current functionality.]
- [Main functionality]
- [UI elements present]
- [Any data persistence — localStorage yes/no]

## Always do
- Show the user the result of every action immediately (no delays)
- Validate user input before processing it — show a clear error message if input is invalid
- Preserve all existing functionality when adding new features

## Never do
- Never add a backend, API, or external data source
- Never require the user to refresh the page to see results
- Never break existing features when adding new ones

## Colour palette
- Background: [hex]
- Text: [hex]
- Button/accent: [hex]
- Error colour: [hex, e.g. #EF4444]
```

---

### CLAUDE.md Quality Checklist

Before you close CLAUDE.md, tick every box:

- [ ] Project description answers "what is this app and what does it do?"
- [ ] Tech stack lists all technologies in use — and explicitly names what is NOT used
- [ ] Current state lists every working feature (nothing missing, nothing planned)
- [ ] "Always do" rules are specific enough that Claude could follow them without asking you
- [ ] "Never do" rules would prevent Claude's most common bad decisions for this project
- [ ] Colour palette lists exact hex values — no "dark blue" or "mint green" (be precise)
- [ ] File is saved in the project root, named exactly `CLAUDE.md`
- [ ] Tested: new Claude session correctly describes the project without prompting
