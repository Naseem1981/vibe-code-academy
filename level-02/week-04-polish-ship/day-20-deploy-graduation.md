# Day 20: Deploy and Graduation

## Outcome artifact

ClearCRM is live in production at a Vercel URL. All 5 modules — contacts, companies, deals, activities, email templates — work on the production URL. The student has a link to share. The Level 2 graduation checklist is complete.

---

## The core idea

Shipping is not a technical step. It is a decision. You decide that this version is good enough to exist in the world, and you push it. Every professional developer ships things that are imperfect. The goal of today is not perfection. The goal is a live URL and a lesson in what "done" actually means.

The production deploy is straightforward if you have been pushing to GitHub throughout the course. Vercel watches your `main` branch. When you push, Vercel builds and deploys automatically. You spend most of today on two things: fixing the build errors that only appear in production mode, and verifying that your Supabase configuration is secure.

---

## Part 1 — Final production push

### Step 1 — Run the build locally

Before pushing, run `npm run build` locally. This catches errors that the dev server ignores.

```bash
# from your project root
npm run build
```

A clean build looks like this:

```
▲ Next.js 14.x.x

   Creating an optimized production build ...
 ✓ Compiled successfully
 ✓ Linting and checking validity of types
 ✓ Collecting page data
 ✓ Generating static pages (18/18)
 ✓ Finalizing page optimization

Route (app)                              Size     First Load JS
┌ ○ /                                    1.2 kB         89 kB
├ ○ /contacts                            4.8 kB         93 kB
├ ○ /companies                           3.1 kB         91 kB
├ ○ /deals                               6.2 kB         95 kB
├ ○ /reports                             2.9 kB         91 kB
└ ○ /login                               1.4 kB         89 kB

○  (Static)   prerendered as static content
ƒ  (Dynamic)  server-rendered on demand
```

If the build fails, you will see error output with file paths and line numbers. Here are the three most common build errors and how to fix each one:

---

**Error 1 — Missing environment variable**

```
Error: supabaseUrl is required.
    at new SupabaseClient (...)
```

This means your production environment does not have the Supabase URL environment variable set. The dev server reads from `.env.local` — but Vercel does not. Fix: add the variables to Vercel.

Go to Vercel → your project → Settings → Environment Variables. Add:

```
NEXT_PUBLIC_SUPABASE_URL       = your-project-url.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY  = your-anon-key
```

These are the same values from your `.env.local` file. After adding them, re-deploy.

---

**Error 2 — TypeScript type error**

```
Type error: Argument of type 'string | undefined' is not assignable to parameter of type 'string'.
  app/contacts/page.tsx:45:22
```

TypeScript is stricter in production builds. The dev server runs with `skipLibCheck` and ignores some type errors. Fix: open the file, find line 45, and add a null check or non-null assertion.

Common fix pattern:

```ts
// Before — fails build
const id = searchParams.id

// After — passes build
const id = searchParams.id ?? ''
// or
const id = searchParams.id as string
```

---

**Error 3 — Unused variable or import**

```
./app/reports/page.tsx
Failed to compile.

  3:9  Error: 'data' is defined but never used.  @typescript-eslint/no-unused-vars
```

ESLint runs during the build. Remove the unused variable or prefix it with `_` to signal intentional non-use:

```ts
// Before
const { data, error } = await supabase.from(...)

// After — suppress the warning
const { data: _data, error } = await supabase.from(...)
// Or just don't destructure data if you don't need it:
const { error } = await supabase.from(...)
```

After fixing all errors, run `npm run build` again until you see "Compiled successfully."

---

### Step 2 — Push to GitHub

```bash
# from your project root
git add .
git commit -m "feat: Level 2 complete — ClearCRM production deploy"
git push origin main
```

Expected output:

```
Enumerating objects: 47, done.
Counting objects: 100% (47/47), done.
Delta compression using up to 8 threads
Compressing objects: 100% (31/31), done.
Writing objects: 100% (31/31), 18.24 KiB | 2.61 MiB/s, done.
Total 31 (delta 15), reused 0 (delta 0), pack-reused 0
remote: Resolving deltas: 100% (15/15), completed with 8 local objects.
To https://github.com/yourusername/clearcrm.git
   c7d4e21..e8f5c03  main -> main
```

---

### Step 3 — Watch the Vercel build

Go to your Vercel dashboard → your project → Deployments. You will see a new deployment in progress. Click it to open the build log.

A successful Vercel build log:

```
[00:00:02] Cloning github.com/yourusername/clearcrm (Branch: main, Commit: e8f5c03)
[00:00:04] Cloning completed: 1.822s
[00:00:05] Running "npm install"
[00:00:28] Installing dependencies...
[00:00:29] Running "npm run build"
[00:00:31] ▲ Next.js 14.x.x
[00:00:58] ✓ Compiled successfully
[00:01:02] ✓ Generating static pages (18/18)
[00:01:03] Build Completed in /vercel/output [54s]
[00:01:04] Deploying outputs...
[00:01:08] ✓ Deployment complete
[00:01:08] Production: https://clearcrm-yourusername.vercel.app
```

