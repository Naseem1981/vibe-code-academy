# Day 15: Portfolio Capstone + Level 1 Graduation

## Outcome artifact

By the end of this day you will have a fully deployed portfolio site at a GitHub Pages URL — showing all four Level 1 projects with screenshots, live links, and repo links, responsive on mobile, shareable with anyone — and a completed Level 1 review in your workflow card.

---

## The core idea

A portfolio site is the single most important thing you can deploy as someone who builds apps. It is proof of work. Employers, clients, and collaborators cannot evaluate skills they cannot see — a live portfolio that links to four working apps is more convincing than any resume bullet point. By the end of today you have five deployed projects to show: the portfolio itself plus the four apps you built in Level 1.

The portfolio follows a standard professional structure: hero, about, projects, skills, contact. Each section has a specific job. The hero communicates who you are and what you do in under five seconds. The projects section is the main event — each card shows a screenshot, a name, a short description, the tech used, and two buttons (View Live and View Code). The skills section lists the actual tools you learned, which signals to anyone reading it that you know what Claude Code, Cursor, GitHub, and GitHub Pages are and how to use them. The contact section makes it easy to reach you.

You will build this section by section using the same one-feature-at-a-time method you have used all week. The only new skill today is taking screenshots of your own apps for the project cards. You will do this in your browser, save the images to the project folder, and reference them in the HTML. Everything else is a combination of techniques you already know.

---

## Step-by-step walkthrough

### Step 1: Create the project folder and open Claude Code

1. Open a terminal.
2. Run:

```bash
mkdir portfolio
cd portfolio
mkdir images
```

3. Open Claude Code:

```bash
claude
```

---

### Step 2: Take screenshots of your four apps

You need one screenshot per project to use as the card image. Do this before writing any code.

1. Open each of your four live GitHub Pages URLs in Chrome:
   - Landing Page
   - Budget Tracker
   - Movie Explorer
   - Country Info App (Day 10)

2. For each app:
   - Press `F12` to open Dev Tools
   - Press `Ctrl+Shift+M` to toggle device simulation mode — set width to 1280
   - Press `Ctrl+Shift+P`, type `screenshot`, select **Capture screenshot**
   - Chrome saves the PNG to your Downloads folder automatically

3. Rename the screenshots clearly:
   - `landing-page.png`
   - `budget-tracker.png`
   - `movie-explorer.png`
   - `country-app.png`

4. Move all four PNGs into your `portfolio/images/` folder.

If any of the four apps is not yet deployed, use a placeholder for now — you can swap the image later.

---

### Step 3: Scaffold the full portfolio structure

In Claude Code:

```
Build a professional portfolio site for a web developer who builds apps with AI tools.

File: single index.html with all styles and scripts inline.

Sections (in this order):
1. Header/nav: name on the left, nav links on the right (About, Projects, Skills, Contact) — sticky, transparent bg that becomes solid on scroll
2. Hero: full-viewport-height section, name as H1, tagline "I build apps with AI", two CTA buttons: "View My Work" (scrolls to Projects) and "Contact Me" (scrolls to Contact)
3. About: short bio section — two to three sentences, left-aligned, with a simple decorative element
4. Projects: heading "What I've Built", grid of 4 project cards (2 columns desktop, 1 column mobile)
5. Skills: heading "Tools I Use", icon-free grid listing: Claude Code, Cursor, HTML/CSS/JS, GitHub, GitHub Pages, Responsive Design
6. Contact: email address placeholder, GitHub link placeholder, LinkedIn link placeholder — simple centred layout
7. Footer: copyright line

Design:
- Colour scheme: dark background #0d0d0d, white text, accent #7c3aed (violet)
- Font: system font stack — apply font-weight 800 to headings, 400 to body
- Clean, modern, no decorative clutter
- Smooth scroll behaviour on the html element
- Mobile-first responsive

Leave all project card content as placeholder text for now — I will fill it in the next step.
Do not add JavaScript beyond the scroll behaviour and nav transparency effect.
```

Open `index.html` in your browser and confirm all sections are visible before continuing.

---

### Step 4: Fill in the hero and about sections

In Claude Code:

```
Update the Hero and About sections with real content.

Hero section:
- H1: "[YOUR NAME]" — replace the placeholder with exactly this text so I can find and edit it: YOUR_NAME_HERE
- Tagline: "I build apps with AI"
- Subtext below tagline: "From idea to deployed app — no prior coding experience required."
- Button 1: "View My Work" — smooth scrolls to #projects
- Button 2: "Contact Me" — smooth scrolls to #contact

About section:
- Heading: "About Me"
- Body text (3 sentences): "I am a [LEVEL 1 GRADUATE] who learned to build web apps using AI tools in 15 days. I can take an idea from nothing to a deployed, working app using Claude Code, Cursor, and GitHub. I am now moving into Level 2 — React and Firebase."
- Add a subtle horizontal divider between About and Projects

Do not change any other sections.
```

