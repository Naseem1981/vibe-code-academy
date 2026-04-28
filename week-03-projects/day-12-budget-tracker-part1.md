# Day 12: Budget Tracker — Part 1

## Outcome artifact

By the end of this day you will have a working Budget Tracker with a functional add-transaction form, a colour-coded transaction list (green for income, red for expense), a live running balance that updates automatically, and localStorage persistence — confirmed by adding three transactions and refreshing the page to see them still there.

---

## The core idea

This is your first real multi-feature project. Up to now you have built things that demonstrate one or two ideas at a time. A budget tracker has a UI, user input, data logic, data persistence, and visual feedback all working together. That is not more difficult than what you have already done — it just requires a different approach to how you sequence your prompts.

The approach is: scaffold everything first, then add features one at a time. Your first prompt builds the full visual skeleton — every section, every panel, every placeholder — with nothing wired up. You can see the whole app before a single feature works. Then you add features in order, confirming each one before moving to the next. If you ask for everything at once, Claude will produce tangled code where features depend on each other in unexpected ways and debugging becomes exponentially harder.

localStorage is the simplest form of persistence available in a browser. It stores data as strings in the browser itself — no server, no database, no account needed. Every time a transaction is added or removed, you save the full transactions array to localStorage as JSON. Every time the page loads, you check if anything is saved and load it. This is enough for a personal tool that one person uses on one device. It is genuinely useful, and it is the right scope for Day 12.

---

## Step-by-step walkthrough

**Step 1: Create the project folder**

```
mkdir budget-tracker
cd budget-tracker
git init
```

Open Claude Code inside the new folder:

```
claude
```

**Step 2: Scaffold the full UI — no functionality yet**

This single prompt builds the entire visual structure. Do not ask for working features yet.

```
Create a single-file index.html Budget Tracker app. Scaffold the full UI with all sections visible but no JavaScript functionality yet. 

Layout requirements:
- Full-page app, dark background (#0f0f13), clean sans-serif font
- Top header section: app title "Budget Tracker" on the left, large balance display in the centre showing "$0.00" — green if positive, red if negative
- Below header: a form panel with four inputs:
  1. Description text input (placeholder: "e.g. Salary, Groceries...")
  2. Amount number input (placeholder: "0.00")
  3. Type selector: two toggle buttons — "Income" and "Expense" — only one active at a time
  4. Submit button labelled "Add Transaction"
- Below the form: a transaction list section with a heading "Transactions" and an empty state message "No transactions yet — add one above"
- Colour scheme: background #0f0f13, card surfaces #1a1a22, accent for income #22c55e, accent for expense #ef4444, text #e2e8f0, muted text #64748b
- Use Tailwind CSS via CDN
- No JavaScript at all in this first version — just the HTML and CSS structure

Make the layout clean, well-spaced, and mobile-first responsive.
```

**Step 3: Review the scaffold in the browser**

Open `index.html` in your browser. You should see the full layout — header, balance display, form, empty state — with nothing working yet. Confirm the visual looks right before proceeding.

If anything looks wrong (wrong colours, broken layout), fix it now before adding any JavaScript.

```
The [specific element] looks wrong — [describe what you see]. Fix only the visual issue, do not add any JavaScript yet.
```

**Step 4: Commit the scaffold**

```
git add index.html
git commit -m "feat: scaffold budget tracker UI"
```

**Step 5: Add the transaction data model and add-transaction feature**

Now add the first working feature: submitting the form adds a transaction to the list.

```
Add the add-transaction feature to the Budget Tracker. Requirements:

Data model: each transaction is an object with these fields:
- id: unique string (use Date.now().toString())
- description: string
- amount: number (always positive)
- type: "income" or "expense"
- date: ISO date string (new Date().toISOString())

Behaviour:
- When the form is submitted, validate: description must not be empty, amount must be a positive number greater than 0
- If validation fails, show an inline error message below the relevant field (red text, no alert() dialogs)
- If validation passes, create a new transaction object and add it to the transactions array
- Clear the form inputs after successful submission
- Re-render the transaction list

Transaction list item display:
- Each item shows: description on the left, amount on the right
- Amount formatted as currency: "+ $24.00" for income (green), "- $24.00" for expense (red)
- Below the description: the date in a readable format (e.g. "Apr 28, 2026") in muted text
- A thin left border: green for income, red for expense
- Items appear in reverse chronological order (newest first)

Remove the "No transactions yet" empty state when the list has items. Show it again when the list is empty.

Use vanilla JavaScript only — no frameworks or libraries.
```

**Step 6: Test the add-transaction feature**

In your browser:
1. Try submitting with empty description — confirm error message appears.
2. Try submitting with 0 amount — confirm error appears.
3. Add a valid income transaction: "Salary", 3000, Income. Confirm it appears in the list with green colour.
4. Add a valid expense transaction: "Groceries", 85.50, Expense. Confirm it appears with red colour.