The production URL appears at the bottom. Click it. Your app is live.

---

### Step 4 — Smoke test the production URL

Open the production URL and manually test each module. Do not assume the local tests cover the production environment — environment variables, Supabase RLS policies, and network conditions can differ.

Test this sequence:

1. Navigate to the root URL — verify it redirects to `/login`
2. Log in with your test credentials — verify it reaches the dashboard
3. Navigate to `/contacts` — verify contacts load
4. Navigate to `/deals` — verify the Kanban board renders
5. Navigate to `/reports` — verify charts appear
6. Download the CSV export — verify the file downloads

If any step fails in production but worked locally, the cause is almost always an environment variable. Go to Vercel → Settings → Environment Variables and verify all variables are set.

---

## Part 2 — Production environment check

### Step 1 — Verify Supabase RLS is enabled

Row Level Security (RLS) is what prevents one user from reading another user's data. If RLS is off, any authenticated user can query all rows in the table — including other users' contacts, deals, and companies.

Go to Supabase dashboard → Table Editor. Click each table: contacts, companies, deals, activities, email_templates. For each one, click the "RLS" column header or look for the shield icon. It should show "RLS enabled."

If RLS is disabled on any table: click the table → Policies → Enable RLS. Then add a policy that allows users to access only their own rows. For the contacts table:

```sql
-- Allow users to select only their own contacts
CREATE POLICY "Users can view their own contacts"
ON contacts FOR SELECT
USING (auth.uid() = user_id);

-- Allow users to insert their own contacts
CREATE POLICY "Users can insert their own contacts"
ON contacts FOR INSERT
WITH CHECK (auth.uid() = user_id);

-- Allow users to update their own contacts
CREATE POLICY "Users can update their own contacts"
ON contacts FOR UPDATE
USING (auth.uid() = user_id);

-- Allow users to delete their own contacts
CREATE POLICY "Users can delete their own contacts"
ON contacts FOR DELETE
USING (auth.uid() = user_id);
```

This assumes your contacts table has a `user_id` column that stores the `auth.uid()` of the user who created the record. If you did not add `user_id` columns during Week 1, this is a critical security fix — add the column and set up the policies before launching to real users.

---

### Step 2 — Verify no API keys are exposed in the browser

Open your production URL in Chrome. Press `F12` → Network tab → reload the page. Click on any request to Supabase (look for requests to `supabase.co` in the URL column). Click the request → Headers tab.

You should see the `apikey` header contains your **anon key** — the key that starts with `eyJ`. This is safe. It is designed to be public.

You must not see your **service role key** anywhere in browser traffic. The service role key bypasses RLS and gives full database access. If it appears in browser requests, you have accidentally used it in a client-side context. Find where `SUPABASE_SERVICE_ROLE_KEY` is referenced and move that code to a server-only file (a server action or API route).

The rule: `SUPABASE_SERVICE_ROLE_KEY` — never in a file with `'use client'`, never passed to a client component, never in an env var starting with `NEXT_PUBLIC_`.

---

### Step 3 — Test auth in production

Sign up with a new email address in the live app. Use a real email you have access to (not your test user). Complete the sign-up flow. If email confirmation is enabled, check your inbox for the confirmation email. Confirm and log in.

Verify you can create a contact as this new user, and that the contact does not appear when you log in as the original test user. This confirms RLS is working correctly.

---

## Part 3 — Optional custom domain

### Step 1 — Add the domain in Vercel

In Vercel → your project → Settings → Domains → Add. Type your domain (e.g. `crm.yourdomain.com`). Vercel shows you the DNS records to add.

---

### Step 2 — Add the DNS record

At your DNS provider (Cloudflare, Namecheap, GoDaddy, etc.), add a CNAME record:

```
Type:    CNAME
Name:    crm         (this creates crm.yourdomain.com)
Value:   cname.vercel-dns.com
TTL:     Auto (or 3600)
```

If you are setting up the root domain (e.g. `yourdomain.com` without a subdomain), use an A record instead:

```
Type:    A
Name:    @           (@ means the root domain)
Value:   76.76.21.21 (Vercel's IP — verify in Vercel dashboard)
TTL:     Auto
```

---

### Step 3 — Wait for DNS propagation

DNS propagation takes 15 minutes to 48 hours, depending on your DNS provider and the TTL value. Cloudflare propagates in under 5 minutes. Some providers take up to 24 hours.

