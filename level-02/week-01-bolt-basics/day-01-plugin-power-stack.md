# Day 1: Your Plugin Power Stack

## Outcome artifact
By the end of this day you will have all 10 Claude Code plugins installed and active in your workspace, and you will know exactly what each one does and when to use it during a build session.

---

## Why plugins change everything

Out of the box, Claude Code is already powerful. With plugins, it becomes a full development team.

Plugins are extensions that give Claude Code new abilities — the ability to read live web pages, test your app in a browser, scan your code for security holes, review what you just built, and inject up-to-date documentation directly into its working context. Without them, Claude Code is working from memory. With them, it has live access to the tools and information it needs to build production-quality apps.

You install all 10 plugins once. They stay active across every project you build from here on.

---

## The 10 plugins — what each one does

### 1. Context7
**Install:** `/plugin install context7@claude-plugins-official`

**What it does:** Every library and framework — React, Supabase, Stripe, Tailwind — releases updates constantly. Claude's training data has a cutoff date, which means it sometimes suggests outdated methods or APIs that no longer exist. Context7 fixes this by injecting the *current* documentation for whatever library you are using directly into Claude's working context before it writes any code.

**In practice:** You tell Claude "add Supabase auth to this app" and Context7 automatically pulls the latest Supabase auth docs, so Claude writes code that matches the version you actually installed — not a version from 18 months ago.

**When you will notice it:** Any time Claude writes an import or API call that matches exactly what the library's current docs say. Without Context7, you get deprecation warnings and broken installs. With it, you don't.

---

### 2. Frontend Design
**Install:** `/plugin install frontend-design@claude-plugins-official`

**What it does:** Left to its defaults, Claude Code produces functional but generic-looking interfaces — default Tailwind blues, flat cards, standard button shapes. Frontend Design overrides this behaviour by applying stronger design judgment: better colour pairing, proper typography scale, intentional spacing, layered shadows, and visual hierarchy.

**In practice:** You prompt "build a dashboard" and instead of a plain white page with blue buttons, you get a design with a proper colour palette, readable font pairing, and depth. It won't win design awards but it won't embarrass you either.

**When you will notice it:** Immediately. The first component Claude generates with this plugin active looks noticeably more considered than without it.

---

### 3. Security Guidance
**Install:** `/plugin install security-guidance@claude-plugins-official`

**What it does:** Before Claude Code writes any file edit to disk, Security Guidance scans the change for common vulnerabilities — SQL injection, exposed API keys, missing input validation, insecure authentication patterns, and other OWASP top-10 issues. If it finds a problem, it flags it before the code lands in your file.

**In practice:** You ask Claude to build a login form. Before writing the file, Security Guidance checks whether the form hashes passwords, validates input, and prevents injection attacks. If something is wrong, Claude is told to fix it first.

**When you will notice it:** When Claude pauses and rewrites something without you asking. That rewrite is the plugin doing its job. You may also see a note in the output like "Security Guidance flagged missing input sanitisation — fixed before writing."

---

### 4. Code Review
**Install:** `/plugin install code-review@claude-plugins-official`

**What it does:** Deploys multiple AI agents to review your code from different angles — one checks logic, one checks performance, one checks readability, one checks for bugs. Each agent reports a confidence score. The result is a structured review report that surfaces problems a single reviewer would miss.

**In practice:** You finish a feature and run `/review`. Three agents analyse the same code independently and consolidate their findings. You get a list of issues ranked by severity before you push to GitHub.

**When you will notice it:** When a review catches something you would have shipped — a missing null check, an unhandled promise rejection, a function that works but breaks under edge-case input.

---

### 5. Chrome DevTools MCP
**Install:** `/plugin install chrome-devtools-mcp@chrome-devtools-plugins`

**What it does:** Connects Claude Code directly to a live Chrome browser session. Claude can open your app in Chrome, read the console errors, inspect the DOM, measure performance, and observe network requests — all without you copying and pasting error messages.