If anything does not work, describe the exact problem to Claude Code:

```
When I submit the form with [these values], [this happens] instead of [expected behaviour]. The error appears to be in the form submission or list rendering. Here is what I see in the browser console: [paste console errors if any]. Fix it.
```

**Step 7: Commit the add-transaction feature**

```
git add index.html
git commit -m "feat: add transaction form and list rendering"
```

**Step 8: Add the running balance**

```
Add the running balance calculation to the Budget Tracker. Requirements:
- Calculate balance as: sum of all income amounts minus sum of all expense amounts
- Display the balance in the header balance display
- Format as currency with two decimal places and a $ sign
- If balance is positive or zero: display in green (#22c55e)
- If balance is negative: display in red (#ef4444)
- Recalculate and re-render the balance every time a transaction is added or removed
- The balance display should update without any page reload
```

**Step 9: Test the running balance**

In your browser:
1. Add an income of $1000. Balance should show +$1,000.00 in green.
2. Add an expense of $200. Balance should show +$800.00 in green.
3. Add an expense of $900. Balance should show -$100.00 in red.

**Step 10: Commit the balance feature**

```
git add index.html
git commit -m "feat: add live running balance calculation"
```

**Step 11: Add localStorage persistence**

```
Add localStorage persistence to the Budget Tracker. Requirements:

Saving:
- Every time the transactions array changes (item added or removed), save the full array to localStorage
- Key: "budget-tracker-transactions"
- Save as JSON: localStorage.setItem("budget-tracker-transactions", JSON.stringify(transactions))

Loading:
- On page load (DOMContentLoaded), check if the key exists in localStorage
- If it does, parse it and use it as the initial transactions array
- If it does not, start with an empty array
- After loading, re-render the full transaction list and recalculate the balance

Do not change any other behaviour — only add the save/load logic.
```

**Step 12: Test localStorage persistence**

1. Add three transactions: one income, two expenses.
2. Note the balance displayed.
3. Close the browser tab.
4. Reopen `index.html` (or refresh the page).
5. Confirm all three transactions are still there and the balance is the same.

**Step 13: Commit localStorage**

```
git add index.html
git commit -m "feat: add localStorage persistence"
```

---

## Practical workflow

1. `mkdir budget-tracker && cd budget-tracker && git init`
2. Open Claude Code: `claude`
3. Scaffold UI — Step 2 prompt
4. Open browser, review visual layout
5. Fix any visual issues before adding JS
6. `git add index.html && git commit -m "feat: scaffold budget tracker UI"`
7. Add transaction feature — Step 5 prompt
8. Browser test: empty validation, income item, expense item
9. `git add index.html && git commit -m "feat: add transaction form and list rendering"`
10. Add running balance — Step 8 prompt
11. Browser test: add income, add expenses, confirm balance sign and colour
12. `git add index.html && git commit -m "feat: add live running balance calculation"`
13. Add localStorage — Step 11 prompt
14. Browser test: add 3 transactions, close tab, reopen, confirm persistence
15. `git add index.html && git commit -m "feat: add localStorage persistence"`

---

## Common mistakes

**Mistake 1: Asking for everything in one prompt**

Scaffolding, transaction logic, balance calculation, and localStorage all in a single prompt produces deeply tangled code. When one part breaks, you cannot isolate which feature caused it.

Fix: Always use the scaffold-first strategy. Build the visual shell, commit it, then add one feature at a time. If you already sent a single large prompt and got tangled code, start the file over:

```
This file has gotten messy. Let's start over with the scaffold only — full UI, no JavaScript. I will add features one at a time after.
```

**Mistake 2: Testing in a `file:///` URL instead of a local server**

localStorage works fine with `file:///`, but some browsers restrict it or behave inconsistently. More importantly, if you later add fetch calls or module scripts, they will fail silently on `file:///`.

Fix: Serve locally. In your project folder:

```
npx serve .
```

This starts a local server at `http://localhost:3000` (or similar). Always test from that URL.

**Mistake 3: Not validating before storing**

If the amount input is empty or contains text, `parseFloat("")` returns `NaN`. Adding `NaN` to your transactions array corrupts the balance calculation silently.

Fix: Always validate before creating a transaction object. Ask Claude Code to add explicit validation if it is missing:

```
Add validation to the form submission: description must be a non-empty string, amount must be a finite number greater than 0. If either fails, show a clear inline error message and do not create the transaction.
```

---

## Your turn

Using the prompts in the walkthrough above, build the Budget Tracker from scratch: scaffold, then add each feature in order. When you have all four features working (form, list, balance, localStorage), add three transactions — one income and two expenses — then close the browser tab and reopen it.

