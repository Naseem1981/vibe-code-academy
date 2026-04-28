# Day 19: Testing and Code Review

## Outcome artifact

A Playwright test run confirms all 4 critical user flows work: login, add contact, add and move a deal, and add an activity. A Code Review has been generated using the `/review` command, and all critical issues have been resolved and committed. ClearCRM is clean, tested, and ready for production deploy.

---

## The core idea

Before you ship anything to real users, you answer two questions:

1. Does it work? (Playwright testing)
2. Is it safe and correct? (Code Review)

These are not optional. Every real product goes through some version of this process before production. At a company, QA engineers write the tests and senior engineers review the code. As a solo builder using AI tools, Playwright and the Code Review plugin give you both.

Playwright drives a real browser and simulates exactly what a user does — it clicks buttons, fills forms, navigates pages, and verifies what it sees. You run it through Claude Code: you write a plain-English prompt, Claude translates it into Playwright actions, and reports the result.

The Code Review plugin (`/review`) runs multiple AI agents in parallel across your codebase. It checks for security issues, performance problems, incorrect logic, and missing error handling. It gives you a prioritised list of issues ranked by severity. You fix the critical ones, re-run the review, and ship.

---

## Part 1 — Playwright testing

The Playwright plugin is already installed from Day 1 of Level 2. You run tests by giving Claude Code prompts that describe what to test. You do not write `.spec.ts` files for this workflow — you use Playwright through the plugin's browser automation capability.

Make sure your local dev server is running before you test:

```bash
# from your project root
npm run dev
# Expected output: ✓ Ready on http://localhost:3000
```

Also make sure you have a test user in Supabase. Go to Supabase → Authentication → Users. If you do not have a test user, add one: email `test@example.com`, password `testpass123`.

---

### Test 1 — Auth flow

Verify the app protects routes and login works correctly.

```
Use Playwright to open http://localhost:3000.
Verify it redirects to /login.
Fill in email: test@example.com and password: testpass123.
Click Sign In.
Tell me what happens — does it redirect to the dashboard or show an error?
```

What to expect: Playwright opens Chrome, navigates to `localhost:3000`, the app redirects to `/login` (which confirms your middleware is protecting routes), Playwright fills in the credentials and clicks Sign In, and the app redirects to `/dashboard` or `/contacts`. If it shows "Invalid credentials", the test user is not set up correctly in Supabase.

If Playwright reports the page is already on `/dashboard` without redirecting to `/login`: your middleware is not protecting the root route. Open `middleware.ts` and check the matcher pattern includes `/` and all protected routes.

---

### Test 2 — Add contact

Verify the add contact form works end to end.

```
Use Playwright to navigate to http://localhost:3000/contacts.
Click the "Add Contact" button.
Fill in: First Name: Sarah, Last Name: Johnson, Email: sarah@techcorp.co.za, Phone: 021-555-0101.
Click Save.
Verify Sarah Johnson appears in the contacts list.
Take a screenshot and show me the result.
```

What to expect: Playwright finds the "Add Contact" button, clicks it (opening a modal or navigating to `/contacts/new`), fills in all four fields, clicks Save, and then confirms Sarah Johnson is visible in the contacts list. Playwright takes a screenshot and shows it to you in the Claude Code conversation.

If the contact does not appear: Playwright will report it. Give this follow-up prompt:

```
The contact did not appear after saving. 
Use Playwright to check the browser console for errors on the /contacts page.
Also check the network tab — was there a request to Supabase and what was the response?
```

---

### Test 3 — Add deal and move stage

Verify the full deal lifecycle: create a deal, open it, change its stage, verify the Kanban board updates.

```
Use Playwright to navigate to http://localhost:3000/deals.
Click "Add Deal". Fill in Title: "Website Redesign", Value: 15000, Stage: Lead, contact: Sarah Johnson.
Click Save. Verify the deal appears in the Lead column.
Click the deal card to open the detail page.
Click the "Qualified" stage button.
Go back to /deals. Verify the deal now appears in the Qualified column.
```

What to expect: Playwright creates the deal, verifies it appears in the Lead column, opens the detail page, clicks the Qualified stage button, navigates back to the Kanban board, and verifies the deal has moved to Qualified. The Kanban board uses real-time data — the move should be reflected immediately on page load.

If the deal does not move to Qualified: the stage update server action may not be calling `revalidatePath('/deals')`. Open the server action and confirm `revalidatePath` is called after the update.

---

### Test 4 — Add activity

Verify notes can be added to a contact and appear in the activity feed.

```
Use Playwright to navigate to the contacts page and click on Sarah Johnson.
On her detail page, click "Add Note".
Type: "Had a great intro call. Very interested in the redesign project."
Click Save.
Verify the note appears in the activity feed with the correct timestamp.
```