What DNS propagation means: when you update a DNS record, the change has to spread from your DNS provider's servers to every DNS resolver on the internet. These resolvers cache records for the duration of the TTL. Until the cache expires and refreshes, some users (in different locations) may still see the old record. For a new domain, this is not an issue — there is no old record, so propagation is just waiting for Vercel's SSL certificate to be issued.

Vercel issues an SSL certificate (HTTPS) automatically once it verifies the domain. This takes 10–30 minutes after DNS propagates.

Check propagation status at `https://dnschecker.org` — type your domain and see when it resolves correctly across different regions.

---

## Part 4 — Level 2 Graduation

### Graduation checklist

Work through this list. Every item corresponds to a feature you built during Level 2. Check it off only on the production URL — not localhost.

- [ ] ClearCRM is deployed and live at a public Vercel URL (or custom domain)
- [ ] New users can sign up with an email and password on the production URL
- [ ] Existing users can log in and are redirected to the dashboard
- [ ] Contacts can be added, edited, deleted, searched by name, and filtered
- [ ] Companies can be added and linked to contacts via the company selector
- [ ] Deals appear in the correct Kanban column when added
- [ ] Moving a deal stage (e.g. Lead → Qualified) updates the Kanban board immediately
- [ ] Activities (notes, calls, emails) can be logged against a contact or deal
- [ ] The activity log shows all entries with timestamps in chronological order
- [ ] The dashboard shows correct total stats: contact count, deal count, pipeline value
- [ ] The dashboard chart renders with real data
- [ ] Email templates can be created, displayed, and copied to clipboard
- [ ] The reports page shows all 4 sections with live data
- [ ] The CSV export downloads a correctly formatted contacts.csv file
- [ ] The CSV import accepts a .csv file, previews rows, and adds contacts to Supabase
- [ ] The app is mobile responsive — tested at 375px viewport width, sidebar collapses
- [ ] Code Review was completed and all critical issues are resolved
- [ ] All 4 Playwright tests passed in the final test run

---

## Level 3 Preview — Client Portal with Lovable

Level 3 is where you stop building tools for yourself and start building products that clients pay for.

You will build a Client Portal — a branded, private web app that businesses give to their clients to track project progress, review and approve deliverables, upload files, and communicate with the team. This is a real product that agencies, consultants, and freelancers charge a monthly recurring fee to offer. The market exists and has paying customers.

The technology stack shifts. You still use Supabase for the database and auth, but the frontend is built with Lovable — an AI-first UI builder that lets you describe interfaces in plain English and get production-quality React components back. Lovable is not a page builder. It writes real code, exports to GitHub, and deploys to Vercel. The difference between Lovable and what you did in Level 2 is speed: what took you 3 days to build in Level 2 takes 2 hours in Lovable, because you are designing at the UI description level rather than the component code level.

You will build role-based access (clients see their own portal, admins see everything), a real-time file upload and review system, a comment thread on each deliverable, and a status board clients can use to track where their project stands. Every feature is a feature a real business needs and will pay for.

On Day 6 of Level 3, you will build a file approval flow where clients can view, comment on, and approve deliverables — the kind of feature agencies charge extra for. You build it in one day using Lovable to design the approval UI, Supabase Storage for the file hosting, and Supabase Realtime to push approval status updates to the agency's dashboard the moment a client clicks Approve.

---

## Commit

```bash
git add .
git commit -m "feat: Level 2 complete — ClearCRM production deploy"
```

---

## Deliverable — Level 2 Graduation Certificate

Copy this certificate, fill in your name and your production URL, and screenshot it.

---

```
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║              VIBE CODE ACADEMY                                       ║
║              Certificate of Completion                               ║
║                                                                      ║
║  This certifies that                                                 ║
║                                                                      ║
║  [ YOUR NAME ]                                                       ║
║                                                                      ║
║  has successfully completed                                          ║
║                                                                      ║
║  LEVEL 2 — ENTERPRISE PROJECT: ClearCRM                             ║
║                                                                      ║
║  and shipped a full-stack CRM application to production,            ║
║  demonstrating proficiency in:                                       ║
║                                                                      ║
║    • Next.js 14 App Router and React Server Components              ║
║    • Supabase Auth, Database, and Row Level Security                ║
║    • Kanban deal pipeline with real-time stage management           ║
║    • Activity logging and contact relationship tracking             ║
║    • Data reporting with pure CSS charts                            ║
║    • CSV import and export for bulk data operations                 ║
║    • Mobile-responsive design with collapsible navigation           ║
║    • End-to-end testing with Playwright                             ║
║                                                                      ║
║  Production URL:                                                     ║
║  [ YOUR VERCEL OR CUSTOM DOMAIN URL ]                               ║
║                                                                      ║
║  Completed: [ DATE ]                                                 ║
║                                                                      ║
║  Issued by Vibe Code Academy                                        ║
║  vibecodeacademy.com                                                ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```
