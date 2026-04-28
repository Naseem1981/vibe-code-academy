# Day 2: Meet Bolt.new — Your AI App Generator

## Outcome artifact
By the end of this day you will have a Bolt.new account set up and a live personal profile card app running in Bolt's browser preview, downloaded to your machine, and opened in VS Code.

---

## The core idea

Bolt.new is a browser-based AI app builder. You open a tab at bolt.new, type a description of the app you want, and Bolt generates a complete, runnable project — file structure, components, styles, and a live preview — directly in your browser. No terminal. No `npm install`. No local environment to configure. You go from zero to a working app in under three minutes.

Claude Code CLI is different. It runs in your terminal and operates on files that already exist on your machine. You give it a task, it reads your codebase, writes and edits files, runs commands, and builds features step by step. It is precise, incremental, and deeply aware of your project's context. It is the right tool once a project exists and you are building features into it.

The workflow that makes both tools powerful is using them in sequence, not in competition. Use Bolt.new to generate the project skeleton in the first ten minutes — get the folder structure, the base components, the routing, and the initial styles done instantly. Then download that project, open it in VS Code, and use Claude Code to build the actual features with real logic, database connections, and production-grade code. Bolt handles the blank-page problem. Claude Code handles the complexity. Together, they eliminate the two biggest time-wasters in development: setup paralysis and slow feature iteration.

---

## Step-by-step walkthrough

### Step 1 — Create your Bolt.new account

1. Open your browser and go to `https://bolt.new`.
2. Click **Sign Up** in the top-right corner.
3. Sign up with your GitHub account (recommended) or with an email address.
4. Bolt will redirect you to your dashboard. You will see a large text prompt box in the centre of the screen.

You sign in with GitHub because it makes exporting to GitHub repositories easier later in the course.

---

### Step 2 — Start a new project

1. On the Bolt dashboard, click **New Project** or click directly inside the prompt box at the top of the page.
2. The prompt box is the only input you need. Bolt does not have a project-type wizard or a template selector on the free plan — everything starts from a prompt.

---

### Step 3 — Type the starter prompt

Copy this prompt exactly and paste it into the Bolt.new prompt box:

```
Build a personal profile card app using React and Tailwind CSS.

The app should display:
- A circular avatar image (use a placeholder image from https://placehold.co/120x120)
- A full name displayed as a large heading
- A short bio paragraph (2-3 sentences of placeholder text)
- Four social link buttons: GitHub, Twitter, LinkedIn, and Website
- Each social button should have an icon and open a link (use # as the href for now)

Design requirements:
- Centered card layout, max width 400px
- White card on a light grey background
- Subtle drop shadow on the card
- Rounded corners on the card
- Hover effect on each social button

Use useState to make the name and bio editable — clicking on them should turn them into an input field. Pressing Enter saves the change.
```

Press **Enter** or click the **Send** button.

---

### Step 4 — Read the Bolt interface while it generates

Bolt will spend 15–30 seconds generating your project. While it works, learn the four panels:

**Left panel — File tree.** This shows every file in the project. You will see folders like `src/`, `src/components/`, and files like `App.tsx`, `index.css`, and `package.json`. Click any file to open it in the editor.

**Centre panel — Code editor.** This is where the generated code lives. It looks like a simplified VS Code. You can read and edit code here directly, though for anything beyond a one-line fix you should prompt Bolt instead.

**Right panel — Preview.** This is a live, running version of your app. It updates automatically every time Bolt makes a change. Resize the preview pane by dragging the divider.

**Bottom panel — Terminal.** Bolt runs a real Node.js environment in the browser. This panel shows the output of `npm install` and the dev server. If the app fails to compile, the error appears here.

---

### Step 5 — Open the project in StackBlitz

At the top of the Bolt interface, click the button labelled **Open in StackBlitz**.

StackBlitz is a full VS Code environment running inside your browser. When you click that button, your project opens in StackBlitz with:
- The complete VS Code interface (sidebar, tabs, terminal, extensions panel)
- A live dev server already running
- The ability to install npm packages via the integrated terminal
- File editing with full IntelliSense and TypeScript support

StackBlitz is useful when you want to make quick edits without downloading the project. It is also where Bolt sends your project when you want to go deeper than the Bolt editor allows.

---

### Step 6 — Edit a file in StackBlitz

1. In the StackBlitz file tree (left sidebar), click `src/App.tsx`.
2. Find the placeholder name (it will be something like `"Jane Doe"` or similar).
3. Change it to your own name directly in the editor.
4. Press `Ctrl + S` (Windows) or `Cmd + S` (Mac) to save.
5. Watch the preview panel on the right update immediately.