What to expect: Playwright opens Sarah Johnson's contact page, finds the "Add Note" button, clicks it, types the note, saves it, and verifies the note text appears in the activity feed below. The timestamp should read "Just now" or the current time.

If the note does not appear: check that the activity feed is re-fetching after the server action. The activity feed may be a Server Component that needs `revalidatePath('/contacts/[id]')` called by the server action.

---

## Part 2 — Code Review

### Step 1 — Make sure all changes are committed

The Code Review plugin reviews your committed code. Run:

```bash
git status
```

If you see uncommitted changes, commit them:

```bash
git add .
git commit -m "chore: pre-review cleanup"
```

---

### Step 2 — Run the Code Review

In the Claude Code conversation, type:

```
/review
```

The Code Review plugin starts multiple AI agents. Each agent reviews a different area of the codebase:

- Security agent: looks for exposed secrets, missing auth checks, SQL injection risks, XSS vulnerabilities
- Performance agent: looks for N+1 queries, unnecessary re-renders, large bundle imports
- Correctness agent: looks for logic errors, missing error handling, type mismatches
- Best practices agent: looks for code patterns that work but will cause problems at scale

The review takes 1–3 minutes. Do not interrupt it.

---

### Step 3 — Read and action the review output

A realistic Code Review output for ClearCRM looks like this:

---

**Code Review Results — ClearCRM**

---

**CRITICAL (1)**

**Missing auth check in importContacts server action**
File: `app/contacts/actions.ts`, line 12

The `importContacts` server action calls `supabase.auth.getSession()` but does not return early if the session is null. On line 12, the code destructures `session` from the response but does not check `if (!session) return`. This means an unauthenticated user who discovers the server action endpoint can call it directly and insert rows into the contacts table.

Fix: add an auth guard immediately after `getSession()`:

```ts
const { data: { session } } = await supabase.auth.getSession()
if (!session) return { error: 'Not authenticated', imported: 0 }
```

---

**MEDIUM (1)**

**No error boundary on the Reports page**
File: `app/reports/page.tsx`

The reports page runs 4 queries in `Promise.all`. If any query throws an unhandled error, the entire page throws and Next.js renders the nearest error boundary. There is no `error.tsx` file in `app/reports/`. If Supabase is temporarily unavailable, users see a raw error page.

Fix: create `app/reports/error.tsx` with a friendly error message and a "Try again" button.

---

**LOW (1)**

**Unused import in HorizontalBarChart**
File: `components/HorizontalBarChart.tsx`, line 1

`import React from 'react'` is present but unused. Next.js 14 with the App Router does not require explicit React imports. Remove it to keep imports clean.

---

### Step 4 — Fix the critical issue

The critical issue is the missing auth check. Give Claude this prompt:

```
Fix the critical issue flagged in the code review: the importContacts server action in app/contacts/actions.ts does not return early when the session is null. Add an auth guard on line 12 that returns { error: 'Not authenticated', imported: 0 } if !session.
```

Claude will open `app/contacts/actions.ts` and add the guard. Verify the fix by reading the file.

Also fix the medium and low issues while you are here:

```
Create app/reports/error.tsx — a simple error boundary component for the reports page. It should show a friendly message: "Could not load reports. Try refreshing the page." with a "Refresh" button that calls window.location.reload().

Also remove the unused React import from components/HorizontalBarChart.tsx.
```

---

### Step 5 — Re-run the review

After fixing the issues, commit the changes and re-run:

```bash
git add .
git commit -m "fix: resolve critical code review issues before production deploy"
```

Then in Claude Code:

```
/review
```

The second review should return no critical issues. Medium and low issues may remain — use your judgment on whether to fix them now or file them as future improvements.

---

## Practical workflow

Run the Playwright tests in order. Each test builds on the previous one — Test 2 creates the contact that Test 3 uses, and Test 3 creates the deal that (implicitly) populates the activity log used in Test 4. If Test 2 fails, fix it before running Test 3.

When Playwright reports a failure, ask for a screenshot:

```
Take a screenshot of the current state and show it to me.
```

Visual inspection is faster than reading DOM output. You can see in one second whether the button was not found, the form did not submit, or the data did not appear.

---

## Common mistakes

**Playwright cannot find the button.**
Button text in the prompt must match exactly what is in the DOM. If your button says "New Contact" but the prompt says "Add Contact", Playwright fails to find it. Check the exact button text in your app and update the prompt.

**The auth test fails because the test user does not exist.**
Go to Supabase → Authentication → Users → Add user. Set email `test@example.com`, password `testpass123`, confirm email. If your app requires email confirmation, you must confirm the user in the Supabase dashboard or turn off email confirmation in Supabase → Authentication → Providers → Email → disable "Confirm email".

**Code Review takes more than 5 minutes and shows no output.**
The `/review` command requires the Code Review plugin to be active. If it times out, the plugin may not be correctly configured. Check your `.claude/settings.json` for the plugin entry. Restart Claude Code and try again.