After the update, find `YOUR_NAME_HERE` in the HTML and replace it with your actual name. Save.

---

### Step 5: Fill in the project cards

In Claude Code:

```
Fill in the four project cards in the Projects section. Each card follows this structure:
- Screenshot image (from the images/ folder, responsive, covers the card top)
- Project name (bold heading)
- Short description (1 sentence)
- "Tech used" line (plain text)
- Two buttons side by side: "View Live" and "View Code"

Card 1:
- Image: images/landing-page.png
- Name: Landing Page
- Description: A responsive product landing page built with HTML, CSS, and animation.
- Tech: HTML / CSS / JavaScript
- View Live: [LANDING_PAGE_URL] (placeholder — I will fill this in)
- View Code: [LANDING_PAGE_REPO] (placeholder — I will fill this in)

Card 2:
- Image: images/budget-tracker.png
- Name: Budget Tracker
- Description: A personal finance app that tracks income and expenses with localStorage persistence.
- Tech: HTML / CSS / JavaScript
- View Live: [BUDGET_TRACKER_URL]
- View Code: [BUDGET_TRACKER_REPO]

Card 3:
- Image: images/movie-explorer.png
- Name: Movie Explorer
- Description: A live movie search app powered by the OMDB API with favourites and a detail modal.
- Tech: HTML / CSS / JavaScript / OMDB API
- View Live: [MOVIE_EXPLORER_URL]
- View Code: [MOVIE_EXPLORER_REPO]

Card 4:
- Image: images/country-app.png
- Name: Country Info App
- Description: A country search app that fetches live data from the REST Countries API.
- Tech: HTML / CSS / JavaScript / REST Countries API
- View Live: [COUNTRY_APP_URL]
- View Code: [COUNTRY_APP_REPO]

Use placeholder hrefs (#) for all links — I will replace them manually.
Do not change any other sections.
```

After the update, manually find every `[..._URL]` and `[..._REPO]` placeholder in the HTML and replace each with the actual GitHub Pages URL and GitHub repository URL for that project. Save.

---

### Step 6: Add hover and interaction polish

In Claude Code:

```
Add interaction polish to the portfolio.

Requirements:
- Project cards: on hover, lift with a box-shadow (use rgba(124, 58, 237, 0.3) for the tint) and translate up 4px. Transition: transform 0.2s ease, box-shadow 0.2s ease.
- "View Live" and "View Code" buttons: on hover, the background fills with the accent colour #7c3aed
- Nav links: on hover, colour changes to #7c3aed with an underline
- CTA buttons in hero: solid accent for primary, outlined accent for secondary — both have hover states
- Skills grid items: on hover, a subtle left border in #7c3aed appears and the item shifts right 4px
- All transitions must use specific properties only — never transition-all

Do not add new sections or change content.
```

---

### Step 7: Make it fully responsive

In Claude Code:

```
Audit the portfolio for mobile responsiveness.

Requirements:
- Test at 375px width (iPhone SE) and 768px width (tablet)
- Hero: text should not overflow; H1 font size should reduce on small screens
- Nav: on mobile (below 768px), replace the nav links with a hamburger menu button. Clicking it toggles a full-width dropdown showing the nav links vertically.
- Projects grid: 2 columns on tablet (768px+), 1 column on mobile
- Project cards: image height should be fixed (e.g., 200px) with object-fit: cover
- About and Skills sections: full width on mobile, centred text on mobile
- Contact section: links stacked vertically on mobile
- Footer: centred on all screen sizes

Use CSS media queries only — no JavaScript for layout.
Do not change desktop styles.
```

Test by resizing your browser window down to mobile width and checking each section.

---

### Step 8: Deploy to GitHub Pages

1. Go to `https://github.com/new`, name the repo `portfolio`, set it to Public, click **Create repository**.

2. In your terminal (inside the `portfolio` folder):

```bash
git init
git add index.html images/
git commit -m "Portfolio — Level 1 capstone build"
git branch -M main
git remote add origin https://github.com/YOUR_GITHUB_USERNAME/portfolio.git
git push -u origin main
```

3. In the GitHub repo: **Settings** → **Pages** → Source: **main** branch, **/ (root)** → **Save**.
4. Wait 90 seconds, refresh the Pages settings page. Your live URL is ready: `https://YOUR_GITHUB_USERNAME.github.io/portfolio/`
5. Open the URL, test every link, test on your phone.

---

### Step 9: Generate your Level 1 summary with Claude Code

In Claude Code, run this graduation prompt:

```
I have just completed Level 1 of a vibe coding course. Generate a structured summary of what I built and what I can now do.

Projects I built:
1. Landing Page — responsive product page, HTML/CSS/JS, deployed to GitHub Pages
2. Budget Tracker — personal finance app with localStorage, deployed to GitHub Pages
3. Country Info App — fetches from REST Countries API, deployed to GitHub Pages
4. Movie Explorer — OMDB API, search, detail modal, favourites with localStorage, deployed to GitHub Pages
5. Portfolio — showcases all four projects, responsive, deployed to GitHub Pages

Tools I used: Claude Code, Cursor, HTML, CSS, JavaScript, Git, GitHub, GitHub Pages

Format the summary as:
- "What I built" (bullet list of projects with one-line descriptions)
- "What I can now do" (bullet list of concrete skills)
- "What I'm ready for next" (one paragraph)
```

Copy the output into your workflow card (the document you have been maintaining throughout the course).

---

## Practical workflow

```bash
# Create project
mkdir portfolio && cd portfolio && mkdir images

# Open Claude Code
claude

# After building
git init
git add index.html images/
git commit -m "Portfolio — Level 1 capstone build"
git branch -M main
git remote add origin https://github.com/USERNAME/portfolio.git
git push -u origin main
```

Screenshots: `F12` → `Ctrl+Shift+M` (device mode, set 1280px width) → `Ctrl+Shift+P` → "Capture screenshot" → Downloads folder → rename → move to `portfolio/images/`.

GitHub Pages: Settings → Pages → Source: main / (root) → Save → wait 90s → live URL appears.

Graduation prompt: run in Claude Code after deploying — copy output to workflow card.

---

## Common mistakes

**Mistake 1: Project card images do not appear after deploying to GitHub Pages.**
The images folder was not included in the git commit. GitHub Pages only serves files that are tracked in the repository.
Fix: In your terminal, run:

```bash
git add images/
git commit -m "Add project screenshots"
git push
```

Then wait 30 seconds and refresh the live URL.

**Mistake 2: The hamburger menu does not close after clicking a nav link.**
The JavaScript toggles the menu open but does not add a close handler for when a nav link is clicked.
Fix: In Claude Code:

```
The mobile hamburger menu opens when the button is clicked but does not close when a nav link is clicked. Add a click event listener to each nav link that closes the menu (removes the open class from the nav list). Do not change any other functionality.
```

**Mistake 3: Live project links still show `#` as the href after deploying.**
The placeholder links were not replaced before the final commit and push.
Fix: Open `index.html` locally, replace every `href="#"` that should be a real URL with the actual GitHub Pages URL or repo URL, save, then run:

```bash
git add index.html
git commit -m "Fix project links"
git push
```

---

## Your turn

Action: Complete all build steps, replace all placeholder links with real URLs, deploy to GitHub Pages, and confirm every project card links to a working app.

Expected output: `https://YOUR_USERNAME.github.io/portfolio/` loads in under 3 seconds, all four project cards show screenshots, "View Live" and "View Code" buttons open real URLs, the site is usable on mobile.

Failure state: Some project links return 404 because the linked GitHub Pages sites are not deployed yet. Fix: go back to those projects (Landing Page, Budget Tracker, etc.) and complete their GitHub Pages deployments. Each one follows the same process: create a public repo, push the code, enable Pages in Settings.

---

## Prompt / Template / Checklist pack

### Portfolio full build prompt sequence

**Prompt 1 — Full scaffold**
```
Build a professional portfolio site for a web developer who builds apps with AI tools.

Single index.html, all styles and scripts inline.

Sections: Header/nav (sticky), Hero (full viewport, name + tagline + 2 CTAs), About, Projects (4-card grid), Skills, Contact, Footer.

Design: dark #0d0d0d background, white text, #7c3aed accent, system font stack, font-weight 800 headings.

Leave project cards as placeholders. Smooth scroll on html element. Nav becomes solid on scroll.
```

**Prompt 2 — Hero and About content**
```
Update the Hero section: H1 text is YOUR_NAME_HERE (I will replace), tagline "I build apps with AI", subtext "From idea to deployed app — no prior coding experience required.", Button 1 "View My Work" scrolls to #projects, Button 2 "Contact Me" scrolls to #contact.

Update About section: heading "About Me", three-sentence bio about a developer learning to build with AI tools. Horizontal divider after About.

Do not change other sections.
```

**Prompt 3 — Project cards**
```
Fill in the four project cards. Each card: image (from images/ folder), project name, 1-sentence description, tech used line, "View Live" and "View Code" buttons with href="#" placeholders.

Cards: Landing Page (images/landing-page.png), Budget Tracker (images/budget-tracker.png), Movie Explorer (images/movie-explorer.png), Country Info App (images/country-app.png).

Do not change other sections.
```

