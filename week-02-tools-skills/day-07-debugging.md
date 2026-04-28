# Module 5: When It Breaks — Debugging with Claude

## Outcome artifact
By the end of this module you will have a completed Fix Log entry documenting one real bug encountered in your app — the error message, what you asked Claude, what it tried, and what fixed it — using the provided template.

<!-- ASSET: diagram | Three-panel sequence: left shows a browser error (red console text), center shows the debugging prompt pasted into Claude, right shows the browser working correctly — arrows connecting each step -->

## The core idea

Every app breaks. Not sometimes — always, at some point. Buttons stop responding. Styles disappear. The page goes blank. You see red text in the browser's developer console that means nothing to you. This is not a sign that something is fundamentally wrong with your project or that vibe coding doesn't work. It is a normal part of building anything.

The beginner mistake is to panic, paste the error into a new Claude session with no context, and accept whatever Claude suggests. Claude gives you a fix. You apply it. Something else breaks. You paste that error in. Repeat. This loop can last hours and leave you worse off than when you started.

The professional approach has three steps: **copy the error, add context, ask for one fix at a time.**

**Copying the error** means getting the exact error message — the full text, not a paraphrase. "It doesn't work" gives Claude almost nothing to work with. `Uncaught TypeError: Cannot read properties of null (reading 'addEventListener')` at line 47 of `script.js` tells Claude exactly what broke, where, and why. That single string of text is worth ten minutes of explanation.

**Adding context** means telling Claude two things: what you were doing when the error appeared, and what you expected to happen. "I added a new button and now the back to top button has stopped working. The error appears in the console when I load the page." That sentence gives Claude the sequence of events — which is often the actual cause.

**Asking for one fix at a time** means resisting the urge to list every problem you can see. If three things broke, pick the most important one and fix it first. Each fix may resolve the others. If you ask Claude to fix three things simultaneously, it changes three parts of the code at once, and if the combined result is wrong, you cannot trace which change caused the problem.

There is one more tool you need to know: `git checkout -- .`

This command discards all uncommitted changes and returns your project to its last committed state. It is the nuclear undo. Use it when Claude's fix makes things worse and you want to start fresh from the last working version. This is why committing after every working feature (Module 2) matters — your last commit is your safety net.

## Step-by-step walkthrough

### Find the error

When something breaks in your browser, your first move is to open the browser's developer console. This is where JavaScript errors appear with exact file names and line numbers.

**1.** In your browser, right-click anywhere on your page and select **Inspect** (Chrome, Edge, Firefox). Or press `F12`.

**2.** In the panel that opens, click the **Console** tab at the top.

**3.** You will see messages. Look for red text — those are errors. A typical error looks like:
```
Uncaught TypeError: Cannot read properties of null (reading 'addEventListener')
    at script.js:47:23
```

This tells you: the type of error (TypeError), what caused it (trying to read `addEventListener` from something that is null), and exactly where it is (script.js, line 47, character 23).

**4.** Click on the `script.js:47` part of the error — it opens that exact line in the Sources tab so you can see the code in question.

### Copy the error correctly

**5.** Click on the error message in the console. In most browsers you can select all the text by clicking and dragging. Copy it with `Ctrl+C` (Windows/Linux) or `Cmd+C` (Mac).

Copy the full error including the file name and line number — not just the first line. If the error has a "stack trace" (multiple lines of indented text below the main error), copy those too.

### Write the debugging prompt

**6.** Open Claude Code in your project folder:
```
claude
```

**7.** Use the 3-part debugging prompt structure:

```
Error:
[paste the full error message here]

Context:
[What were you doing when this happened? What did you last change?]

Expected behaviour:
[What should the app do instead?]

Please explain what is causing this error and suggest a fix. Change only what is needed to fix this specific error — do not refactor or change anything else.
```

**Example of a filled-in debugging prompt:**
```
Error:
Uncaught TypeError: Cannot read properties of null (reading 'addEventListener')
    at script.js:47:23

Context:
This error appeared after I added a new "Share" button to the page. 
The back-to-top button was working before I made this change. 
Now the page loads but the back-to-top button doesn't respond to scrolling.

Expected behaviour:
The back-to-top button should appear when the user scrolls down 300px 
and disappear when they scroll back up, as it did before.

Please explain what is causing this error and suggest a fix. 
Change only what is needed to fix this specific error — do not refactor or change anything else.
```

**8.** Claude will diagnose the error and propose a fix. It usually explains the cause in plain English first — read this explanation. It often reveals something about how your code works that will help you avoid the same error in the future.

**9.** Review the diff Claude proposes. Does it touch only the part of the code related to the error? If Claude is rewriting large sections of code to fix a one-line bug, that is a warning sign. Ask: "Can you make a smaller change that only addresses this specific error?"

**10.** Accept the fix if it looks targeted and correct. Test in browser: does the error disappear? Does the feature work? Does anything else look different from before?

### When the fix makes things worse

**11.** If Claude's fix introduces a new problem, do not stack another fix on top of it. Instead, revert to your last commit and start fresh:

Ask Claude to run:
```
Please run: git checkout -- .
```

Or exit Claude and run it yourself in the terminal:
```
git checkout -- .
```

This returns all files to the state of your last commit — the last state you confirmed was working. Your session history in Claude is gone, but your code is safe.

**12.** Start a new Claude session and try the debugging prompt again with more specifics about what didn't work with the previous fix.

### Document the fix

**13.** Once the bug is fixed and the app is working again, commit immediately:
```
Commit with message "fix: [describe what you fixed]"
```

