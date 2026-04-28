# Day 6: Project Setup — ClearCRM Tech Stack from Zero to Live

## Outcome artifact
By the end of this day you will have ClearCRM live at a Vercel URL with a Next.js 14 App Router frontend, a Supabase project with all five database tables created, and environment variables wired up so the app connects to your database.

## The core idea

You are assembling four services that work together: Bolt.new generates your starting codebase in minutes, Supabase gives you a hosted PostgreSQL database and authentication backend, GitHub stores and versions your code, and Vercel hosts the deployed app. Each service has one job. None of them requires a credit card to start.

The reason you scaffold with Bolt.new first — before touching your local machine — is speed. Bolt generates a working Next.js 14 project with the correct file structure, Tailwind configured, and Supabase client stubs already in place. You then own that code entirely. You download it, version it, and extend it yourself. Bolt is the starting gun, not the race.

Supabase runs PostgreSQL under the hood. You interact with it through the Supabase dashboard SQL editor, the JavaScript client library, and Row Level Security (RLS) policies. RLS is a PostgreSQL feature that enforces rules at the database level — even if someone gets hold of your anon key, they can only read data the policy allows. You will enable RLS on every table today even before you add complex policies, because the habit of enabling it immediately prevents accidental data exposure.

## Step-by-step walkthrough

### Part 1 — Scaffold with Bolt.new

**Step 1.** Open your browser and go to `https://bolt.new`.

**Step 2.** In the chat prompt box, paste this prompt exactly:

```
Create a Next.js 14 app called ClearCRM. Use the App Router. Include Tailwind CSS. Add Supabase client setup with environment variables for NEXT_PUBLIC_SUPABASE_URL and NEXT_PUBLIC_SUPABASE_ANON_KEY. Add a basic layout with a dark sidebar (#0F172A) on the left and white main area on the right. The sidebar should have placeholder nav links for Dashboard, Contacts, Companies, Deals, Activities. Export the project.
```

**Step 3.** Wait for Bolt to generate the project. This takes 30–60 seconds. You will see files appear in the file tree on the left side of the Bolt interface.

**Step 4.** Click the **Download** button (top-right area of the Bolt interface). Bolt exports a ZIP file named something like `project.zip` or `bolt-export.zip`.

**Step 5.** Unzip the file into a folder named `clearcrm`. On Mac: double-click the ZIP. On Windows: right-click → Extract All. The unzipped folder should contain `package.json`, `app/`, `tailwind.config.js`, and similar Next.js files.

---

### Part 2 — Set up Supabase

**Step 1.** Go to `https://supabase.com`. Click **Start your project**. Sign up with GitHub or email.

**Step 2.** Click **New project**. Fill in:
- Name: `clearcrm`
- Database Password: choose something strong and save it in a password manager — you cannot recover this later
- Region: choose the region closest to your users (e.g. US East, EU West)

Click **Create new project**. Wait approximately 2 minutes for the project to provision. The dashboard will show a loading indicator.

**Step 3.** When the project is ready, go to **Settings → API** in the left sidebar. You will see two values you need:
- **Project URL** — looks like `https://xyzxyzxyz.supabase.co`
- **anon / public key** — a long JWT string starting with `eyJ`

Copy both. Keep them open in a tab.

**Step 4.** In the left sidebar, click **SQL Editor**. Click **New query**. Paste the following SQL and click **Run**:

```sql
-- Companies must be created before contacts (contacts references companies)
CREATE TABLE companies (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  website TEXT,
  industry TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE contacts (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  first_name TEXT NOT NULL,
  last_name TEXT NOT NULL,
  email TEXT,
  phone TEXT,
  company_id UUID REFERENCES companies(id) ON DELETE SET NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE deals (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  title TEXT NOT NULL,
  value NUMERIC(12, 2),
  stage TEXT CHECK (stage IN ('lead','qualified','proposal','negotiation','won','lost')) DEFAULT 'lead',
  contact_id UUID REFERENCES contacts(id) ON DELETE SET NULL,
  close_date DATE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE activities (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  type TEXT CHECK (type IN ('note','call','email')) NOT NULL,
  description TEXT,
  contact_id UUID REFERENCES contacts(id) ON DELETE SET NULL,
  deal_id UUID REFERENCES deals(id) ON DELETE SET NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

CREATE TABLE email_templates (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  subject TEXT NOT NULL,
  body TEXT NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);
```

