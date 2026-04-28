# CLAUDE.md Starter Templates
*Module 3 Deliverable — Vibe Code from Zero*

Three ready-to-use CLAUDE.md templates. Copy the one that matches your project. Fill in every blank — do not leave placeholders in the live file.

---

## Template 1 — Web App (plain HTML/CSS/JS)

```markdown
# Project: [App Name]

## What this is
[What the app does in 1-2 sentences. Include who uses it and what problem it solves.]

## Tech stack
- Plain HTML, CSS, JavaScript — no frameworks, no npm, no build steps
- Single index.html opens directly in a browser without a server
- CSS in styles.css, JavaScript in script.js

## Current state
[List working features as bullet points. Update this section after every major feature.]
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

## Template 2 — Personal Landing Page

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
[Fill in as features are built. Example:]
- Hero section with name and tagline
- Three navigation buttons
- [Continue listing...]

## Always do
- Keep the design consistent with the existing visual aesthetic
- Make all new sections responsive (works correctly on mobile and desktop)
- Use semantic HTML elements: <section>, <header>, <footer>, <nav>

## Never do
- Never add more than one external font family
- Never use stock placeholder images in the final version — leave space for real images
- Never change the colour palette without being explicitly asked

## Colour palette
- Background: [hex]
- Text: [hex]
- Accent: [hex]
```

---

## Template 3 — Personal Tool (calculator, timer, tracker, converter, etc.)

```markdown
# Project: [Tool Name]

## What this is
[What the tool does. What problem it solves. Who uses it. One paragraph.]

## Tech stack
- Plain HTML, CSS, JavaScript — no frameworks
- index.html opens directly in a browser
- State managed with JavaScript variables (no external state library)
- [Data persistence: localStorage / none]

## Current state
[Describe what the tool currently does — its current functionality.]
- [Main functionality]
- [UI elements present]
- [Data persistence approach]

## Always do
- Show the user the result of every action immediately (no delays)
- Validate user input before processing — show a clear error message if input is invalid
- Preserve all existing functionality when adding new features

## Never do
- Never add a backend, API, or external data source
- Never require the user to refresh the page to see updated results
- Never break existing features when adding new ones

## Colour palette
- Background: [hex]
- Text: [hex]
- Button/accent: [hex]
- Error colour: [hex — e.g. #EF4444]
- Success/complete colour: [hex]
```

---

## CLAUDE.md Quality Checklist

Before saving your CLAUDE.md, tick every box:

- [ ] Project description answers "what is this app and what does it do?"
- [ ] Tech stack lists all technologies in use — and explicitly names what is NOT used
- [ ] Current state lists every working feature (nothing missing, nothing planned)
- [ ] "Always do" rules are specific enough that Claude could follow them without asking you
- [ ] "Never do" rules would prevent Claude's most likely bad decisions for this project
- [ ] Colour palette lists exact hex values — no "dark blue" or "mint green"
- [ ] File is saved in the project root, named exactly `CLAUDE.md` (capitals matter)
- [ ] Tested: new `claude` session correctly describes the project without prompting
