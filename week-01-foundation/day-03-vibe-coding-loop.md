# Module 2: The Vibe Coding Loop — Prompt, Review, Iterate

## Outcome artifact
By the end of this module you will have the app from Module 1 with 3 new features added, each built using a single clear prompt and reviewed before accepting — plus a completed Feature Log documenting what you added and how.

<!-- ASSET: diagram | Circular loop diagram with three nodes: "Write prompt" → "Read Claude's output" → "Accept or revise" → back to "Write prompt" -->

## The core idea

Building an app is not one big prompt. It is a loop you run once per feature.

The loop has three steps: write a prompt for one feature, read what Claude did, then decide to accept it or ask for a revision. That's it. Every feature you will ever add to any app follows this loop. The apps get more complex, but the loop stays the same.

The single most important rule in this loop is: **one feature per prompt.** Not two features. Not "add a dark mode, fix the button colors, and also add a search bar." One. The reason is practical: when you give Claude three things to do, it does all three at once. If one of the three breaks something, you can't tell which instruction caused the problem. You have to undo everything and start over. When you give Claude one thing to do, a problem is always traceable to that one change.

The second important concept is the difference between **accepting** and **reverting**. When Claude makes a change, nothing is permanent until you accept it. If Claude's output isn't right, you can ask for a revision without losing what was already working. This is your safety net. Use it freely.

The third concept is the **diff**. A diff is a summary of what changed — which lines were added, which were removed. Claude shows you a diff before applying changes. You don't need to understand every line of code in it. You need to read it enough to answer: does this look like what I asked for? Added lines mean new code. Removed lines mean deleted code. If the diff adds fifty lines and you asked for one small feature, that's a signal to look more carefully before accepting.

Once a feature is working and accepted, you commit it to git. A commit is a save point — it captures the state of your project at that moment so you can always return to it. Two commands: `git add .` to stage all changes, then `git commit -m "description"` to save them. Commit after every working feature. Never commit a broken state.

## Step-by-step walkthrough

You will add three features to the landing page you built in Module 1. The features are:
1. A dark/light mode toggle button
2. A footer with a copyright line
3. A "Back to top" button that appears when the user scrolls down

Work through them in order. Each one uses the same loop.

### Open your project

**1.** Open VS Code. Press `` Ctrl+` `` to open the terminal.

**2.** Navigate into your project folder:
```
cd my-first-app
```

**3.** Start Claude Code:
```
claude
```

You'll see the `>` prompt. Claude is ready.

### Initialise git (first time only)

Before you can commit, you need git tracking set up for this project.

**4.** In the terminal (while Claude Code is running), you can either type `git init` outside Claude, or ask Claude to do it:
```
Initialise a git repository in this folder and create a first commit that captures all existing files.
```

Claude will run `git init`, stage all files, and create an initial commit. You'll see output like:
```
Initialized empty Git repository in /path/to/my-first-app/.git/
[main (root-commit) abc1234] Initial commit
 3 files changed, 87 insertions(+)