Expected result: The Supabase SQL editor shows `Success. No rows returned` for each statement.

**Step 5.** In the SQL Editor, open a new query. Enable Row Level Security on all tables:

```sql
ALTER TABLE companies ENABLE ROW LEVEL SECURITY;
ALTER TABLE contacts ENABLE ROW LEVEL SECURITY;
ALTER TABLE deals ENABLE ROW LEVEL SECURITY;
ALTER TABLE activities ENABLE ROW LEVEL SECURITY;
ALTER TABLE email_templates ENABLE ROW LEVEL SECURITY;
```

**Step 6.** Add basic RLS policies. For now, allow authenticated users to read and write their own data. Paste this in a new query:

```sql
-- Contacts: authenticated users can select, insert, update, delete
CREATE POLICY "Authenticated users can read contacts"
  ON contacts FOR SELECT
  TO authenticated
  USING (true);

CREATE POLICY "Authenticated users can insert contacts"
  ON contacts FOR INSERT
  TO authenticated
  WITH CHECK (true);

CREATE POLICY "Authenticated users can update contacts"
  ON contacts FOR UPDATE
  TO authenticated
  USING (true);

CREATE POLICY "Authenticated users can delete contacts"
  ON contacts FOR DELETE
  TO authenticated
  USING (true);

-- Repeat the same pattern for companies
CREATE POLICY "Authenticated users can read companies"
  ON companies FOR SELECT TO authenticated USING (true);

CREATE POLICY "Authenticated users can insert companies"
  ON companies FOR INSERT TO authenticated WITH CHECK (true);

CREATE POLICY "Authenticated users can update companies"
  ON companies FOR UPDATE TO authenticated USING (true);

CREATE POLICY "Authenticated users can delete companies"
  ON companies FOR DELETE TO authenticated USING (true);

-- Deals
CREATE POLICY "Authenticated users can read deals"
  ON deals FOR SELECT TO authenticated USING (true);

CREATE POLICY "Authenticated users can insert deals"
  ON deals FOR INSERT TO authenticated WITH CHECK (true);

CREATE POLICY "Authenticated users can update deals"
  ON deals FOR UPDATE TO authenticated USING (true);

CREATE POLICY "Authenticated users can delete deals"
  ON deals FOR DELETE TO authenticated USING (true);

-- Activities
CREATE POLICY "Authenticated users can read activities"
  ON activities FOR SELECT TO authenticated USING (true);

CREATE POLICY "Authenticated users can insert activities"
  ON activities FOR INSERT TO authenticated WITH CHECK (true);

CREATE POLICY "Authenticated users can update activities"
  ON activities FOR UPDATE TO authenticated USING (true);

CREATE POLICY "Authenticated users can delete activities"
  ON activities FOR DELETE TO authenticated USING (true);

-- Email templates
CREATE POLICY "Authenticated users can read email_templates"
  ON email_templates FOR SELECT TO authenticated USING (true);

CREATE POLICY "Authenticated users can insert email_templates"
  ON email_templates FOR INSERT TO authenticated WITH CHECK (true);

CREATE POLICY "Authenticated users can update email_templates"
  ON email_templates FOR UPDATE TO authenticated USING (true);

CREATE POLICY "Authenticated users can delete email_templates"
  ON email_templates FOR DELETE TO authenticated USING (true);
```

---

### Part 3 — Local setup

**Step 1.** Open VS Code. Go to **File → Open Folder** and select the `clearcrm` folder you unzipped.

**Step 2.** In VS Code, open the **Explorer** panel (left sidebar, first icon). Right-click in the file list below the existing files and click **New File**. Name it `.env.local`.

Paste this content into `.env.local`, replacing the placeholder values with your actual Supabase values from Part 2 Step 3:

```
NEXT_PUBLIC_SUPABASE_URL=https://your-project-ref.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJyour-anon-key-here
```

