# Day 13: Budget Tracker — Part 2

## Outcome artifact

By the end of this day you will have a completed Budget Tracker with categories, category filtering, summary stat cards, delete, inline edit, and CSV export — deployed to a live GitHub Pages URL you can share with anyone.

---

## The core idea

When a project grows beyond three or four features, a new challenge appears: Claude Code does not have unlimited context about your codebase. If you paste a 400-line file and ask for a large change, the model must hold the entire file in mind while also processing the new requirement. The more complex the existing code, the higher the chance it introduces a subtle regression — a small change in one function that quietly breaks another.

The solution is to keep your prompts narrow and your feature scope small. Each prompt you send today adds exactly one feature. Before sending the next prompt, you test the feature you just added. If it works, you commit. That commit becomes your safe point. If the next feature breaks something, you can always revert to the last working commit. This is the vibe coding loop at scale, and it is what separates projects that ship from projects that get rewritten from scratch every weekend.

There is also a specific technique for keeping Claude focused on complex files: describe what you want added without asking it to rewrite things that already work. Phrases like "do not change the existing add-transaction logic" and "only add to this section" constrain the model's reach. You are the architect. Claude is the builder. You tell it exactly where to build and what to leave alone.

---

## Step-by-step walkthrough

**Step 1: Open your Day 12 project in Claude Code**

Navigate to your budget-tracker folder and open Claude Code.

```
claude
```

Confirm the Day 12 features all still work before adding anything new.

**Step 2: Add categories**

```
Add a category field to the Budget Tracker transaction data model. Requirements:

Categories: Food, Transport, Entertainment, Income, Other

Changes needed:
1. Add a category dropdown to the transaction form, between the Amount field and the Type toggle
2. Update the transaction data model: add a "category" field (string)
3. Default category when the form loads: "Other"
4. When "Income" type is selected, automatically set the category to "Income"
5. When "Expense" type is selected, reset the category to "Other" if it was "Income"
6. Display the category on each transaction list item as a small pill/badge, muted colour
7. Update localStorage save/load logic to include the category field
8. Do not change the existing add-transaction logic, balance calculation, or list rendering beyond adding the category display

Use the existing colour scheme. No new libraries.
```

**Step 3: Test categories**

1. Add an income transaction — confirm category auto-sets to "Income".
2. Add an expense transaction with category "Food" — confirm it appears with the Food badge.
3. Refresh the page — confirm categories persisted correctly.

Commit:

```
git add index.html
git commit -m "feat: add transaction categories"
```

**Step 4: Add filter by category**

```
Add category filter pills to the Budget Tracker. Requirements:

- Below the "Transactions" heading, show a row of filter pills: "All", "Food", "Transport", "Entertainment", "Income", "Other"
- "All" is selected by default (active style: solid background matching the accent colour)
- Clicking a pill filters the transaction list to show only transactions with that category
- Clicking "All" shows all transactions
- The balance display always shows the TOTAL balance across all transactions regardless of the active filter
- The empty state message ("No transactions yet") should show if the filtered list is empty
- Do not change any other feature — only add the filter pills and filtering logic
```

**Step 5: Test filtering**

1. Add transactions across multiple categories.
2. Click each category pill — confirm only matching transactions appear.
3. Click "All" — confirm all transactions return.
4. Confirm the balance does not change when you apply a filter.

Commit:

```
git add index.html
git commit -m "feat: add category filter pills"
```

**Step 6: Add summary stat cards**

```
Add summary stat cards to the Budget Tracker. Requirements:

- Add a row of three stat cards between the form and the transaction list
- Card 1: "Total Income" — sum of all income transactions, formatted as currency, displayed in green
- Card 2: "Total Expenses" — sum of all expense transactions, formatted as currency, displayed in red
- Card 3: "Savings Rate" — (Total Income - Total Expenses) / Total Income * 100, displayed as a percentage with one decimal place (e.g. "32.4%"), displayed in green if positive, red if negative or zero; show "—" if Total Income is 0
- Update the stats whenever a transaction is added, removed, or edited
- Match the existing card surface colour (#1a1a22)
- Do not change any other feature
```

**Step 7: Test summary stats**

1. Add a $1000 income transaction — Total Income should show $1,000.00, Savings Rate 100.0%.
2. Add a $300 expense — Total Expenses $300.00, Savings Rate 70.0%.
3. Add another $200 expense — Total Expenses $500.00, Savings Rate 50.0%.

Commit:

```
git add index.html
git commit -m "feat: add summary stat cards"
```

**Step 8: Add delete transaction**