**In practice:** Your app throws a runtime error. Instead of copying the console trace and pasting it into Claude, Claude opens Chrome, sees the error directly, traces it to the source file, and fixes it — then reloads the browser to confirm the fix worked.

**When you will notice it:** When debugging sessions that used to take 20 minutes of copy-paste take 2 minutes because Claude is reading the browser directly.

---

### 6. Playwright
**Install:** `/plugin install playwright@claude-plugins-official`

**What it does:** Opens a controllable Chrome window that Claude can drive with natural language commands. Claude can click buttons, fill forms, navigate pages, take screenshots, and verify that your app behaves correctly — all as part of a build session.

**In practice:** You ask Claude to "test the signup flow." Claude opens your app in Playwright's browser, clicks through the signup form, submits it, checks that the user lands on the dashboard, and reports back whether it worked. If it didn't, Claude sees the failure and fixes it.

**When you will notice it:** When you stop manually clicking through your app to check if something works. Claude does it for you.

---

### 7. Firecrawl
**Install:** `/plugin install firecrawl@claude-plugins-official`

**What it does:** Gives Claude Code the ability to scrape and search the live web. Claude can fetch any URL, extract structured data from web pages, search for real-time information, and use that content as context when building features.

**In practice:** You are building an app that displays property listings. Instead of making up sample data, you ask Claude to "scrape 10 property listings from [a public listings site] and use that as the seed data." Firecrawl fetches the real data and Claude builds the UI around actual content.

**When you will notice it:** Any time you need real-world data, current information, or content from a website built into your app.

---

### 8. Figma MCP
**Install:** `/plugin install figma@claude-plugins-official`

**What it does:** Connects Claude Code to your Figma account. Claude can read a Figma design file directly — frames, components, colours, typography, spacing — and translate it into functional component code without you manually describing the design.

**In practice:** Your designer sends you a Figma link. Instead of spending hours translating measurements and colours, you paste the Figma URL into Claude and say "build this screen." Claude reads the Figma file and generates the component with the correct layout, colours, and font sizes.

**When you will notice it:** The first time you go from Figma to working code in under 5 minutes instead of 2 hours.

---

### 9. Ralph Loop
**Install:** `/plugin install ralph-loop@claude-plugins-official`

**What it does:** Enables autonomous, multi-hour coding sessions. You give Claude a task list and Ralph Loop manages the execution — running tasks one by one, checking results, handling errors, and continuing without requiring you to approve every step. It is designed for sessions where you want Claude to build independently while you step away.

**In practice:** You give Claude a list of 12 tasks to complete on your CRM app. Instead of confirming each one, you activate Ralph Loop and come back an hour later to find 10 of the 12 done, with a report on what happened and what needs your input.

**When you will notice it:** When you return from lunch to a significant amount of completed work instead of Claude sitting idle waiting for your next message.

---

### 10. Linear
**Install:** `/plugin install linear@claude-plugins-official`

**What it does:** Connects Claude Code to Linear, a professional issue tracking and project management tool used by engineering teams. Claude can create issues, update their status, add comments, and link commits to tickets — all from inside a build session.

**In practice:** You find a bug while building. You tell Claude "log this as a bug in Linear." Claude creates the ticket with the right title, description, and priority without you switching apps. When you fix it, Claude marks it done.

**When you will notice it:** When your issue tracker stays up to date automatically instead of becoming a graveyard of forgotten tickets.

---

## Step-by-step: Install all 10

Open your terminal, navigate to your project folder, and run Claude Code:

```bash
cd your-project-folder
claude
```

Once Claude Code is running, paste each install command one at a time and wait for confirmation before running the next:

```
/plugin install context7@claude-plugins-official
```
Wait for: `✓ Plugin context7 installed`

```
/plugin install frontend-design@claude-plugins-official
```
Wait for: `✓ Plugin frontend-design installed`

```
/plugin install security-guidance@claude-plugins-official
```
Wait for: `✓ Plugin security-guidance installed`

```
/plugin install code-review@claude-plugins-official
```
Wait for: `✓ Plugin code-review installed`

