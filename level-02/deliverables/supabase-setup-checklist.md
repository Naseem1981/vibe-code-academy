# Supabase Setup Checklist — ClearCRM

Complete these 20 steps in order before writing any application code. Each step is a dependency for the next.

---

- [ ] **1. Create a Supabase account**
  Go to [supabase.com](https://supabase.com) and sign up with GitHub or email. Free tier is sufficient for the entire course.

- [ ] **2. Create a new Supabase project**
  Click "New Project". Name it `clearcrm`. Choose a strong database password and save it somewhere safe. Select the region closest to you. Wait for provisioning (takes 1–2 minutes).

- [ ] **3. Copy your Project URL**
  In the Supabase dashboard, go to **Settings → API**. Copy the **Project URL** (looks like `https://xyzxyz.supabase.co`). Paste it into a scratch file — you will use it in multiple places.

- [ ] **4. Copy your anon/public API key**
  On the same **Settings → API** page, copy the **anon public** key. This is safe to use in your frontend. Do not use the `service_role` key in client-side code.

- [ ] **5. Open the Supabase SQL Editor**
  In the left sidebar, click **SQL Editor** → **New query**. This is where you will run all schema setup SQL.

- [ ] **6. Run the CREATE TABLE statements**
  Paste and run the full CREATE TABLE block from `deliverables/clearcrm-schema.md` (profiles, companies, contacts, deals, activities). Confirm no errors appear in the output panel.

- [ ] **7. Run the updated_at trigger function**
  Paste and run the `handle_updated_at()` function and all 5 trigger CREATE statements from the schema file.

- [ ] **8. Enable Row Level Security on all 5 tables**
  Run the 5 `ALTER TABLE ... ENABLE ROW LEVEL SECURITY;` statements from the schema file. Verify in **Table Editor → [table name] → RLS** that the shield icon shows green.

- [ ] **9. Create RLS policies for all 5 tables**
  Run the full CREATE POLICY block from the schema file. Each table should have 4 policies (SELECT, INSERT, UPDATE, DELETE). Verify in **Authentication → Policies** that you see 4 policies per table.

- [ ] **10. Enable email/password authentication**
  Go to **Authentication → Providers → Email**. Make sure the Email provider toggle is **enabled**. Confirm "Enable Email Signup" is on.

- [ ] **11. Disable email confirmation for development**
  Still in **Authentication → Providers → Email**, turn off **"Confirm email"**. This lets you sign up and log in immediately without checking your inbox during development. Re-enable this before going to production.

- [ ] **12. Set the site URL for auth redirects**
  Go to **Authentication → URL Configuration**. Set **Site URL** to `http://localhost:3000` for development. You will update this to your Vercel URL after deployment.

- [ ] **13. Add redirect URLs**
  In the same URL Configuration section, add `http://localhost:3000/**` to the **Redirect URLs** list. Also add your future Vercel URL (e.g. `https://clearcrm.vercel.app/**`) even if it does not exist yet.

- [ ] **14. Create your `.env.local` file**
  In the root of your Next.js project, create a file named `.env.local` with these values:

  ```
  NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
  NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
  ```

  Replace the placeholder values with the URL and key you copied in steps 3 and 4.

- [ ] **15. Add `.env.local` to `.gitignore`**
  Open `.gitignore` and confirm `.env.local` is listed. Never commit this file. Your Supabase keys should never appear in your git history.

- [ ] **16. Install the Supabase JS client**
  In your project terminal, run:

  ```bash
  npm install @supabase/supabase-js @supabase/ssr
  ```

- [ ] **17. Test the database connection from your app**
  Create a simple test in your app — for example, fetch from the `contacts` table and log the result. If you get an empty array (not an error), the connection is working. If you get an RLS error, check that you are passing the auth session correctly.

- [ ] **18. Add environment variables to Vercel**
  In your Vercel project dashboard, go to **Settings → Environment Variables**. Add `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` with the same values as your `.env.local`. Set them for all environments (Production, Preview, Development).

- [ ] **19. Update the Supabase Site URL for production**
  After your first Vercel deployment, go back to **Supabase → Authentication → URL Configuration** and update **Site URL** to your live Vercel URL (e.g. `https://clearcrm-your-username.vercel.app`). Also add it to the Redirect URLs list.

- [ ] **20. Test a full sign-up and sign-in flow in production**
  Visit your live Vercel URL. Create a new account. Sign out. Sign back in. Confirm the session persists and you can access protected routes. If auth fails in production but works locally, check that your Vercel environment variables are correct and the Supabase site URL is updated.

---

> **Once all 20 boxes are checked, your Supabase project is fully configured and ready for the CRM build.**