**Step 3.** Open the VS Code integrated terminal: **Terminal → New Terminal** (or press `` Ctrl+` ``). Make sure the terminal is in the `clearcrm` directory. Run:

```
npm install
```

Directory: `clearcrm/`
Expected output:
```
added 312 packages, and audited 313 packages in 18s
found 0 vulnerabilities
```

The exact number of packages will vary. The important part is no errors.

**Step 4.** Start the development server:

```
npm run dev
```

Directory: `clearcrm/`
Expected output:
```
▲ Next.js 14.x.x
- Local:        http://localhost:3000
- Environments: .env.local
ready started server on 0.0.0.0:3000, url: http://localhost:3000
```

**Step 5.** Open your browser and go to `http://localhost:3000`. You should see the ClearCRM layout: a dark sidebar on the left with nav links, and a white main area on the right. If Bolt generated a placeholder heading or dashboard content in the main area, that is expected.

---

### Part 4 — GitHub and Vercel

**Step 1.** In the VS Code terminal, press `Ctrl+C` to stop the dev server. Initialize a git repository and make the first commit:

```
git init && git add . && git commit -m "feat: initial ClearCRM scaffold"
```

Directory: `clearcrm/`
Expected output ends with:
```
[main (root-commit) a1b2c3d] feat: initial ClearCRM scaffold
 24 files changed, 1204 insertions(+)
```

The hash and file count will differ on your machine.

**Step 2.** Create a GitHub repository. If you have the GitHub CLI installed:

```
gh repo create clearcrm --public --source=. --remote=origin --push
```

Directory: `clearcrm/`
Expected output:
```
✓ Created repository yourusername/clearcrm on GitHub
✓ Added remote https://github.com/yourusername/clearcrm.git
✓ Pushed commits to https://github.com/yourusername/clearcrm.git
```

If you do not have the GitHub CLI, do it manually:
1. Go to `https://github.com/new`
2. Repository name: `clearcrm`
3. Set to Public (or Private — your choice)
4. Do NOT initialize with a README (your local code already has files)
5. Click **Create repository**
6. GitHub shows you the push commands. In your terminal, run:

```
git remote add origin https://github.com/yourusername/clearcrm.git
git branch -M main
git push -u origin main
```

**Step 3.** Go to `https://vercel.com`. Sign in with your GitHub account.

**Step 4.** Click **Add New → Project**. Vercel shows a list of your GitHub repositories. Click **Import** next to `clearcrm`.

**Step 5.** On the configuration screen, before clicking Deploy, scroll down to **Environment Variables**. Add these two variables:

| Name | Value |
|------|-------|
| `NEXT_PUBLIC_SUPABASE_URL` | Your Supabase Project URL |
| `NEXT_PUBLIC_SUPABASE_ANON_KEY` | Your Supabase anon/public key |

Click **Add** after each one.

**Step 6.** Click **Deploy**. Vercel clones your repo, runs `npm run build`, and deploys. This takes 1–3 minutes. The build log streams in real time — watch for any red error lines.

**Step 7.** When the build succeeds, Vercel shows a success screen with confetti and your live URL. It looks like `https://clearcrm-yourusername.vercel.app`. Click **Visit** to open it. You should see the same sidebar layout you saw at `localhost:3000`.

Copy this URL. You will use it throughout the course.

---

## Practical workflow

1. Go to bolt.new, paste the scaffold prompt, download the ZIP
2. Unzip into folder named `clearcrm`
3. Create Supabase project at supabase.com
4. Copy Project URL and anon key from Settings → API
5. Run the 5-table CREATE TABLE SQL in the Supabase SQL Editor
6. Enable RLS on all 5 tables
7. Add RLS policies for authenticated users on all 5 tables
8. Open `clearcrm` folder in VS Code
9. Create `.env.local` with the two Supabase env vars
10. Run `npm install` in the terminal
11. Run `npm run dev` and verify `http://localhost:3000` loads
12. Run `git init && git add . && git commit -m "feat: initial ClearCRM scaffold"`
13. Create GitHub repo and push
14. Import repo to Vercel, add env vars, deploy
15. Copy the live Vercel URL

---

## Common mistakes

**1. Adding NEXT_PUBLIC_ env vars to Vercel but the app still can't connect to Supabase**

The Vercel environment variables screen has three environment tabs: Production, Preview, and Development. If you only add vars to Production, preview deployments cannot connect. Add the vars to all three tabs. After adding, redeploy — Vercel does not automatically redeploy when you add env vars.

**2. The Supabase tables exist but all queries return empty arrays or "row level security violation" errors**

You enabled RLS but did not add the policies. RLS with no policies = no access for anyone. Go back to the Supabase SQL Editor and run the RLS policy SQL from Part 2 Step 6. Verify in **Authentication → Policies** that each table shows your policies listed.

**3. `npm run dev` throws "Module not found: Can't resolve '@supabase/supabase-js'"**

Bolt generated the import but the package is not installed yet, or the download did not include `node_modules` (it should not — `node_modules` is always excluded). Run `npm install` first. If the error persists, run `npm install @supabase/supabase-js` explicitly to add the package.

---

## Your turn

Start your terminal. Navigate to the `clearcrm` folder. Run `npm run dev`. Open `http://localhost:3000` in your browser.

Expected output: A dark sidebar on the left side of the screen with five nav links (Dashboard, Contacts, Companies, Deals, Activities). A white content area on the right. No 404 error. No red console errors.

**Failure state:** The page loads but shows a blank white screen with no sidebar.
Fix: Open the browser console (F12 → Console tab). Look for a JavaScript error. The most common cause is a missing or malformed `app/layout.tsx` — Bolt may have generated a root layout without the sidebar wrapper. Open `app/layout.tsx` in VS Code and verify it renders a `<div>` that contains both the sidebar component and a `<main>` element that wraps `{children}`.

**Failure state:** Terminal shows `Error: NEXT_PUBLIC_SUPABASE_URL is not defined`.
Fix: Make sure your `.env.local` file is in the `clearcrm/` root folder (same level as `package.json`), not inside the `app/` folder. Restart the dev server after creating or editing `.env.local`.

---

## Prompt / Template / Checklist pack

### ClearCRM Project Setup Checklist

Use this checklist to verify your setup before moving to Day 7.

**Bolt.new scaffold**
- [ ] Visited bolt.new and pasted the scaffold prompt
- [ ] Downloaded the project ZIP
- [ ] Unzipped into a folder named `clearcrm`
- [ ] Verified `package.json` exists in the `clearcrm/` root

**Supabase project**
- [ ] Created Supabase account
- [ ] Created new project named `clearcrm`
- [ ] Saved the database password in a safe place
- [ ] Copied the Project URL from Settings → API
- [ ] Copied the anon/public key from Settings → API
- [ ] Ran the 5-table CREATE TABLE SQL with no errors
- [ ] Ran ALTER TABLE ... ENABLE ROW LEVEL SECURITY on all 5 tables
- [ ] Added RLS policies for authenticated users on all 5 tables
- [ ] Verified tables appear in the Supabase Table Editor (left sidebar → Table Editor)

**Local development**
- [ ] Opened `clearcrm/` in VS Code
- [ ] Created `.env.local` with NEXT_PUBLIC_SUPABASE_URL
- [ ] Created `.env.local` with NEXT_PUBLIC_SUPABASE_ANON_KEY
- [ ] `.env.local` is in the project root (same level as `package.json`)
- [ ] Ran `npm install` with no errors
- [ ] Ran `npm run dev` — server starts on port 3000
- [ ] Opened `http://localhost:3000` — sidebar layout visible

**GitHub**
- [ ] Ran `git init`
- [ ] Ran `git add .`
- [ ] Ran `git commit -m "feat: initial ClearCRM scaffold"`
- [ ] Created GitHub repo named `clearcrm`
- [ ] Pushed code to GitHub — repo shows files on github.com

**Vercel deployment**
- [ ] Signed into vercel.com with GitHub
- [ ] Imported the `clearcrm` repo
- [ ] Added NEXT_PUBLIC_SUPABASE_URL to Vercel environment variables (all environments)
- [ ] Added NEXT_PUBLIC_SUPABASE_ANON_KEY to Vercel environment variables (all environments)
- [ ] Clicked Deploy — build succeeded with no errors
- [ ] Copied the live Vercel URL
- [ ] Opened the live URL — sidebar layout visible (same as localhost)

**Final verification**
- [ ] Live Vercel URL loads the app
- [ ] No red errors in the Vercel build log
- [ ] No console errors in the browser on the live URL