**14.** Fill in a Fix Log entry using the template at the end of this module. This takes two minutes and is worth doing — your Fix Log becomes a personal reference. The next time you see a similar error, you have your own documented solution.

<!-- ASSET: diagram | Fix Log entry filled in: three columns showing Error / What I asked Claude / What worked — a concrete example with real error text -->

## Practical workflow

1. Something breaks → open browser → press `F12` → click **Console** tab
2. Find red error text → copy the full error including file name and line number
3. Start `claude` in project folder
4. Paste error using 3-part prompt: Error / Context / Expected behaviour
5. Read Claude's explanation → review the diff
6. Accept if targeted and correct → test in browser
7. If fix creates new problems: `git checkout -- .` → start fresh with more specific prompt
8. Once fixed: commit → `git commit -m "fix: [description]"`
9. Fill in Fix Log entry (2 minutes)

## Common mistakes

**Mistake 1: Pasting only the first line of the error.**
"Uncaught TypeError: Cannot read properties of null" without the file name and line number is half an error message. Claude has to guess where the problem is. Fix: always copy the full error including the `at filename.js:line:column` part. That line number is often what lets Claude find the exact cause in ten seconds instead of ten minutes.

**Mistake 2: Asking Claude to fix multiple bugs at once.**
"Fix the scroll bug, also the button colour is wrong, and the footer is overlapping the button" — Claude changes three things simultaneously. If the result is wrong, you don't know which change caused the new problem. Fix: one bug per debugging session. Fix the most important one first. Re-test. Then address the next one.

**Mistake 3: Continuing to stack fixes on top of a broken state.**
Claude's fix didn't work. You ask Claude to fix the fix. That fix also doesn't work. Now you have three layers of changes on top of code that was working fine. Fix: the moment a fix makes things worse, run `git checkout -- .` and go back to the last working commit. Start from clean ground.

## Your turn

Introduce a deliberate bug in your project so you can practice the debugging loop.

**1.** Open `script.js` in VS Code.

**2.** Find a line where an element is selected by its ID — it will look something like:
```javascript
const button = document.getElementById('back-to-top');
```

**3.** Change the ID to a non-existent one — misspell it:
```javascript
const button = document.getElementById('back-to-topp');
```

**4.** Save the file and open `index.html` in your browser.

**5.** Press `F12` → Console tab → you should see a TypeError about the back-to-top button.

**6.** Run the full debugging loop: copy the error, write the 3-part debugging prompt, let Claude diagnose and fix it.

After Claude fixes it, fix the Fix Log template with this bug entry. You have now run the debugging loop once in a low-stakes environment. The next real bug will feel familiar.

## Prompt / Template / Checklist pack

### 5 Bug Fix Prompt Templates

Copy the one that matches your situation. Fill in the brackets.

---

**Template 1 — JavaScript error in console:**
```
Error:
[paste full error from browser console, including file name and line number]

Context:
I was [describe what you just changed or added] when this appeared.
Before this change, [describe what was working].

Expected behaviour:
[What the app should do instead of throwing this error.]

Please diagnose the cause and suggest the minimal change to fix it. 
Do not refactor or change anything beyond what is needed.
```

---

**Template 2 — Feature stopped working (no visible error):**
```
The [feature name] has stopped working. There are no errors in the browser console.

What it used to do:
[Describe the expected behaviour in detail.]

What it does now:
[Describe what actually happens — or doesn't happen.]

What I last changed:
[Describe the last thing you added or modified before it stopped working.]

Please look at [feature name]-related code and find why it stopped working. 
Propose the smallest possible fix.
```

---

**Template 3 — Broken layout or style:**
```
The layout is broken after my last change.

Problem:
[Describe what looks wrong — e.g., "the footer overlaps the back-to-top button", 
"the buttons are no longer centred", "the dark mode toggle is off-screen"]

What I last changed:
[Describe the CSS or HTML you last modified.]

Expected appearance:
[Describe what it should look like — or reference the original design.]

Please fix only the layout issue without changing any other styles.
```

---

**Template 4 — "It works on my computer but looked wrong yesterday":**
```
My app was working correctly, but today it [describe the problem].
I have not changed the code since it was working.

Browser: [e.g. Chrome, Firefox, Safari]
Operating system: [Windows / Mac / Linux]

Steps to reproduce:
1. Open index.html
2. [Describe the exact steps that produce the wrong behaviour]

Expected: [What should happen]
Actual: [What happens]

Console errors (if any): [paste any red text from F12 → Console]

Please help me diagnose what changed.
```

---

**Template 5 — Claude's previous fix made things worse:**
```
The fix you applied for [original error] introduced a new problem.

New problem:
[Describe what is now broken that wasn't broken before.]

New console error (if any):
[paste error]

Please:
1. Run git checkout -- . to discard the current changes
2. Look at the original error again and propose a different approach

I want to go back to the last working commit and try again.
```

---

### Fix Log Template

Save this as `fix-log.md` in your project folder. Add a row every time you fix a bug.

```markdown
# Fix Log — [Project Name]

## How to use this log
After fixing every bug: add a row below.
Date | Error | Cause | Fix | Prompt that worked

---

| Date | Error | Cause (what Claude said) | Fix applied | Did first fix work? |
|------|-------|--------------------------|-------------|---------------------|
| [date] | [paste error summary] | [what Claude diagnosed] | [what change was made] | Yes / No — needed [N] attempts |

---

## Debugging prompts that worked well

[Paste prompts that produced a clear diagnosis on the first try. These are your proven templates.]

---

## Patterns I keep seeing

[After 5+ bugs, you will start to notice patterns. Write them here.
Example: "I keep forgetting to update the element ID in both HTML and JS when I rename something."]
```
