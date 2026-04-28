# Bug Fix Prompt Templates + Fix Log
*Module 5 Deliverable — Vibe Code from Zero*

---

## 5 Bug Fix Prompt Templates

Copy the one that matches your situation. Fill in the brackets. The more specific you are, the fewer attempts Claude needs.

---

### Template 1 — JavaScript error in browser console

```
Error:
[paste full error from browser console — include file name and line number]

Context:
I was [describe what you just changed or added] when this appeared.
Before this change, [describe what was working correctly].

Expected behaviour:
[What the app should do instead of throwing this error.]

Please diagnose the cause and suggest the minimal change to fix it.
Do not refactor or change anything beyond what is needed to resolve this specific error.
```

---

### Template 2 — Feature stopped working (no visible error)

```
The [feature name] has stopped working. There are no errors in the browser console.

What it used to do:
[Describe the expected behaviour in detail — what the user sees and what happens.]

What it does now:
[Describe what actually happens — or doesn't happen — when you try to use it.]

What I last changed:
[Describe the last thing you added or modified before it stopped working.]

Please look at the [feature name]-related code and find why it stopped working.
Propose the smallest possible fix that restores the original behaviour.
```

---

### Template 3 — Broken layout or visual style

```
The layout is broken after my last change.

Problem:
[Describe what looks wrong — e.g., "the footer overlaps the back-to-top button",
"the buttons are no longer centred", "the dark mode toggle is off-screen on mobile"]

What I last changed:
[Describe the CSS or HTML you last modified.]

Expected appearance:
[Describe what it should look like — or reference the style from before the change.]

Please fix only the layout issue. Do not change any other styles or functionality.
```

---

### Template 4 — "It worked before but now looks wrong"

```
My app was working correctly, but today it [describe the problem].
I have not changed the code since it was last working.

Browser: [Chrome / Firefox / Safari / Edge]
Device: [desktop / mobile / both]

Steps to reproduce:
1. Open index.html in a browser
2. [Describe the steps that produce the wrong result]

Expected: [What should happen]
Actual: [What happens instead]

Console errors (F12 → Console tab): [paste any red text, or write "none"]

Please help me diagnose what is causing this.
```

---

### Template 5 — Previous fix made things worse

```
The fix you applied for [original error description] introduced a new problem.

New problem:
[Describe what is now broken that was not broken before the fix.]

New console error (if any):
[paste error, or write "no new error — the feature just doesn't work correctly"]

Please:
1. Run: git checkout -- .
   (This discards the current uncommitted changes and returns to the last commit.)
2. Look at the original error again with fresh context
3. Propose a different approach — something more targeted that avoids the side effects

I want to start from the last working commit and try a different fix.
```

---

## Fix Log Template

Save this as `fix-log.md` in your project root. Fill in a row after every bug you fix.

```markdown
# Fix Log — [Project Name]

## Bug history

| Date | Error/Problem | Cause (what Claude said) | Fix applied | Attempts needed |
|------|--------------|--------------------------|-------------|-----------------|
| [date] | [brief description] | [one sentence] | [one sentence] | 1 / 2 / 3 |

---

## Debugging prompts that worked well

[After a debugging session that worked quickly on the first try, paste the prompt here.
These are your proven templates — use them as starting points for similar future bugs.]

Prompt that worked for [error type]:
```
[paste the prompt that produced a clean diagnosis]
```

---

## Patterns I keep seeing

[After 5+ bugs, patterns emerge. Write them here.
Example: "I keep forgetting to update the element ID in both HTML and JS when I rename things."
Example: "Whenever I add a new button, I need to check that the scroll listener still references the right elements."]

- [Pattern 1]
- [Pattern 2]
```

---

## Debugging Quick Reference

### Find the error
1. Browser → right-click → Inspect (or press F12)
2. Click Console tab
3. Look for red text
4. Click the filename:linenumber link in the error to see the code

### Copy the error correctly
- Select all the red text — including the `at filename.js:line:col` part
- That line number is what lets Claude find the exact cause

### Revert to last commit
If a fix makes things worse and you want to start over:
```
git checkout -- .
```
This discards ALL uncommitted changes. Cannot be undone. Only run after a bad fix that you want to completely undo.

### Key rule
One bug per debugging session. Never ask Claude to fix multiple bugs at once.