This is the same workflow as editing in VS Code locally — the only difference is that the environment runs in your browser tab.

---

### Step 7 — Download the project to your machine

You want the project locally so you can open it in your real VS Code and use Claude Code on it.

**In Bolt.new** (go back to your Bolt tab):
1. Click the three-dot menu (`...`) in the top-right corner of the Bolt interface.
2. Click **Download Project**.
3. Bolt packages the project as a `.zip` file and downloads it to your machine.

**In StackBlitz** (if you are working there instead):
1. Open the hamburger menu in the top-left corner.
2. Click **Download Project**.
3. Save the `.zip` file.

The `.zip` will have a name like `bolt-project-XXXXXXXX.zip`. It contains the complete project including `package.json`, all source files, and config files.

---

### Step 8 — Open the downloaded project in VS Code

1. Find the downloaded `.zip` file in your Downloads folder.
2. Right-click it and select **Extract All** (Windows) or double-click to unzip (Mac).
3. Move the extracted folder to a location you use for projects (e.g. `D:/Projects/profile-card` on Windows or `~/Projects/profile-card` on Mac).
4. Open VS Code.
5. Click **File** → **Open Folder** and select the extracted folder.
6. VS Code will open the project. You will see the file tree in the **Explorer** panel on the left.
7. Open the integrated terminal: press `` Ctrl + ` `` (backtick) on Windows/Mac.
8. In the terminal, run:

**Directory:** the root of your extracted project (where `package.json` lives)
```bash
npm install
```

**Expected output:**
```
added 243 packages, and audited 244 packages in 12s
found 0 vulnerabilities
```

9. Then run the dev server:

**Directory:** same project root
```bash
npm run dev
```

**Expected output:**
```
  VITE v5.x.x  ready in 312 ms

  ➜  Local:   http://localhost:5173/
  ➜  Network: use --host to expose
```

10. Open `http://localhost:5173` in your browser. You will see your profile card app running locally.

Your project now exists as a real local project. You can close the browser-based Bolt/StackBlitz tabs and work entirely in your local VS Code from this point.

---

## Practical workflow

Use this as your reference during any future Bolt session.

1. Open bolt.new in a browser tab.
2. Write a precise prompt describing what you want — name the framework, name the libraries, name the features.
3. Press Enter and wait 15–30 seconds for generation.
4. Read the preview panel — confirm the app renders without errors.
5. Check the terminal panel for any compile errors.
6. If something is wrong, type a follow-up prompt describing exactly what to fix.
7. When the app looks correct, click the three-dot menu → Download Project.
8. Unzip the download to your projects folder.
9. Open the folder in VS Code.
10. Run `npm install` in the integrated terminal.
11. Run `npm run dev` to confirm it works locally.
12. You are ready to hand off to Claude Code.

---

## Common mistakes

**Mistake 1 — Signing up with email instead of GitHub, then not being able to export to GitHub later.**

The GitHub OAuth connection is set during signup. If you signed up with email, go to your Bolt account settings and link your GitHub account under the **Integrations** or **Connected Accounts** section before you need to export.

**Mistake 2 — Editing code directly in Bolt's editor instead of prompting.**

Bolt's code editor is functional but the app is regenerated when you prompt again, which can overwrite manual edits. For any change larger than fixing a single typo — change the text of a heading, swap a colour, add a section — write a prompt instead. Keep manual edits to absolute minimum (one-liners only) and always write them after your last prompt, not before.

**Mistake 3 — Skipping `npm install` after downloading and then seeing "module not found" errors.**

The `.zip` download does not include the `node_modules` folder (it would be hundreds of megabytes). You must run `npm install` yourself after unzipping. If VS Code shows red underlines on imports or the `npm run dev` command crashes with `Cannot find module`, run `npm install` in the terminal first.

---

## Your turn

Open bolt.new right now. Paste in the starter prompt from Step 3. Let it generate. Confirm the preview shows a profile card with an avatar, name, bio, and social buttons. Then download the project, unzip it, open it in VS Code, run `npm install`, and run `npm run dev`. Open `http://localhost:5173` in your browser.

**Expected output:** A profile card app renders in the browser with a circular placeholder avatar, a name, a bio paragraph, and four social link buttons with hover effects. The name and bio are click-to-edit.

**Failure state 1:** The Bolt preview shows a blank white screen. Fix: check the terminal panel at the bottom of Bolt for a red error message. Copy the error text, type a new prompt in Bolt: `"Fix this error: [paste the error]"`. Bolt will diagnose and fix it.

