# Module 1: Your First Prompt, Your First App

## Outcome artifact
By the end of this module you will have a working web app running in your browser at `http://localhost:3000` — built entirely from Claude prompts, with zero manual code written by you.

<!-- ASSET: diagram | Split screen: left side shows a terminal with a Claude prompt being typed, right side shows a browser with a simple web app running -->

## The core idea

The biggest mistake beginners make with vibe coding is writing a bad first prompt. They type something like "make me an app" and get confused output. Then they assume vibe coding doesn't work.

Vibe coding works. Bad prompts don't.

A good first prompt gives Claude four things: what the app is, what it does, how it looks, and what technology to use. That last one matters — Claude Code works best when you tell it to use simple, standard technologies that don't require complex setup. For beginners, that means plain HTML, CSS, and JavaScript. No frameworks, no databases, no complicated build steps. Just files that open in a browser.

Here's the difference between a bad prompt and a good one:

**Bad prompt:**
```
make me an app
```

**Good prompt:**
```
Create a personal landing page using plain HTML, CSS, and JavaScript (no frameworks).
The page should show my name, a short bio (2 sentences), and three buttons: "My Projects", "Contact Me", and "About". 
Each button should highlight when hovered.
Use a dark background (#0F172A), white text (#F8FAFC), and a mint green accent (#6EE7B7) for the buttons.
Save everything in a folder called "my-landing-page". Include an index.html file that I can open directly in a browser.
```

The good prompt specifies: type of app (landing page), technology (plain HTML/CSS/JS), content (name, bio, three buttons), visual style (colours), file structure (folder name, index.html), and how to run it (open in browser). Claude now has everything it needs.

This module walks you through your first build using a ready-made prompt. By the end you'll understand why each part of the prompt matters and how to write your own.

## Step-by-step walkthrough

### Create your project folder

**1.** Open VS Code. Press `` Ctrl+` `` (Windows/Linux) or `` Cmd+` `` (Mac) to open the terminal at the bottom of the screen.

**2.** Navigate to where you want to create your project. If you want it on your Desktop:
- **Windows:** `cd C:\Users\YourName\Desktop`
- **Mac:** `cd ~/Desktop`
- **Linux:** `cd ~/Desktop`

**3.** Create a new folder for your project and navigate into it:
```
mkdir my-first-app
cd my-first-app
```

You're now inside your project folder. This is where Claude will create all your files.

### Start Claude Code

**4.** In the terminal (while inside `my-first-app`), type:
```
claude
```

Press Enter. Claude Code starts. You'll see a prompt:
```
> 
```

This means Claude is ready to receive your instruction.

### Give Claude your first prompt

**5.** Type or paste this prompt and press Enter:

```
Create a personal landing page using plain HTML, CSS, and JavaScript — no frameworks, no npm, no build steps. The page should show a name (use "Alex" as placeholder), a short bio (2 sentences: "I build things with AI. This is my first vibe-coded app."), and three buttons: "My Projects", "Contact Me", and "About". Each button should highlight with a mint green glow when hovered. Use a dark background (#0F172A), white text (#F8FAFC), and mint green (#6EE7B7) for button accents. Save everything in the current folder. Create an index.html file I can open directly in any browser — no server needed.
```

Claude will respond by listing the files it's about to create. You'll see something like:
```
I'll create a personal landing page with those specifications. Let me create the files:

Creating index.html...
Creating styles.css...
Creating script.js...
Done. Open index.html in your browser to see the result.
```

**6.** Claude may ask you to confirm before making changes. If you see a prompt like `Allow Claude to create these files? (y/n)`, type `y` and press Enter.

### Open your app in the browser

**7.** In VS Code, press `Ctrl+Shift+E` (Windows/Linux) or `Cmd+Shift+E` (Mac) to open the **Explorer panel** on the left. You'll see the files Claude created: `index.html`, `styles.css`, `script.js`.

**8.** Right-click `index.html` in the Explorer panel and select **Open with Default Browser**. If you don't see that option, find `index.html` in your file manager (Finder on Mac, File Explorer on Windows) and double-click it.

Your browser opens and shows your landing page — a dark page with "Alex", a short bio, and three mint-green buttons.

**9.** Hover over the buttons. They should glow. If they do: your first app works.

### Verify it's real

**10.** Right-click anywhere on the page in your browser and select **View Page Source** (or press `Ctrl+U`). You'll see the actual HTML code that makes the page work. This is real code — Claude wrote it, but it's yours.

Close the source tab. You're done.

<!-- ASSET: diagram | Three-step flow: Terminal (claude command) → VS Code Explorer (files appear) → Browser (app running) -->

## Practical workflow