**Prompt 4 — Hover and interaction polish**
```
Add hover effects to project cards (lift + violet shadow), CTA buttons, nav links, and skills grid items. Use specific transition properties only — never transition-all. Accent colour is #7c3aed.
```

**Prompt 5 — Mobile responsiveness**
```
Audit for mobile at 375px and 768px. Add hamburger menu for mobile nav. Projects: 2 columns at 768px+, 1 column on mobile. Hero text scales down. All sections stack correctly on mobile. CSS media queries only.
```

---

### Level 1 Graduation Checklist

Use this to confirm you have completed everything in Level 1 before starting Level 2.

**Projects built and deployed**
- [ ] Landing Page — live GitHub Pages URL: ___________________________
- [ ] Budget Tracker — live GitHub Pages URL: ___________________________
- [ ] Country Info App — live GitHub Pages URL: ___________________________
- [ ] Movie Explorer — live GitHub Pages URL: ___________________________
- [ ] Portfolio — live GitHub Pages URL: ___________________________

**Skills confirmed**
- [ ] Can open Claude Code in any folder and run prompts
- [ ] Can use Cursor Chat (`Ctrl+L`) to ask questions about code and API responses
- [ ] Can use Cursor Composer (`Ctrl+I`) to make targeted edits to existing files
- [ ] Can write CSS media queries for responsive layouts
- [ ] Can fetch data from a public API and display it on a page
- [ ] Can read and write localStorage (save and restore state across page refreshes)
- [ ] Can initialise a git repo, commit files, and push to GitHub
- [ ] Can enable GitHub Pages on a public repository
- [ ] Can take a screenshot of a live app in Chrome Dev Tools

**Workflow card**
- [ ] Workflow card updated with Level 1 summary (projects, skills, tools)
- [ ] Level 1 graduation prompt output copied into workflow card
- [ ] All five GitHub Pages URLs saved in workflow card

**Ready for Level 2**
- [ ] Node.js installed (required for React development)
- [ ] Have reviewed what React is: component-based UI framework, runs in the browser
- [ ] Have reviewed what Firebase is: cloud database + authentication platform
- [ ] Understand that Level 2 apps will have real user accounts and live data

---

### Level 2 Preview Card

**What changes in Level 2**
You stop writing plain HTML/JS files and start building with React — a component-based framework that makes large apps manageable. Every piece of UI is a reusable component. State is managed properly instead of through the DOM. Apps are compiled before they run.

**The new tools**
- React (via Vite) — the UI framework
- Firebase — authentication (user accounts) and Firestore (live database)
- Vite — the build tool that compiles your React app
- npm — the package manager (you already have this if Node is installed)

**What you will build in Level 2**
- A task manager app with real user accounts (sign up, log in, log out)
- A live collaborative app where multiple users see each other's changes in real time
- A full-stack app that combines everything: React frontend, Firebase backend, deployed to Netlify

**What stays the same**
Claude Code remains your backbone tool. Cursor remains your editor. Git and GitHub remain your version control. The one-feature-at-a-time build method stays exactly the same. The skills you built in Level 1 carry directly forward — Level 2 adds new tools on top of a foundation you already have.

**Before you start Level 2**
1. Make sure your workflow card is up to date with Level 1
2. Install Node.js from `https://nodejs.org` (LTS version) if you have not already
3. Confirm `node -v` and `npm -v` both return version numbers in your terminal
4. Review your portfolio — you will be adding Level 2 projects to it

---

## Level 1 Graduation

This is what you have now that you did not have 15 days ago.

**What you built:**
- A responsive landing page with animations, deployed live
- A budget tracker that stores data in the browser and survives page refreshes
- A country info app that fetches live data from a public API
- A movie explorer with search, a detail modal, and a favourites system
- A portfolio that shows all four of those apps to anyone on the internet

**What you can do:**
- Write a prompt that produces a complete, working HTML/CSS/JS app
- Break a build into sequential prompts and know when each one is done before starting the next
- Read an API's JSON response, understand its structure, and write a prompt that displays the data correctly
- Use localStorage to persist state across page refreshes without a database
- Style a responsive layout that works on mobile and desktop
- Use git to commit changes and push them to a remote repository
- Deploy any static app to a live URL using GitHub Pages in under five minutes
- Debug a broken app by reading the browser console and writing a targeted fix prompt
- Use Cursor Chat to understand unfamiliar code and Cursor Composer to edit specific parts of a file

**What those things add up to:**
On Day 1 you had no working apps. Today you have five deployed, live, linked projects at a URL you can hand to anyone. That is not a beginner's output. That is a working portfolio.

Level 2 starts where this ends. Same tools, same method, bigger apps.

---

*End of Level 1.*
