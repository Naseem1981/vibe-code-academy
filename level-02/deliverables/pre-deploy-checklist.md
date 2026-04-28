# Pre-Deploy Checklist — ClearCRM

Run through all 20 checks before sharing your live URL. Do not skip any item — each one is there because it has burned someone before.

---

- [ ] **1. Local build passes with no errors**
  Run `npm run build` in your terminal. The build must complete with zero TypeScript errors and zero build failures. Warnings are acceptable; errors are not. Fix every error before proceeding.

- [ ] **2. All environment variables are set in Vercel**
  In your Vercel project dashboard go to **Settings → Environment Variables**. Confirm `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` are present and set for the Production environment. If you have any other `.env.local` variables, add those too.

- [ ] **3. Supabase RLS is enabled on all 5 tables**
  In Supabase go to **Table Editor** and check each table (profiles, companies, contacts, deals, activities). The shield icon next to each table name must be green/active. If any table shows RLS as disabled, run `ALTER TABLE [table] ENABLE ROW LEVEL SECURITY;` immediately.

- [ ] **4. Authentication works in production — sign up**
  Visit your live Vercel URL. Click "Sign Up". Create a new account with a real email. Confirm you land on the dashboard or protected page with no redirect loops or errors.

- [ ] **5. Authentication works in production — sign in**
  Sign out from the account you just created. Sign back in using the same credentials. Confirm the session is restored and you can see your data.

- [ ] **6. All 5 CRUD modules work in production**
  Test each module end-to-end on the live URL: create, read, update, and delete at least one record in each of: Contacts, Companies, Deals, Activities, and your own Profile. If any operation silently fails, check the Supabase RLS policies and your Vercel env vars.

- [ ] **7. App is mobile responsive at 375px viewport**
  Open Chrome DevTools, set the viewport to 375px wide (iPhone SE), and navigate through every page. Confirm: no horizontal scroll, all buttons are tappable, text is readable, tables either scroll or stack, modals are fully visible.

- [ ] **8. No console errors in the production browser**
  Open your live Vercel URL in an incognito window. Open DevTools → Console. Navigate through all pages and perform key actions. The console must show zero red errors. Yellow warnings are acceptable.

- [ ] **9. Playwright tests passed for critical flows**
  Run `npx playwright test` locally against your production URL (or localhost). Your test suite should cover: sign in, create a contact, create a deal, move a deal stage, and sign out. All tests must pass green.

- [ ] **10. Code Review run and critical issues resolved**
  Use the Claude Code code-review skill (`/code-review`) or run your linter (`npm run lint`). Any issues marked as critical or error-level must be resolved. Dead code and unused imports should be cleaned up.

- [ ] **11. CSV export downloads correctly**
  Navigate to the CSV Export page in your live app. Click the export button for contacts. Confirm a `.csv` file downloads to your machine. Open it — confirm the column headers are correct and the data matches what is in Supabase.

- [ ] **12. CSV import adds records correctly**
  Prepare a sample CSV file with 3–5 contacts (including all required columns). Use the import feature in your live app. Upload the file. Confirm the records appear in the contacts list with correct data. Check Supabase Table Editor to verify they were inserted.

- [ ] **13. Dashboard stats match Supabase data**
  On the Dashboard page, note the numbers displayed (total contacts, total deals, deal values by stage). Then open Supabase Table Editor and count the rows manually or run a SQL query. The numbers must match exactly.

- [ ] **14. Email templates copy to clipboard correctly**
  Navigate to the Email Templates page. Click "Copy" on at least 2 different templates. Open a text editor and paste — confirm the full template text is there with no truncation or formatting issues.

- [ ] **15. Reporting page shows correct data**
  Visit the Reporting page. Confirm the charts and tables reflect actual data from your Supabase database. Add a test deal if needed to confirm new data appears in the report after a page refresh.

- [ ] **16. All images load correctly**
  Navigate every page that displays images (avatars, logos, placeholder images). Open DevTools → Network tab, filter by "Img". Confirm there are zero 404 responses for image assets.

- [ ] **17. Page load is under 3 seconds**
  Open your live URL in an incognito window. Open DevTools → Network tab. Hard refresh (Ctrl+Shift+R). Confirm the page reaches "Load" in under 3 seconds on a standard connection. If it is slow, check for large unoptimised images or unnecessary client-side data fetching on mount.

- [ ] **18. No exposed API keys visible in browser network tab**
  Open DevTools → Network tab. Reload the page. Click on any Supabase API request. Check the request headers and URL. You should see the `apikey` header with your anon public key — this is expected and safe. You must NOT see your `service_role` key anywhere. Also confirm no other secret keys (e.g. email API keys) are visible in network requests.

- [ ] **19. GitHub repo has the latest commit**
  Run `git status` in your terminal — confirm there are no uncommitted changes. Run `git push origin main`. Check your GitHub repo in the browser and confirm the latest commit matches your local state.

- [ ] **20. Share the live URL with at least one person**
  Copy your Vercel production URL. Send it to one real person — a friend, a mentor, a potential client, or post it in the Vibe Code Academy community. You built a real product. Let someone see it. This step is not optional.

---

> **All 20 boxes checked? You are cleared for launch. Congratulations — you shipped ClearCRM.**