1. Open VS Code → press `` Ctrl+` `` to open terminal
2. `cd` into your project folder
3. Run `claude` to start a session
4. Paste your prompt → press Enter
5. Review what Claude says it will create → type `y` to confirm
6. Wait for files to appear in Explorer panel (`Ctrl+Shift+E`)
7. Open `index.html` in browser (right-click → Open with Default Browser)
8. Verify it works → if yes, move on

## Common mistakes

**Mistake 1: Asking Claude to use React, Next.js, or other frameworks on the first app.**
These require additional setup (npm install, dev server, build process) that adds steps and failure points for beginners. Fix: always specify "plain HTML, CSS, and JavaScript — no frameworks" for your first apps. You can add complexity later once you're comfortable.

**Mistake 2: Running `claude` from the wrong folder.**
If you run `claude` from your Desktop instead of inside `my-first-app`, Claude creates files on your Desktop. Fix: always run `cd your-project-folder` before running `claude`. Confirm you're in the right place with `pwd` (Mac/Linux) or `cd` (Windows) — it prints your current location.

**Mistake 3: Typing a vague prompt and expecting Claude to fill in the gaps.**
Claude will make choices when you leave things unspecified, and they often aren't what you wanted. Fix: use the prompt template below. Fill in every field before sending.

## Your turn

Open your browser and confirm your landing page shows:
1. A name
2. Two sentences of bio text
3. Three buttons that glow when you hover over them

You should end up with a working page that looks styled — dark background, white text, coloured buttons. If you see an error in the browser (blank page, broken layout, or "Cannot GET /"), go back to the terminal, make sure you're in `my-first-app`, and check that `index.html` exists: run `ls` (Mac/Linux) or `dir` (Windows) in the terminal to list files.

## Prompt / Template / Checklist pack

### First App Prompt Template

Fill in the blanks, then paste the full prompt into Claude Code:

```
Create a [TYPE OF APP] using plain HTML, CSS, and JavaScript — no frameworks, no npm, no build steps.

The page should show:
- [MAIN CONTENT ITEM 1]
- [MAIN CONTENT ITEM 2]
- [MAIN CONTENT ITEM 3]

Visual style:
- Background colour: [HEX CODE, e.g. #0F172A]
- Text colour: [HEX CODE, e.g. #F8FAFC]
- Accent colour: [HEX CODE, e.g. #6EE7B7]
- Font feel: [e.g. "clean and modern", "bold and dramatic", "warm and friendly"]

Save everything in the current folder. Create an index.html file I can open directly in any browser — no server needed.
```

### 5 Ready-to-Use Starter Prompts

**Prompt 1 — Personal Landing Page:**
```
Create a personal landing page using plain HTML, CSS, and JavaScript — no frameworks, no npm, no build steps. Show my name (use "Alex" as placeholder), a 2-sentence bio ("I build things with AI. This is my first vibe-coded app."), and three buttons: "My Projects", "Contact Me", "About". Buttons should glow on hover. Dark background (#0F172A), white text (#F8FAFC), mint accent (#6EE7B7). Save as index.html in the current folder, openable in any browser without a server.
```

**Prompt 2 — Simple Calculator:**
```
Create a calculator app using plain HTML, CSS, and JavaScript — no frameworks. It should do basic maths: add, subtract, multiply, divide. Show a display at the top for the current number, and a grid of buttons for digits 0-9, the four operations, equals, and clear. Style it with a dark charcoal background (#1E293B), white digits, and blue accent (#38BDF8) for the operation buttons. Save as index.html in the current folder, openable in any browser without a server.
```

**Prompt 3 — Quote Generator:**
```
Create a random quote generator using plain HTML, CSS, and JavaScript — no frameworks, no API calls. Include 10 motivational quotes hardcoded in the JavaScript. Show one quote at a time with the author's name below. Add a "New Quote" button that picks a random quote. Add a fade animation when switching quotes. Cream background (#FFFBEB), dark brown text (#1C1917), gold accent (#D97706) for the button. Save as index.html in the current folder.
```

**Prompt 4 — To-Do List:**
```
Create a to-do list app using plain HTML, CSS, and JavaScript — no frameworks. Allow the user to type a task and press Enter or click "Add" to add it to the list. Each task should have a checkbox (clicking it strikes through the text and fades it) and a delete button (X). Show a count of remaining tasks at the top. Use local storage so tasks survive a page refresh. Dark background (#0F172A), white text (#F8FAFC), purple accent (#A78BFA). Save as index.html in the current folder.
```

**Prompt 5 — Countdown Timer:**
```
Create a countdown timer using plain HTML, CSS, and JavaScript — no frameworks. Show three inputs: hours, minutes, seconds. A "Start" button begins the countdown. A "Pause" button pauses it. A "Reset" button clears it. When the timer reaches zero, flash the display red and show "Time's up!" Display the remaining time in large, bold digits (HH:MM:SS format). Dark background (#0F172A), white text (#F8FAFC), orange accent (#F97316). Save as index.html in the current folder.
```