```
Add delete functionality to the Budget Tracker. Requirements:

- Add a delete button (×) to each transaction list item, aligned to the right
- The delete button should only be visible on hover of the transaction item (opacity 0 by default, opacity 1 on parent hover)
- Clicking delete removes the transaction from the transactions array
- After deletion: re-render the list, recalculate the balance, update summary stats, save to localStorage
- Add a brief fade-out animation before the item is removed (opacity from 1 to 0, 0.2s ease-out) — only animate opacity and transform
- Do not change any other feature
```

**Step 9: Test delete**

1. Add three transactions.
2. Hover over a transaction — confirm the × button appears.
3. Click × — confirm the item fades out and disappears.
4. Confirm the balance and summary stats update immediately.
5. Refresh — confirm the deleted transaction is gone.

Commit:

```
git add index.html
git commit -m "feat: add delete transaction"
```

**Step 10: Add inline edit**

```
Add inline edit functionality to the Budget Tracker. Requirements:

- Clicking on a transaction item (not on the delete button) opens it in an editable state
- In edit mode, the item's description, amount, category, and type become editable fields
- Show a "Save" button and a "Cancel" button on the item
- Saving updates the transaction in the array, re-renders the list, recalculates balance, updates summary stats, saves to localStorage
- Cancelling restores the item to its display state without any changes
- Only one item can be in edit mode at a time — if a second item is clicked while one is being edited, save the first one before opening the second (or cancel — your choice, just be consistent)
- Do not change any other feature
```

**Step 11: Test inline edit**

1. Click a transaction item — confirm it enters edit mode.
2. Change the description and amount — click Save.
3. Confirm the list updates with the new values, and the balance recalculates.
4. Click another item — confirm the first exits edit mode.
5. Open an item for editing — click Cancel — confirm no changes were made.
6. Refresh — confirm edits persisted.

Commit:

```
git add index.html
git commit -m "feat: add inline edit for transactions"
```

**Step 12: Add CSV export**

```
Add a CSV export button to the Budget Tracker. Requirements:

- Add an "Export CSV" button in the header area, near the app title
- Clicking it generates a CSV file from the current transactions array
- CSV columns: Date, Description, Category, Type, Amount
- Date format in CSV: YYYY-MM-DD
- Amount: always a positive number (no + or - prefix in the CSV)
- Type: "income" or "expense" (lowercase)
- The file should download automatically to the user's Downloads folder with the filename: budget-tracker-[YYYY-MM-DD].csv where the date is today's date
- Use a Blob and a temporary anchor element to trigger the download — no server needed
- The export should include all transactions regardless of the active category filter
- Do not change any other feature
```

**Step 13: Test CSV export**

1. Add several transactions across different categories.
2. Click "Export CSV".
3. Open the downloaded file in Excel or Google Sheets.
4. Confirm all transactions are present with correct columns, dates, and amounts.

Commit:

```
git add index.html
git commit -m "feat: add CSV export"
```

**Step 14: Update CLAUDE.md**

```
Create a CLAUDE.md file for this project. This is a single-file vanilla JavaScript Budget Tracker app (index.html). Include:
1. What the app does (2 sentences)
2. The data model for a transaction (all fields and types)
3. The localStorage key used
4. The list of categories
5. The rule: do not use any JavaScript frameworks or libraries — vanilla JS only
6. The rule: only animate transform and opacity — no layout properties
7. The existing colour scheme (exact hex values)
```

**Step 15: Deploy to GitHub Pages**

Create a new repository on GitHub named `budget-tracker`. Then:

```
git remote add origin https://github.com/YOUR-USERNAME/budget-tracker.git
git branch -M main
git push -u origin main
```

Enable GitHub Pages in the repository settings:
- Settings → Pages → Source: Deploy from a branch → Branch: main → Folder: / (root) → Save

Your app will be live at `https://YOUR-USERNAME.github.io/budget-tracker/` within 1-2 minutes.

---

## Practical workflow

1. Open Claude Code in budget-tracker folder: `claude`
2. Confirm Day 12 features work before starting
3. Add categories — Step 2 prompt → test → `git commit -m "feat: add transaction categories"`
4. Add filter pills — Step 4 prompt → test → `git commit -m "feat: add category filter pills"`
5. Add summary stats — Step 6 prompt → test → `git commit -m "feat: add summary stat cards"`
6. Add delete — Step 8 prompt → test → `git commit -m "feat: add delete transaction"`
7. Add inline edit — Step 10 prompt → test → `git commit -m "feat: add inline edit for transactions"`
8. Add CSV export — Step 12 prompt → test, open file in spreadsheet app → `git commit -m "feat: add CSV export"`
9. Update CLAUDE.md — Step 14 prompt → `git add CLAUDE.md && git commit -m "docs: add CLAUDE.md"`
10. Create GitHub repo, push, enable GitHub Pages
11. Wait 1-2 minutes, open live URL, confirm app loads and works

