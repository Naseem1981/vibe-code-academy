# Day 6: Cursor — Your AI-Powered Editor

## Outcome artifact
By the end of this day you will have Cursor installed, connected to the Claude model, and used to add one new feature to your Day 2 landing page using Composer — with the diff reviewed and accepted.

<!-- ASSET: diagram | Split screen: left shows Cursor open with the landing page files in the sidebar, right shows the Composer panel with a diff highlighted in green — an "Accept" button visible at the bottom -->

## The core idea

Cursor is a code editor — the same kind of tool as VS Code — but with AI built into every part of it. It is not a replacement for Claude Code. It is a second, complementary tool that handles a specific layer of your work: the in-editor layer. When you want to ask a question about a specific line of code, get a suggestion as you type, or make a focused change to one function, Cursor is faster and more precise than switching to the terminal and running a full Claude Code session.

The reason Cursor feels familiar immediately is that it is a fork of VS Code. That means it starts as an exact copy of VS Code and adds AI on top. When you install Cursor, it offers to import your existing VS Code settings — themes, keybindings, extensions — so you land in a familiar environment. All the VS Code shortcuts you already know work identically. Everything you did in VS Code on Days 1–5 works in Cursor with zero relearning. The only addition is AI.

Cursor has three AI features you will use throughout this course. **Tab autocomplete** watches what you are typing and predicts the next line or block — you press `Tab` to accept the suggestion. It is faster than copy-pasting snippets and learns the patterns in your specific project. **Chat** opens a side panel where you ask questions about your code — highlight a function, press `Ctrl+L` (Windows/Linux) or `Cmd+L` (Mac), and ask "what does this do?" or "why is this variable undefined?" — Claude answers with full context of the file. **Composer** is for multi-file edits: you describe a change, Claude generates a diff across all affected files, and you review and accept or reject each change before anything is saved. Composer is the Cursor feature you will use most heavily for actual building.

The relationship between Cursor and Claude Code is straightforward: **Claude Code handles the big picture, Cursor handles the detail work.** Use Claude Code when you are building a new feature from scratch, setting up a project, writing a CLAUDE.md, managing git, or installing plugins — it operates across your whole project and handles complex multi-step tasks well from the terminal. Use Cursor when you are inside a file and want to iterate quickly: fix one function, refactor a section, understand unfamiliar code, or add a small feature to an existing page. Think of Claude Code as your architect and Cursor as your precision editor. Both tools use Claude as the underlying AI model — you are not switching AI, you are switching the interface that fits the task.

## Step-by-step walkthrough

### Install Cursor

**1.** Open your browser and go to:
```
https://cursor.com
```

**2.** Click the **Download** button. Cursor detects your operating system and offers the correct installer:
- **Windows:** downloads a `.exe` installer
- **Mac:** downloads a `.dmg` file
- **Linux:** downloads an `.AppImage` or `.deb` package

**3.** Run the installer:
- **Windows:** double-click the `.exe` → click **Install** → wait for it to finish → click **Launch Cursor**
- **Mac:** open the `.dmg` → drag **Cursor** to the **Applications** folder → open it from Applications
- **Linux:** make the AppImage executable (`chmod +x cursor-*.AppImage`) → double-click to run, or install the `.deb` with `sudo dpkg -i cursor-*.deb`

**4.** On first launch, Cursor shows a setup screen with the option **Import VS Code Settings**. Click **Import VS Code Settings**. Cursor copies your existing VS Code themes, extensions, and keybindings automatically. If you skip this, you get a clean install — you can still import later via **File → Preferences → Import VS Code Settings**.

**5.** Sign in or create a free Cursor account when prompted. Click **Sign Up** if you do not have one, or **Log In** if you do. A free account gives you a limited number of AI requests per month — enough for this module and most beginner projects.

### Set Claude as your AI model

**6.** Inside Cursor, open **Settings** with `Ctrl+,` (Windows/Linux) or `Cmd+,` (Mac). This opens the settings panel.

**7.** In the settings search bar at the top, type:
```
model
```

**8.** Look for the section labelled **Cursor > AI > Default Model** (or **Models** depending on your version). Click the dropdown and select:
```
claude-3-5-sonnet-20241022
```
or the most recent Claude model shown in the list. Any model labelled "claude-sonnet" or "claude-opus" is Claude.