**Failure state 2:** After downloading and running `npm run dev`, the terminal shows `Error: Cannot find module 'react'`. Fix: you skipped `npm install`. Run it now from the project root. The `npm install` command must always come before `npm run dev` in a fresh project.

**Failure state 3:** The preview port `http://localhost:5173` shows "This site can't be reached". Fix: the dev server is not running. Go back to the VS Code terminal and check whether `npm run dev` is still active. If the terminal shows a command prompt (`$` or `>`) instead of the Vite output, the server stopped — run `npm run dev` again.

---

## Prompt / Template / Checklist pack

Five complete Bolt.new starter prompts. Copy any one and paste directly into bolt.new.

---

### Prompt 1 — Personal profile card

```
Build a personal profile card app using React and Tailwind CSS.

The app should display:
- A circular avatar image (use a placeholder from https://placehold.co/120x120)
- A full name as a large heading
- A short bio paragraph (2-3 sentences of placeholder text)
- Four social link buttons: GitHub, Twitter, LinkedIn, Website
- Each button has an icon and uses # as the href

Design:
- Centered card, max width 400px
- White card on a light grey background
- Subtle drop shadow and rounded corners
- Hover effect on each social button

Use useState to make the name and bio click-to-edit. Pressing Enter saves the change.
```

---

### Prompt 2 — Marketing landing page

```
Build a SaaS product landing page using React and Tailwind CSS.

Sections:
1. Hero: large headline, one-line subheading, two CTA buttons (primary and secondary), hero image placeholder (https://placehold.co/800x400)
2. Features: three feature cards in a row, each with an icon, heading, and two-sentence description
3. Pricing: two pricing tiers side by side (Free and Pro), each listing four features and a CTA button
4. Footer: logo text, three nav links, copyright line

Design:
- Dark navy background (#0f172a), white text
- Bright teal accent (#14b8a6) for buttons and highlights
- No gradients except a subtle one on the hero background
- Fully responsive — single column on mobile
```

---

### Prompt 3 — Todo app

```
Build a Todo app using React and Tailwind CSS.

Features:
- Text input at the top to add a new task, with an Add button
- Task list below showing all tasks
- Each task has a checkbox to mark complete, the task text, and a delete button
- Filter tabs at the top: All, Active, Completed
- Task count displayed: "X tasks remaining"
- Tasks stored in useState (no database)

Design:
- White card on a light grey background, centered, max width 500px
- Completed tasks show with strikethrough text and grey colour
- Smooth transition when tasks are marked complete
```

---

### Prompt 4 — Weather widget

```
Build a weather widget using React and Tailwind CSS.

Features:
- City name input with a Search button
- Fetches live weather from the Open-Meteo API (no API key required)
- Step 1: call https://geocoding-api.open-meteo.com/v1/search?name=CITY&count=1 to get latitude and longitude
- Step 2: call https://api.open-meteo.com/v1/forecast?latitude=LAT&longitude=LON&current=temperature_2m,wind_speed_10m,weather_code
- Display: city name, current temperature in Celsius, wind speed in km/h, weather description
- Weather code mapping: 0=Clear sky, 1-3=Partly cloudy, 45-48=Fog, 51-67=Rainy, 71-77=Snowy, 80-82=Rain showers, 95=Thunderstorm
- Loading spinner while fetching, error message if city not found

Design:
- Gradient background from sky blue to deep blue
- White card for the results, rounded, with shadow
- Large temperature display (48px font)
- Responsive, works on mobile
```

---

### Prompt 5 — Blog homepage

```
Build a blog homepage using React and Tailwind CSS.

Sections:
1. Header: blog name "The Daily Read", navigation links (Home, Articles, About, Newsletter)
2. Featured post: large card at the top with a placeholder image (https://placehold.co/800x400), post title, author name, date, and 2-sentence excerpt
3. Article grid: six article cards in a 3-column grid (2-column on tablet, 1-column on mobile), each with a thumbnail (https://placehold.co/400x250), category tag, title, and date
4. Sidebar: next to the grid — top 5 tags list and a newsletter signup input with a Subscribe button
5. Footer: copyright and three social icons

Design:
- Off-white background (#fafaf9), dark charcoal text (#1c1917)
- Rust/terracotta accent (#c2410c) for tags and links
- Clean serif font for headings via Google Fonts (Merriweather), sans-serif for body (Inter)
- Load both fonts via a @import in the CSS
```