---

## Common mistakes

**Mistake 1: Asking for multiple features in one prompt**

Asking for "categories, filtering, and stats all at once" produces a rewrite of the existing JavaScript that quietly breaks balance calculation or localStorage. One feature at a time is not slower — it is faster because debugging is isolated.

Fix: If you sent a multi-feature prompt and something broke, isolate the broken feature and revert to the last working commit, then add features one at a time:

```
git revert HEAD
```

Or if you have not committed since the break:

```
git checkout -- index.html
```

Then start the broken feature again with a focused single-feature prompt.

**Mistake 2: Filtering the balance along with the transaction list**

Students often ask Claude to "show filtered balance" meaning they want the balance to reflect the current filter. This is rarely what the user actually needs and makes the app confusing — the true balance disappears when you filter by a single category.

Fix: Be explicit in your filter prompt that the balance must always reflect all transactions. If Claude filters the balance anyway:

```
The balance should always show the total across all transactions regardless of the active filter. Fix the balance calculation so it uses the full transactions array, not the filtered array.
```

**Mistake 3: CSV export produces a blank or corrupted file**

This usually happens because the Blob is created with the wrong MIME type, or the anchor click is not triggered correctly in all browsers.

Fix: Paste the export function to Claude Code with this prompt:

```
The CSV export is producing a blank or corrupted file. Here is the current export function: [paste function]. Fix it to correctly create a Blob with type "text/csv;charset=utf-8;", build the CSV string with proper line endings (\r\n), and trigger the download using a temporary anchor element with the download attribute. Make it work in Chrome, Firefox, and Safari.
```

---

## Your turn

Using the prompts in the walkthrough, add all six features to your Day 12 Budget Tracker one at a time. After all six are working, export a CSV, open it in a spreadsheet, then deploy to GitHub Pages and open the live URL.

Expected output: A live URL (e.g. `https://your-username.github.io/budget-tracker/`) where the full app works — categories display on each item, filter pills show only matching transactions, stat cards show income/expenses/savings rate, each item can be deleted and edited, and the Export CSV button downloads a valid file.

Failure state: GitHub Pages shows a 404 after deployment. Most likely cause: the repository name does not match the branch and folder configuration, or Pages has not been enabled yet. Fix: go to Settings → Pages in your GitHub repository and confirm that Source is set to "Deploy from a branch", Branch is "main", and folder is "/ (root)". If it was already set and showing 404, wait another 2 minutes and refresh — Pages deployment has a delay.

---

## Prompt / Template / Checklist pack

### Budget Tracker Feature Prompt Pack — All 6 Prompts

**Feature 1 — Categories**
```
Add a category field to the Budget Tracker transaction data model. Requirements:

Categories: Food, Transport, Entertainment, Income, Other

Changes needed:
1. Add a category dropdown to the transaction form, between the Amount field and the Type toggle
2. Update the transaction data model: add a "category" field (string)
3. Default category when the form loads: "Other"
4. When "Income" type is selected, automatically set the category to "Income"
5. When "Expense" type is selected, reset the category to "Other" if it was "Income"
6. Display the category on each transaction list item as a small pill/badge, muted colour
7. Update localStorage save/load logic to include the category field
8. Do not change the existing add-transaction logic, balance calculation, or list rendering beyond adding the category display

Use the existing colour scheme. No new libraries.
```

**Feature 2 — Filter by category**
```
Add category filter pills to the Budget Tracker. Requirements:

- Below the "Transactions" heading, show a row of filter pills: "All", "Food", "Transport", "Entertainment", "Income", "Other"
- "All" is selected by default (active style: solid background matching the accent colour)
- Clicking a pill filters the transaction list to show only transactions with that category
- Clicking "All" shows all transactions
- The balance display always shows the TOTAL balance across all transactions regardless of the active filter
- The empty state message shows if the filtered list is empty
- Do not change any other feature — only add the filter pills and filtering logic
```

**Feature 3 — Summary stat cards**
```
Add summary stat cards to the Budget Tracker. Requirements:

- Add a row of three stat cards between the form and the transaction list
- Card 1: "Total Income" — sum of all income transactions, formatted as currency, displayed in green
- Card 2: "Total Expenses" — sum of all expense transactions, formatted as currency, displayed in red
- Card 3: "Savings Rate" — (Total Income - Total Expenses) / Total Income * 100, displayed as a percentage with one decimal place (e.g. "32.4%"), displayed in green if positive, red if negative or zero; show "—" if Total Income is 0
- Update the stats whenever a transaction is added, removed, or edited
- Match the existing card surface colour (#1a1a22)
- Do not change any other feature
```