**9.** Close settings with `Ctrl+W` (Windows/Linux) or `Cmd+W` (Mac). Claude is now the model powering Tab, Chat, and Composer.

### Open your Day 2 project

**10.** In Cursor, go to **File → Open Folder** (Windows/Linux) or **File → Open...** (Mac). Navigate to your `my-first-app` folder from Day 2 — the one containing `index.html`, `styles.css`, and `script.js`. Click **Open**.

You will see your Day 2 files in the Explorer panel on the left side (press `Ctrl+Shift+E` or `Cmd+Shift+E` to toggle it open if it is not visible).

**11.** Click on `index.html` in the Explorer panel to open it. You are now viewing your Day 2 landing page code inside Cursor.

### Experience Tab autocomplete

**12.** Click at the end of any line inside `index.html` — for example, at the end of the line that contains your page title or a button element.

**13.** Press `Enter` to create a new blank line and start typing:
```
<p
```

**14.** Pause for one second. Cursor suggests a completion in grey text — it predicts a full `<p>` element based on the context of your file. Press `Tab` to accept the suggestion. Press `Escape` to dismiss it and type something different.

This is Tab autocomplete. It works in every file type and learns the patterns in your specific project. You do not need to do anything to activate it — it runs in the background whenever you type.

### Use Chat to understand your code

**15.** In `index.html`, highlight the entire `<style>` block (or any section of CSS or JavaScript you want to understand). Click and drag to select it.

**16.** Press `Ctrl+L` (Windows/Linux) or `Cmd+L` (Mac). The Chat panel opens on the right side. Your selected code appears in the chat input automatically.

**17.** Type a question and press Enter:
```
Explain what each part of this CSS block does in plain English. No jargon.
```

Claude explains the selected code line by line. This is the Chat feature — a conversation about your code with full context. You can ask follow-up questions in the same panel without re-selecting code.

**18.** Close the Chat panel when done: press `Ctrl+L` again or click the **X** at the top of the Chat panel.

### Use Composer to add a new feature

Composer is the most powerful Cursor feature for building. It generates a diff — a preview of all changes — before touching any file.

**19.** Open Composer with `Ctrl+Shift+I` (Windows/Linux) or `Cmd+Shift+I` (Mac). A panel appears at the bottom (or right side) of the screen. This is Composer.

**20.** You will see a text input field in the Composer panel. Paste this prompt exactly:

```
Add a "Skills" section to the landing page below the bio. The section should show three skill cards in a row. Each card has: an emoji icon at the top, a skill name in bold, and one short sentence describing it. Use these three skills:
- 🤖 AI Tools | "I build apps by prompting AI — no manual coding."
- 🎨 Design Thinking | "I focus on what users need before I touch any tool."
- 🚀 Fast Shipping | "I go from idea to live product in hours."

Match the existing dark background (#0F172A), white text (#F8FAFC), and mint green (#6EE7B7) accent style already used on the page. Cards should have a subtle border and a soft hover effect. Do not change any existing content or styles — only add the new section.
```

Press Enter or click the **Submit** button.

**21.** Cursor runs Claude and generates a diff. You will see the Composer panel fill with proposed changes — green lines are additions, red lines are deletions. Scroll through the diff to read what Cursor is about to change. Confirm it is only adding a new section, not modifying the existing buttons or bio.

**22.** If the diff looks correct, click **Accept All** at the top of the Composer panel. All changes are applied to your files immediately. If you want to accept changes file by file, click **Accept** next to each file name separately.

**23.** If the diff looks wrong or you want to try a different approach, click **Reject All** — nothing is changed and you can rewrite the prompt.

### Preview the result

**24.** Open a terminal in Cursor with `` Ctrl+` `` (Windows/Linux) or `` Cmd+` `` (Mac).

**25.** Navigate into your project folder and open `index.html` in your browser:
- **Windows:** type `start index.html` and press Enter
- **Mac:** type `open index.html` and press Enter
- **Or:** in the Explorer panel, right-click `index.html` → **Open with Default Browser**

**26.** Verify your Skills section appears below the bio with three cards, matching the existing dark styling. Hover over the cards — the hover effect should activate.