**The second Code Review still shows the critical issue.**
You fixed the code but did not commit before re-running. The Code Review plugin reviews committed code. Always commit your fixes first.

**Playwright tests pass locally but the app breaks in production.**
Local tests use `http://localhost:3000` with local environment variables. Production uses your Vercel deployment with Vercel environment variables. A common difference: your local `.env.local` has `NEXT_PUBLIC_SUPABASE_URL` set, but you forgot to add it to Vercel's environment variables. Check Vercel → Settings → Environment Variables before deploying.

---

## Your turn

1. Run all 4 Playwright tests. Screenshot the result of each one. Fix any failures before moving on.
2. Run `/review`. Read the full output. Fix all critical issues. Fix at least one medium issue.
3. Write a 5th Playwright test for the email templates page:

```
Use Playwright to navigate to http://localhost:3000/email-templates.
Click "New Template".
Fill in Subject: "Welcome to our service", Body: "Hi {{first_name}}, we are thrilled to have you on board."
Click Save.
Verify the template appears in the template list.
Click the "Copy" button on the template.
Verify the browser shows a success indicator (tooltip or toast saying "Copied").
```

4. After all tests pass and the review is clean, commit with `git add . && git commit -m "fix: resolve critical code review issues before production deploy"`.

---

## Prompt / Template / Checklist pack

### Test prompts (copy-paste ready)

**Test 1 — Auth flow**

```
Use Playwright to open http://localhost:3000.
Verify it redirects to /login.
Fill in email: test@example.com and password: testpass123.
Click Sign In.
Tell me what happens — does it redirect to the dashboard or show an error?
```

**Test 2 — Add contact**

```
Use Playwright to navigate to http://localhost:3000/contacts.
Click the "Add Contact" button.
Fill in: First Name: Sarah, Last Name: Johnson, Email: sarah@techcorp.co.za, Phone: 021-555-0101.
Click Save.
Verify Sarah Johnson appears in the contacts list.
Take a screenshot and show me the result.
```

**Test 3 — Add deal and move stage**

```
Use Playwright to navigate to http://localhost:3000/deals.
Click "Add Deal". Fill in Title: "Website Redesign", Value: 15000, Stage: Lead, contact: Sarah Johnson.
Click Save. Verify the deal appears in the Lead column.
Click the deal card to open the detail page.
Click the "Qualified" stage button.
Go back to /deals. Verify the deal now appears in the Qualified column.
```

**Test 4 — Add activity**

```
Use Playwright to navigate to the contacts page and click on Sarah Johnson.
On her detail page, click "Add Note".
Type: "Had a great intro call. Very interested in the redesign project."
Click Save.
Verify the note appears in the activity feed with the correct timestamp.
```

---

## Deliverable — Pre-deploy testing checklist

Complete every item on this checklist before pushing to production. Check it off only when you have personally verified it — not when you think it should work.

**Authentication**
- [ ] Navigating to `/contacts` without being logged in redirects to `/login`
- [ ] Logging in with valid credentials redirects to the dashboard
- [ ] Logging in with invalid credentials shows an error message
- [ ] Logging out redirects to `/login` and clears the session

**Contacts**
- [ ] A new contact can be added — it appears in the contacts list immediately
- [ ] A contact can be searched by name or email — the list filters correctly
- [ ] A contact detail page loads and shows all fields
- [ ] A contact can be edited and the change persists after a page refresh

**Companies**
- [ ] A new company can be added — it appears in the companies list
- [ ] A company can be linked to a contact

**Deals**
- [ ] A new deal can be added and appears in the correct Kanban column
- [ ] Clicking a deal opens the detail page
- [ ] The stage can be changed on the detail page and the Kanban board reflects it after navigating back

**Activities**
- [ ] A note can be added on a contact detail page — it appears in the activity feed
- [ ] The activity timestamp is correct (shows "Just now" or current time)

**Dashboard**
- [ ] The stats cards show non-zero numbers (assuming there is data)
- [ ] The chart renders without errors

**Email templates**
- [ ] A template can be created and appears in the list
- [ ] The copy button copies the template to the clipboard

**Reports**
- [ ] All 4 report sections load without error
- [ ] Charts show data (assuming deals exist in the database)

**Mobile**
- [ ] At 375px viewport, the sidebar is hidden and the hamburger is visible
- [ ] The contacts list reformats as cards on mobile

**Performance**
- [ ] The dashboard page loads in under 3 seconds on a standard connection (test in Chrome DevTools → Network tab → check "No throttling" or set to "Fast 3G" for a conservative test)

**Security**
- [ ] No Supabase service role key is visible in the browser (DevTools → Network → any request → check response headers and body)
- [ ] Code Review has been run and all critical issues are resolved

**Final state**
- [ ] All changes are committed: `git status` shows "nothing to commit, working tree clean"
- [ ] All 4 Playwright tests have passed in this session