```

You now have a save point. Any future commits stack on top of this one.

### Feature 1: Dark/light mode toggle

**5.** Type this prompt and press Enter:
```
Add a dark/light mode toggle button to the landing page. The button should appear in the top-right corner of the page. Clicking it switches the page between the current dark theme (#0F172A background, #F8FAFC text) and a light theme (#F8FAFC background, #0F172A text). The mint green accent (#6EE7B7) stays the same in both modes. Use JavaScript to toggle a class on the body element. Save the user's preference to localStorage so it persists after a page refresh.
```

**6.** Claude will show you its plan and then display a diff. The diff shows lines with `+` (added) and `-` (removed). You don't need to read every line. Check: does the diff mention a button, a toggle function, localStorage, and CSS changes for light/dark classes? If yes, those are the right things.

**7.** If Claude asks `Allow changes? (y/n)`, type `y` and press Enter.

**8.** Open `index.html` in your browser (or refresh if it's already open). Test the toggle: click the button, confirm the theme changes, refresh the page, confirm the preference was saved.

If the toggle works: commit it.

**9.** Ask Claude to commit:
```
Commit these changes with message "feat: add dark/light mode toggle with localStorage persistence"
```

Claude runs `git add .` and `git commit`. You'll see:
```
[main a1b2c3] feat: add dark/light mode toggle with localStorage persistence
 2 files changed, 34 insertions(+), 2 deletions(-)
```

Feature 1 is done and saved. If anything goes wrong later, you can always return to this exact state.

### Feature 2: Footer

**10.** Type this prompt:
```
Add a footer at the bottom of the page. The footer should contain one line of text: "© 2025 Alex. Built with Claude Code." Centre the text. Use the same dark background (#0F172A) and white text (#F8FAFC) as the rest of the page. Add a thin top border using the mint green accent colour (#6EE7B7) with 20% opacity. Padding: 24px top and bottom.
```

**11.** Review the diff. Expect to see a `<footer>` element added to the HTML and corresponding CSS styles.

**12.** Accept the changes, refresh your browser, confirm the footer appears.

**13.** Ask Claude to commit:
```
Commit with message "feat: add footer with copyright line"
```

### Feature 3: Back to top button

**14.** Type this prompt:
```
Add a "Back to top" button that appears in the bottom-right corner of the page when the user has scrolled down more than 300px. Clicking it smoothly scrolls the page back to the top. The button should be a small circular button using the mint green accent (#6EE7B7) with a dark arrow icon (↑). It should fade in when it appears and fade out when it hides. Use JavaScript scroll event listener to show/hide it.
```

**15.** Review the diff. Expect: a button element in the HTML, CSS for fixed positioning, opacity transitions, and a JavaScript scroll event listener.

**16.** Accept, test in browser (scroll down — button should appear; click it — page should scroll to top), then commit:
```
Commit with message "feat: add back to top button with scroll detection"
```

### Complete your Feature Log

**17.** Open a new file in VS Code. Press `Ctrl+N` to create a new untitled file. Save it as `feature-log.md` in your project folder (`Ctrl+S`, then type the filename).

**18.** Fill in the Feature Log template (at the end of this module). Copy it, paste it into `feature-log.md`, and fill in what you did.

<!-- ASSET: diagram | Side-by-side: left shows a diff with + and - lines, right shows a browser with the updated app — arrow connecting diff review to the live result -->

## Practical workflow

1. Open VS Code → press `` Ctrl+` `` → `cd your-project-folder` → run `claude`
2. Decide what one feature to add
3. Write a single-feature prompt → include: what it does, where it appears, colours/style, any behaviour (e.g. animation, persistence)
4. Press Enter → read Claude's output
5. Review the diff — check: does it match what you asked for?
6. If yes: type `y` to accept → test in browser
7. If no: type `n` to reject → write a revised prompt with more specific instructions
8. Once feature works in browser: ask Claude to commit → `Commit with message "feat: [description]"`
9. Repeat from step 2 for next feature

## Common mistakes

**Mistake 1: Writing a multi-feature prompt.**
"Add dark mode, a footer, a contact form, and fix the button spacing" — Claude attempts all four at once. When something breaks you don't know which instruction caused it. Fix: one feature per prompt, every time. If you have a list of features you want, save the list somewhere and add them one at a time.

**Mistake 2: Accepting changes without testing in the browser.**
The diff looks right, you accept it, you move on. But the feature is broken because Claude wrote valid code that does the wrong thing. Fix: always open the browser and test the feature manually before committing. Commit is the moment you say "this works." Never commit untested.

**Mistake 3: Asking Claude to undo changes inside Claude Code.**
You accepted a change, it broke something, and now you want to undo it. Typing "undo that" in Claude Code doesn't reliably reverse file changes — it asks Claude to write more code to reverse the last change, which can cascade. Fix: use git. Ask Claude to run `git checkout -- .` to discard all uncommitted changes and return to the last commit. This is why you commit after every working feature.

## Your turn

Add the three features to your landing page:
1. Dark/light mode toggle
2. Footer
3. Back to top button

After all three are added and committed, run `git log --oneline` in the terminal. You should see at least 4 commits: the initial commit, and one for each feature.

Expected output:
```
a1b2c3 feat: add back to top button with scroll detection
b2c3d4 feat: add footer with copyright line
c3d4e5 feat: add dark/light mode toggle with localStorage persistence
d4e5f6 Initial commit
```

If you see this: you've completed the vibe coding loop three times, and your project has a full commit history. That's a professional workflow. Complete the Feature Log template and you're done.

## Prompt / Template / Checklist pack

### Vibe Coding Loop Workflow Card

Reference this during every build session. Print it, pin it, or keep this page open.

```
THE VIBE CODING LOOP

Step 1 — One feature only
  Define the ONE thing you are adding in this loop.
  If you have a list: pick the first item. Save the rest for later.

Step 2 — Write the prompt
  Include:
  - What it does (the behaviour)
  - Where it appears (position, layout)
  - How it looks (colours, size, animation)
  - Any data or state it needs (localStorage? API? Calculated?)

Step 3 — Read the diff
  + lines = added code
  - lines = removed code
  Ask yourself: does this match what I asked for?
  If unsure: type n and ask Claude to explain what it changed.

Step 4 — Accept or revise
  Accept (y) → test in browser
  Revise (n) → write a new prompt with more specific instructions
  Never accept code you haven't tested.

Step 5 — Test in browser
  Open the app. Test the feature.
  Does it do what you asked?
  Does anything else look broken?

Step 6 — Commit if working
  Ask Claude: "Commit with message 'feat: [what you added]'"
  Never commit a broken state.
  Never skip the commit.

WHEN TO COMMIT: After every working feature. Before you add the next thing.
WHEN TO REVERT: git checkout -- . discards all uncommitted changes.
```

---

### Common Revision Prompts

Use these when Claude's first attempt isn't right. Copy-paste, fill in the blank.

**When the style is wrong:**
```
The feature works but the style doesn't match the rest of the page. Update the [element] to use [exact specification — e.g., background #0F172A, border-radius 8px, font-size 14px]. Do not change any other styles.
```

**When the feature is in the wrong place:**
```
Move the [element] to [exact location — e.g., "top-right corner of the page, 20px from each edge, fixed position"]. Do not change anything else.
```

**When the feature partially works:**
```
The [feature] does [what it does correctly] but [what it doesn't do]. Fix only the [specific broken behaviour] — do not change anything else.
```

**When Claude changed something you didn't ask it to:**
```
Revert only the changes to [file/element] — I did not ask for those to change. Keep the [feature I did ask for].
```

**When you need Claude to explain before you accept:**
```
Before making any changes: explain what you plan to do in plain English. What files will you change, and why?
```

---

### Feature Log Template

Save this as `feature-log.md` in your project root. Fill in each row after adding a feature.

```markdown
# Feature Log — [Project Name]

| # | Feature | Prompt summary | What Claude did | Did it work first try? | Notes |
|---|---------|---------------|-----------------|----------------------|-------|
| 1 | Dark/light mode toggle | Asked for toggle button, CSS classes, localStorage persistence | Added toggle button in HTML, JavaScript class switcher, localStorage save/load | Yes / No | |
| 2 | Footer | Asked for footer with copyright, mint border, centred text | Added <footer> element with CSS styles | Yes / No | |
| 3 | Back to top button | Asked for fixed-position button, scroll detection, fade animation | Added button HTML, scroll event listener, CSS transitions | Yes / No | |

## Revision log (fill in when Claude needs more than one try)

| Feature | What went wrong | Revision prompt I used | What fixed it |
|---------|----------------|----------------------|---------------|
| | | | |
```