**27.** If the section looks correct, switch back to Cursor, open the terminal, and commit:
```
git add index.html
git commit -m "feat: add skills section with three cards"
```

<!-- ASSET: diagram | Four-panel sequence: (1) Cursor with Composer panel open and the prompt typed in, (2) diff preview showing green additions for the skills section, (3) Accept All button highlighted, (4) browser showing the updated page with three skill cards -->

## Practical workflow

1. Open Cursor → `Ctrl+Shift+E` to open Explorer
2. **File → Open Folder** → select your project folder
3. `Ctrl+,` → search "model" → set model to Claude (claude-sonnet or claude-opus)
4. Open a file → start typing → press `Tab` to accept autocomplete suggestions
5. Select code → `Ctrl+L` → ask a question in Chat panel → close Chat with `Ctrl+L`
6. `Ctrl+Shift+I` → type feature request in Composer → press Enter
7. Review diff → green = added, red = removed → scroll through all files
8. **Accept All** if correct → **Reject All** if not
9. Open browser → verify the change works
10. In terminal: `git add [filename]` → `git commit -m "feat: [description]"`

## Common mistakes

**Mistake 1: Accepting a Composer diff without reading it.**
Composer shows you the full diff before saving anything — that preview is the entire point. If you click Accept All without reading it, you may accept large rewrites of sections you did not intend to change. Fix: before clicking Accept All, scroll through the entire diff. Check that only the expected files are touched. If Composer is changing five files when you only expected it to touch one, type "Reject All" and rewrite the prompt with a narrower scope: "Only modify `index.html` — do not touch any other files."

**Mistake 2: Using Composer when Claude Code is the right tool.**
Composer is powerful for in-editor, focused changes. But if you are building a whole new feature from scratch — creating new files, a new route, a new component — Claude Code in the terminal handles multi-file creation and project setup better. Fix: use Composer for changes to existing files. Use Claude Code (`claude` in terminal) for anything that creates new files or restructures the project. The decision card at the end of this module gives you the exact rule.

**Mistake 3: Ignoring Tab autocomplete suggestions because they look wrong.**
Autocomplete suggestions are predictions, not commands. If a suggestion is wrong, press `Escape` — the suggestion disappears and you keep typing. Beginners sometimes press `Backspace` on the grey suggestion text, which deletes the code they just typed, not the suggestion. Fix: `Escape` dismisses suggestions. `Tab` accepts them. `Backspace` edits your real code. Never press Backspace to reject a suggestion.

## Your turn

Open your Day 2 `index.html` file in Cursor. Use Composer (`Ctrl+Shift+I` or `Cmd+Shift+I`) to add a "Contact" section at the bottom of the page with a single email link styled to match the existing design.

Use this prompt in Composer:

```
Add a simple "Get in Touch" section at the bottom of the page above the closing </body> tag. It should show a short line of text: "Want to build something together?" and below it a styled email link: hello@yourname.com — link it to mailto:hello@yourname.com. Match the existing dark background and mint green accent colour. Do not change any other part of the page.
```

Expected output: a diff showing a new section added to `index.html` only, with no changes to existing sections. Accept the diff and verify the section appears correctly in your browser.

Failure state: Composer changes sections you did not expect. Fix: click **Reject All**, then add to your prompt: "Only insert HTML inside the `<body>` tag of `index.html` — do not modify any existing styles, scripts, or other sections."

## Prompt / Template / Checklist pack

### Cursor Setup Checklist

Run through this once when you first install Cursor.

**Installation:**
- [ ] Downloaded Cursor from cursor.com
- [ ] Installer run and Cursor launched
- [ ] VS Code settings imported (or clean install accepted)
- [ ] Free account created and signed in

**Model configuration:**
- [ ] `Ctrl+,` → search "model" → Claude model selected (claude-sonnet-20241022 or latest)
- [ ] Test: open any file, press `Ctrl+L`, ask "say hello" — Claude responds in Chat panel

**First use verification:**
- [ ] Project folder opened via File → Open Folder
- [ ] Tab autocomplete tested: typed in a file, saw grey suggestion, pressed Tab to accept
- [ ] Chat tested: selected code, pressed `Ctrl+L`, asked a question, got an answer
- [ ] Composer tested: pressed `Ctrl+Shift+I`, typed a small change, reviewed diff, accepted