Expected output: All three transactions are still displayed, and the balance matches what it showed before you closed the tab.

Failure state: Transactions disappear on refresh. Most likely cause: the save-to-localStorage call is not being triggered after adding a transaction, or it is saving before the transactions array is updated. Fix: paste your JavaScript to Claude Code and ask "The transactions are not persisting on refresh. Check that localStorage.setItem is called after the transactions array is updated, not before. Fix the save/load logic."

---

## Prompt / Template / Checklist pack

### Budget Tracker Starter Prompt (full scaffold)

```
Create a single-file index.html Budget Tracker app. Scaffold the full UI with all sections visible but no JavaScript functionality yet.

Layout requirements:
- Full-page app, dark background (#0f0f13), clean sans-serif font
- Top header section: app title "Budget Tracker" on the left, large balance display in the centre showing "$0.00" — green if positive, red if negative
- Below header: a form panel with four inputs:
  1. Description text input (placeholder: "e.g. Salary, Groceries...")
  2. Amount number input (placeholder: "0.00")
  3. Type selector: two toggle buttons — "Income" and "Expense" — only one active at a time
  4. Submit button labelled "Add Transaction"
- Below the form: a transaction list section with a heading "Transactions" and an empty state message "No transactions yet — add one above"
- Colour scheme: background #0f0f13, card surfaces #1a1a22, accent for income #22c55e, accent for expense #ef4444, text #e2e8f0, muted text #64748b
- Use Tailwind CSS via CDN
- No JavaScript at all in this first version — just the HTML and CSS structure

Make the layout clean, well-spaced, and mobile-first responsive.
```

---

### Feature Addition Prompt 1 — Add Transaction

```
Add the add-transaction feature to the Budget Tracker. Requirements:

Data model: each transaction is an object with these fields:
- id: unique string (use Date.now().toString())
- description: string
- amount: number (always positive)
- type: "income" or "expense"
- date: ISO date string (new Date().toISOString())

Behaviour:
- When the form is submitted, validate: description must not be empty, amount must be a positive number greater than 0
- If validation fails, show an inline error message below the relevant field (red text, no alert() dialogs)
- If validation passes, create a new transaction object and add it to the transactions array
- Clear the form inputs after successful submission
- Re-render the transaction list

Transaction list item display:
- Each item shows: description on the left, amount on the right
- Amount formatted as currency: "+ $24.00" for income (green), "- $24.00" for expense (red)
- Below the description: the date in readable format (e.g. "Apr 28, 2026") in muted text
- A thin left border: green for income, red for expense
- Items appear in reverse chronological order (newest first)

Remove the empty state message when the list has items. Show it again when the list is empty.

Use vanilla JavaScript only — no frameworks or libraries.
```

---

### Feature Addition Prompt 2 — Running Balance

```
Add the running balance calculation to the Budget Tracker. Requirements:
- Calculate balance as: sum of all income amounts minus sum of all expense amounts
- Display the balance in the header balance display
- Format as currency with two decimal places and a $ sign
- If balance is positive or zero: display in green (#22c55e)
- If balance is negative: display in red (#ef4444)
- Recalculate and re-render the balance every time a transaction is added or removed
- The balance display should update without any page reload
```

---

### Feature Addition Prompt 3 — localStorage Persistence

```
Add localStorage persistence to the Budget Tracker. Requirements:

Saving:
- Every time the transactions array changes (item added or removed), save the full array to localStorage
- Key: "budget-tracker-transactions"
- Save as JSON: localStorage.setItem("budget-tracker-transactions", JSON.stringify(transactions))

Loading:
- On page load (DOMContentLoaded), check if the key exists in localStorage
- If it does, parse it and use it as the initial transactions array
- If it does not, start with an empty array
- After loading, re-render the full transaction list and recalculate the balance

Do not change any other behaviour — only add the save/load logic.
```

---

### Day 12 Build Checklist

- [ ] Project folder created, `git init` run
- [ ] Scaffold prompt sent — visual layout matches spec
- [ ] Browser opened, layout reviewed before adding any JS
- [ ] Scaffold committed: `git commit -m "feat: scaffold budget tracker UI"`
- [ ] Add-transaction feature added and tested: empty validation works, income appears green, expense appears red
- [ ] Feature committed: `git commit -m "feat: add transaction form and list rendering"`
- [ ] Running balance added and tested: positive balance green, negative balance red
- [ ] Feature committed: `git commit -m "feat: add live running balance calculation"`
- [ ] localStorage added and tested: transactions survive page refresh
- [ ] Feature committed: `git commit -m "feat: add localStorage persistence"`
- [ ] Final check: add 3 transactions, refresh, confirm all persist with correct balance
