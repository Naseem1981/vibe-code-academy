# Personal Workflow Card Template
*Module 7 Deliverable — Vibe Code from Zero*

Copy this entire document. Save it as `workflow-card.md` in your project folder.
Fill in every blank. Write in your own words — the way you actually work.

---

```markdown
# My Vibe Coding Workflow Card

Last updated: [date]
Projects shipped with this workflow: [number]

---

## 1. One-time setup (do once, never again)

- [ ] VS Code installed — check: Help → About → version [paste yours]
- [ ] Node.js installed — check: `node --version` → [paste your version, e.g. v22.14.0]
- [ ] Claude Code CLI installed — check: `claude --version` → [paste your version]
- [ ] Claude account: [Pro / Max] — logged in via `claude login`
- [ ] claude.ai VS Code extension installed (publisher: Anthropic), connected to account
- [ ] MCP plugins installed — `claude mcp list` shows:
  - [plugin name] (connected)
  - [plugin name] (connected)
  - [plugin name] (connected)

---

## 2. Starting a new project

1. Create folder and navigate in:
   ```
   mkdir [project-name]
   cd [project-name]
   ```
2. Run `claude` and ask: "Initialise a git repo and make an initial commit"
3. Create CLAUDE.md — [describe what you put in yours: project description, tech stack, rules, colour palette]
4. Commit CLAUDE.md:
   ```
   git add CLAUDE.md && git commit -m "docs: add CLAUDE.md"
   ```
5. Run: `claude`

My first prompt pattern:
```
Create a [type of app] using plain HTML, CSS, and JavaScript — no frameworks.
[describe minimal first version — scaffolding only, not all features]
Save as index.html in the current folder, openable in any browser.
```

---

## 3. The vibe coding loop

Before sending any prompt, I check:
- [ ] Am I asking for ONE feature only?
- [ ] Did I specify what it does + where it appears + how it looks?
- [ ] Did I include any data/state details it needs?

My loop:
1. Write one-feature prompt → press Enter
2. Read Claude's explanation of what it plans to do
3. Review diff: does it match what I asked?
4. If yes: type `y` → test in browser
5. If no: type `n` → write a more specific revision prompt
6. Once working in browser: commit →
   ```
   Commit with message "feat: [what I added]"
   ```

Commit frequency: [describe when you commit — e.g., "after every working feature, minimum once per session"]

---

## 4. Debugging

When something breaks, I:

1. Open browser → F12 → Console tab → find red text
2. Copy full error including file name and line number
3. Start `claude` in the project folder
4. Use this prompt:
   ```
   Error: [paste full error]
   Context: [what I last changed / what was working before]
   Expected behaviour: [what it should do]
   Fix only this specific error. Do not change anything else.
   ```
5. Review Claude's fix → test in browser
6. If fix makes things worse: `git checkout -- .` → start fresh

Things I've learned to do differently: [add notes here after debugging sessions]

---

## 5. Plugins I use

| Plugin | What I use it for |
|--------|------------------|
| [plugin name] | [your specific use case] |
| [plugin name] | [your specific use case] |
| [plugin name] | [your specific use case] |

Check all plugins: `claude mcp list`
Add a plugin: `claude mcp add [name] [options]`
Remove a plugin: `claude mcp remove [name]`

---

## 6. Deploy

**First deploy (new project):**
1. Create GitHub repo at github.com → copy the 3 setup commands
2. Run the 3 commands in terminal
3. Netlify → Add new site → Import from GitHub → select repo
4. Leave Build command empty → Deploy site → wait ~30 seconds
5. Rename site: Site configuration → Change site name

**Every deploy after the first:**
```
git add .
git commit -m "[description]"
git push
```
Netlify auto-deploys. Check: Site dashboard → recent deploy status.

Pre-launch: run through Deploy Checklist from Module 6 before sharing URL.

---

## 7. My rules

[Add rules that you have discovered work for your specific way of building. Start with these:]

- One feature per prompt, always
- Test in browser before committing, always
- Commit after every working feature, always
- Write CLAUDE.md at the start of every project, always
- Update CLAUDE.md after every major feature, always
- `git checkout -- .` when fixes stack — never stack fixes on broken code
- [Add your own rules here as you discover them]

---

## 8. Project history

[Add one row per project you ship with this workflow.]

| Date | Project | What I built | What worked well | What I'd improve |
|------|---------|-------------|-----------------|------------------|
| [date] | [name] | [what it does] | [what worked] | [what to improve] |
```

---

## Next Project Starter Pack

Three project ideas with complete starter prompts. Pick one and start today.

---

**Project 1 — Portfolio Page**

Use this when: you have something to show and want a professional landing page for it.

First prompt:
```
Create a personal portfolio page using plain HTML, CSS, and JavaScript — no frameworks.

Sections (in order):
1. Hero: name, one-line tagline, two buttons ("View My Work" scrolls down, "Contact Me" is a mailto link)
2. Projects: grid of project cards — each shows a placeholder image (placehold.co/400x240), project name, one-sentence description, "View" button
3. Contact: email address, GitHub link, LinkedIn link

Start with 2 placeholder project cards.
Dark background (#0F172A), white text (#F8FAFC), mint green accent (#6EE7B7).
Save as index.html in the current folder.
```

---

**Project 2 — Daily Habit Tracker**

Use this when: you want to build something useful for your own daily life.

First prompt:
```
Create a daily habit tracker using plain HTML, CSS, and JavaScript — no frameworks.

Features for the first version:
1. Text input + "Add Habit" button — pressing Enter or clicking adds the habit to the list
2. Each habit: checkbox, habit name, streak count (consecutive days checked)
3. Checking marks it done for today — streak increments by 1 each day checked, resets to 0 if a day is missed
4. All data saved to localStorage (habits and streaks survive page refresh)
5. "Clear All" button at the bottom

Dark background (#1E293B), off-white text (#F1F5F9), green accent (#34D399).
Save as index.html in the current folder.
```

---

**Project 3 — Link-in-Bio Page**

Use this when: you want something useful to share on social media right now.

First prompt:
```
Create a link-in-bio page using plain HTML, CSS, and JavaScript — no frameworks.

Layout (mobile-first, centred):
1. Circular profile photo (placehold.co/120x120)
2. Name in large text
3. One-line bio (2–3 sentences)
4. Stack of 4 link buttons (full-width on mobile, max 480px on desktop):
   - "My Portfolio" → #
   - "GitHub" → #
   - "LinkedIn" → #
   - "Email Me" → mailto:you@email.com

Style: [describe the look you want — e.g., "minimalist, warm cream background (#FFFBEB), dark brown text (#1C1917), subtle gold buttons (#D97706)"]

Mobile-first responsive. Save as index.html in the current folder.
```