---

### Composer Prompt Templates

Copy the one that matches your task. Fill in the brackets.

---

**Template 1 — Add a new section to an existing page:**
```
Add a "[SECTION NAME]" section to [FILE NAME] below the [EXISTING SECTION NAME].

Content:
- [ITEM 1]
- [ITEM 2]
- [ITEM 3]

Style requirements:
- Match the existing [DESCRIBE EXISTING STYLE — e.g. dark background, mint accent]
- [Any specific styling notes]

Do not change any existing content, styles, or scripts — only add the new section.
```

---

**Template 2 — Modify one specific function or element:**
```
In [FILE NAME], change [SPECIFIC ELEMENT OR FUNCTION NAME] to do the following:

Current behaviour:
[What it does now]

New behaviour:
[What it should do instead]

Do not change anything else in the file.
```

---

**Template 3 — Fix a visual problem with a specific element:**
```
In [FILE NAME], the [ELEMENT — e.g. navigation bar, footer, card] looks wrong:

Problem:
[Describe the visual issue — e.g. "the cards are not aligned in a row", "the button has no padding"]

Expected appearance:
[Describe what it should look like]

Fix only the styles for that element. Do not change any other CSS or HTML.
```

---

**Template 4 — Refactor a section for readability:**
```
In [FILE NAME], refactor the [SECTION — e.g. hero section, JavaScript event listeners] to be cleaner and easier to read.

Rules:
- Do not change any visible behaviour or appearance
- Do not rename IDs or class names used elsewhere
- Add a short comment above each logical group of code
- Keep all changes inside this file only
```

---

**Template 5 — Add interactivity to an existing element:**
```
In [FILE NAME], add the following interactive behaviour to the [ELEMENT NAME — e.g. "Skills cards", "navigation links"]:

Behaviour:
[Describe the interaction — e.g. "cards should expand to show more detail when clicked", "links should scroll smoothly to their section"]

Implementation notes:
- Use vanilla JavaScript — no libraries
- Add the JavaScript at the bottom of the file inside a <script> tag
- Do not modify any existing JavaScript

Style:
- [Any visual changes to support the interaction — e.g. "add a cursor: pointer to cards", "add a transition for the expand animation"]
```

---

### Cursor vs Claude Code Decision Card

Use this card every time you are about to start a task. Pick the column that matches.

| Task | Use This Tool |
|---|---|
| Build a new feature from scratch | Claude Code (terminal) |
| Create new files or folders | Claude Code (terminal) |
| Add a section to an existing page | Cursor Composer |
| Fix a specific function | Cursor Composer |
| Understand what a block of code does | Cursor Chat |
| Ask why a variable is undefined | Cursor Chat |
| Set up or edit CLAUDE.md | Claude Code (terminal) |
| Install or configure plugins (MCP) | Claude Code (terminal) |
| Commit and push to git | Claude Code (terminal) |
| Refactor one file | Cursor Composer |
| Fix a layout or style issue in one file | Cursor Composer |
| Run the dev server | Claude Code (terminal) |
| Rename a variable across all files | Cursor Composer |
| Generate boilerplate for a whole new project | Claude Code (terminal) |
| Accept or reject a diff before it saves | Cursor Composer |

**Quick rule:** If the task creates new files or touches the whole project → Claude Code. If the task modifies existing files and you want to review a diff first → Cursor Composer. If you want to ask a question about code → Cursor Chat.

---

### Chat Prompt Starters

Copy these into Cursor Chat (`Ctrl+L`) with code selected.

**Understand code:**
```
Explain what this code does in plain English. Assume I have no coding background.
```

**Find a bug:**
```
Look at this code. Is there anything that could cause an error or unexpected behaviour? If yes, explain what and why.
```

**Improve readability:**
```
How could this code be made easier to read without changing what it does? Do not rewrite it — just explain what you would change and why.
```

**Check for accessibility issues:**
```
Does this HTML have any basic accessibility problems — missing alt text, unlabelled buttons, poor contrast? List any issues you find.
```

**Ask about a specific line:**
```
What does line [X] do, and why is it written this way?
```