**Feature 4 — Delete transaction**
```
Add delete functionality to the Budget Tracker. Requirements:

- Add a delete button (×) to each transaction list item, aligned to the right
- The delete button should only be visible on hover of the transaction item (opacity 0 by default, opacity 1 on parent hover)
- Clicking delete removes the transaction from the transactions array
- After deletion: re-render the list, recalculate the balance, update summary stats, save to localStorage
- Add a brief fade-out animation before the item is removed (opacity from 1 to 0, 0.2s ease-out) — only animate opacity and transform
- Do not change any other feature
```

**Feature 5 — Inline edit**
```
Add inline edit functionality to the Budget Tracker. Requirements:

- Clicking on a transaction item (not on the delete button) opens it in an editable state
- In edit mode, the item's description, amount, category, and type become editable fields
- Show a "Save" button and a "Cancel" button on the item
- Saving updates the transaction in the array, re-renders the list, recalculates balance, updates summary stats, saves to localStorage
- Cancelling restores the item to its display state without any changes
- Only one item can be in edit mode at a time
- Do not change any other feature
```

**Feature 6 — CSV export**
```
Add a CSV export button to the Budget Tracker. Requirements:

- Add an "Export CSV" button in the header area, near the app title
- Clicking it generates a CSV file from the current transactions array
- CSV columns: Date, Description, Category, Type, Amount
- Date format in CSV: YYYY-MM-DD
- Amount: always a positive number (no + or - prefix in the CSV)
- Type: "income" or "expense" (lowercase)
- The file downloads automatically with filename: budget-tracker-[YYYY-MM-DD].csv where the date is today's date
- Use a Blob and a temporary anchor element — no server needed
- The export includes all transactions regardless of the active category filter
- Do not change any other feature
```

---

### Completed CLAUDE.md Template for Budget Tracker

```markdown
# Budget Tracker — CLAUDE.md

## What this app does
A personal budget tracker that runs entirely in the browser. Users add income and expense transactions, which are saved to localStorage and persist across page refreshes.

## Tech stack
- Single file: index.html
- Vanilla JavaScript only — no frameworks, no libraries
- Tailwind CSS via CDN for styling
- localStorage for persistence

## Transaction data model
Each transaction is an object with these fields:
- id: string (Date.now().toString())
- description: string
- amount: number (always positive float)
- type: "income" | "expense"
- category: "Food" | "Transport" | "Entertainment" | "Income" | "Other"
- date: ISO date string (new Date().toISOString())

## localStorage
Key: "budget-tracker-transactions"
Format: JSON.stringify(transactions array)
Load on: DOMContentLoaded
Save on: every mutation (add, delete, edit)

## Categories
Food, Transport, Entertainment, Income, Other

## Colour scheme
- Background: #0f0f13
- Card surface: #1a1a22
- Income accent: #22c55e
- Expense accent: #ef4444
- Primary text: #e2e8f0
- Muted text: #64748b

## Rules
- Vanilla JavaScript only — do not add any JS frameworks or npm packages
- Only animate transform and opacity — never animate width, height, color, background, margin, or padding
- Balance always reflects ALL transactions regardless of active category filter
- Validate all form inputs before creating a transaction object
- One item in edit mode at a time maximum
```

---

### Day 13 Build Checklist

- [ ] Day 12 features confirmed working before starting Day 13
- [ ] Categories added — auto-sets to Income when Income type selected, persists to localStorage
- [ ] Categories tested — committed: `feat: add transaction categories`
- [ ] Filter pills added — All, Food, Transport, Entertainment, Income, Other
- [ ] Filter tested — balance does not change when filter is applied, committed: `feat: add category filter pills`
- [ ] Summary stat cards added — Total Income, Total Expenses, Savings Rate
- [ ] Stats tested — update on add/delete/edit, committed: `feat: add summary stat cards`
- [ ] Delete added — × button visible on hover, fade-out animation, updates balance and stats
- [ ] Delete tested — item removed from localStorage on refresh, committed: `feat: add delete transaction`
- [ ] Inline edit added — click to edit, Save/Cancel buttons, one item at a time
- [ ] Edit tested — edits persist after refresh, committed: `feat: add inline edit for transactions`
- [ ] CSV export added — correct columns, downloads as .csv file
- [ ] CSV tested — opened in spreadsheet app, all columns correct, committed: `feat: add CSV export`
- [ ] CLAUDE.md created and committed: `docs: add CLAUDE.md`
- [ ] GitHub repository created
- [ ] Code pushed to main branch
- [ ] GitHub Pages enabled in repository settings
- [ ] Live URL confirmed working in browser
