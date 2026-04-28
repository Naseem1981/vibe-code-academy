# Vibe Coding Loop Workflow Card
*Module 2 Deliverable — Vibe Code from Zero*

---

## THE VIBE CODING LOOP

```
Step 1 — One feature only
  Define the ONE thing you are adding in this loop.
  If you have a list: pick the first item. Save the rest for later.

Step 2 — Write the prompt
  Include:
  - What it does (the behaviour)
  - Where it appears (position, layout)
  - How it looks (colours, size, animation)
  - Any data or state it needs (localStorage? Calculated? API?)

Step 3 — Read the diff
  + lines = added code
  - lines = removed code
  Ask: does this match what I asked for?
  If unsure: type n and ask Claude to explain what it changed.

Step 4 — Accept or revise
  Accept (y) → test in browser
  Revise (n) → write a new prompt with more specific instructions

Step 5 — Test in browser
  Open the app. Test the feature manually.
  Does it do what you asked?
  Does anything else look broken?

Step 6 — Commit
  Ask Claude: "Commit with message 'feat: [what you added]'"
  Never commit untested code.
  Never skip the commit.
```

**COMMIT RULE:** After every working feature. Before you add the next thing.

**REVERT COMMAND:** `git checkout -- .` discards all uncommitted changes.

---

## Common Revision Prompts

**When style is wrong:**
```
The feature works but the style doesn't match the rest of the page. Update [element] 
to use [exact specification]. Do not change any other styles.
```

**When position is wrong:**
```
Move [element] to [exact location — e.g., "top-right corner, 20px from each edge, fixed position"]. 
Do not change anything else.
```

**When feature partially works:**
```
The [feature] does [what works] but [what doesn't work]. Fix only the [broken behaviour]. 
Do not change anything else.
```

**When Claude changed something extra:**
```
Revert only the changes to [file/element] — I did not ask for those. 
Keep the [feature I did ask for].
```

**Before accepting a large change:**
```
Before making any changes: explain in plain English what you plan to do. 
What files will you change, and why?
```

---

## Feature Log Template

Save as `feature-log.md` in your project root.

```markdown
# Feature Log — [Project Name]

| # | Feature | Prompt summary | What Claude did | First try? | Notes |
|---|---------|---------------|-----------------|------------|-------|
| 1 | | | | Yes / No | |
| 2 | | | | Yes / No | |
| 3 | | | | Yes / No | |

## Revisions needed
| Feature | What went wrong | Revision prompt | What fixed it |
|---------|----------------|-----------------|---------------|
| | | | |
```

---

## Git Quick Reference

| Action | Command |
|--------|---------|
| Initialise git | `git init` |
| Stage all files | `git add .` |
| Commit | `git commit -m "feat: description"` |
| View commit history | `git log --oneline` |
| Revert uncommitted changes | `git checkout -- .` |

**Commit message format:**
- `feat: add [feature name]` — new feature
- `fix: resolve [bug description]` — bug fix
- `docs: update CLAUDE.md` — documentation
- `style: adjust [element] colours` — style only