```
/plugin install chrome-devtools-mcp@chrome-devtools-plugins
```
Wait for: `✓ Plugin chrome-devtools-mcp installed`

```
/plugin install playwright@claude-plugins-official
```
Wait for: `✓ Plugin playwright installed`

```
/plugin install firecrawl@claude-plugins-official
```
Wait for: `✓ Plugin firecrawl installed`

```
/plugin install figma@claude-plugins-official
```
Wait for: `✓ Plugin figma installed`

```
/plugin install ralph-loop@claude-plugins-official
```
Wait for: `✓ Plugin ralph-loop installed`

```
/plugin install linear@claude-plugins-official
```
Wait for: `✓ Plugin linear installed`

---

## Verify all 10 are active

Run this command to see your installed plugins:

```
/plugins
```

Expected output — you should see all 10 listed:

```
Installed plugins:
  ✓ context7
  ✓ frontend-design
  ✓ security-guidance
  ✓ code-review
  ✓ chrome-devtools-mcp
  ✓ playwright
  ✓ firecrawl
  ✓ figma
  ✓ ralph-loop
  ✓ linear
```

If any plugin is missing from the list, re-run its install command.

---

## Practical workflow

1. Open terminal → navigate to project folder
2. Run `claude` to start Claude Code
3. Run `/plugin install context7@claude-plugins-official` → wait for confirmation
4. Run `/plugin install frontend-design@claude-plugins-official` → wait for confirmation
5. Run `/plugin install security-guidance@claude-plugins-official` → wait for confirmation
6. Run `/plugin install code-review@claude-plugins-official` → wait for confirmation
7. Run `/plugin install chrome-devtools-mcp@chrome-devtools-plugins` → wait for confirmation
8. Run `/plugin install playwright@claude-plugins-official` → wait for confirmation
9. Run `/plugin install firecrawl@claude-plugins-official` → wait for confirmation
10. Run `/plugin install figma@claude-plugins-official` → wait for confirmation
11. Run `/plugin install ralph-loop@claude-plugins-official` → wait for confirmation
12. Run `/plugin install linear@claude-plugins-official` → wait for confirmation
13. Run `/plugins` → confirm all 10 appear in the list

---

## Common mistakes

**Installing without waiting for confirmation.** Running the next command before the previous one confirms causes install conflicts. Run one, see the `✓` confirmation, then run the next.

**Expecting to see plugins do something visible on install.** Plugins activate silently. You will not see a pop-up or banner. The way you know they are working is by watching Claude's output during a build — Context7 references current docs, Security Guidance rewrites unsafe code, Frontend Design produces better components.

**Thinking you need to reinstall every session.** Plugins are installed once and persist across all future sessions. You never need to run these install commands again unless you start on a new machine.

---

## Your turn

Run all 10 install commands, then run `/plugins` and confirm all 10 appear in the list.

You should see:
```
Installed plugins:
  ✓ context7
  ✓ frontend-design
  ✓ security-guidance
  ✓ code-review
  ✓ chrome-devtools-mcp
  ✓ playwright
  ✓ firecrawl
  ✓ figma
  ✓ ralph-loop
  ✓ linear
```

If you see fewer than 10, check which one is missing and re-run its install command. If you see an error message during install, copy the exact error text and prompt Claude: "I got this error installing a plugin: [paste error]. How do I fix it?"

---

## Plugin quick-reference card

| Plugin | What it gives Claude | Use it when |
|--------|---------------------|-------------|
| Context7 | Current library docs | Always — active automatically |
| Frontend Design | Better UI judgment | Always — active automatically |
| Security Guidance | Vulnerability scanning | Always — active automatically |
| Code Review | Multi-agent code review | Before every GitHub push |
| Chrome DevTools MCP | Live browser access | Debugging runtime errors |
| Playwright | Browser automation | Testing user flows |
| Firecrawl | Live web data | Scraping, search, real content |
| Figma MCP | Design file reading | Translating Figma to code |
| Ralph Loop | Autonomous build sessions | Long task lists, step away builds |
| Linear | Issue tracking | Logging bugs and tracking features |
