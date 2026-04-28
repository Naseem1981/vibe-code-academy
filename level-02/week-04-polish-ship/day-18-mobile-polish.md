# Day 18: Mobile Polish

## Outcome artifact

ClearCRM looks and works correctly on a 375px mobile screen. The sidebar collapses to a hamburger menu that slides in from the left. All data tables reformat as stacked card lists on screens smaller than `md` (768px). All buttons, inputs, and interactive controls have a minimum touch target height of 44px. You have verified the layout using Chrome DevTools MCP and tested on an actual mobile device using ngrok.

---

## The core idea

You built ClearCRM desktop-first. That is normal. But "works on desktop" and "usable on mobile" are two different things. A CRM that sales reps cannot open on their phone while walking into a meeting is not a real tool.

Mobile polish has three layers:

1. **Navigation** — the sidebar takes up 240px on desktop. On a 375px screen, that leaves 135px for content. The sidebar must collapse to a hamburger menu.
2. **Tables** — `<table>` elements overflow on small screens. A contacts table with 6 columns does not fit at 375px. You replace it with a card list on mobile.
3. **Touch targets** — Apple's Human Interface Guidelines and Google's Material Design both specify a minimum 44×44px touch target for any interactive control. Tailwind: `min-h-[44px]`.

You use Chrome DevTools MCP to audit the layout and catch overflow issues you might miss visually. Then you expose the app on your local network with ngrok so you can open it on your actual phone.

---

## Step-by-step walkthrough

### Step 1 — Add state and hamburger button to the top bar

Your layout file manages the sidebar open/closed state. Because this is a state toggle, it must be a Client Component.

Open your root layout or main layout component. If your `app/layout.tsx` is a Server Component (no `'use client'`), create a separate `components/AppShell.tsx` wrapper that is a Client Component:

```bash
touch components/AppShell.tsx
```

```tsx
// components/AppShell.tsx
'use client'

import { useState } from 'react'
import Sidebar from './Sidebar'

export default function AppShell({ children }: { children: React.ReactNode }) {
  const [sidebarOpen, setSidebarOpen] = useState(false)

  return (
    <div className="flex h-screen bg-gray-50 overflow-hidden">
      {/* Mobile overlay — shown when sidebar is open */}
      {sidebarOpen && (
        <div
          className="fixed inset-0 z-20 bg-black/40 backdrop-blur-sm lg:hidden"
          onClick={() => setSidebarOpen(false)}
          aria-hidden="true"
        />
      )}

      {/* Sidebar */}
      <div
        className={`
          fixed inset-y-0 left-0 z-30 w-60 bg-[#0F172A] transform transition-transform duration-300 ease-in-out
          lg:relative lg:translate-x-0 lg:block
          ${sidebarOpen ? 'translate-x-0' : '-translate-x-full'}
        `}
      >
        <Sidebar onClose={() => setSidebarOpen(false)} />
      </div>

      {/* Main content */}
      <div className="flex flex-col flex-1 min-w-0 overflow-hidden">
        {/* Top bar — mobile only */}
        <div className="flex items-center gap-4 h-14 px-4 bg-white border-b border-gray-200 lg:hidden">
          <button
            onClick={() => setSidebarOpen(true)}
            className="p-2 rounded-lg text-gray-500 hover:text-gray-900 hover:bg-gray-100 transition-colors min-h-[44px] min-w-[44px] flex items-center justify-center"
            aria-label="Open menu"
          >
            <svg className="w-5 h-5" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M4 6h16M4 12h16M4 18h16" />
            </svg>
          </button>
          <span className="text-sm font-semibold text-gray-800">ClearCRM</span>
        </div>

        {/* Page content */}
        <main className="flex-1 overflow-y-auto p-4 lg:p-6">
          {children}
        </main>
      </div>
    </div>
  )
}
```

Now update `app/layout.tsx` to use `AppShell`:

```tsx
// app/layout.tsx
import AppShell from '@/components/AppShell'

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="en">
      <body>
        <AppShell>{children}</AppShell>
      </body>
    </html>
  )
}
```

---

### Step 2 — Update the Sidebar component

The sidebar now receives an `onClose` prop — a function to call when the user clicks a navigation link (which closes the sidebar on mobile).

Open `components/Sidebar.tsx` and add the prop:

```tsx
// components/Sidebar.tsx
type SidebarProps = {
  onClose?: () => void
}

export default function Sidebar({ onClose }: SidebarProps) {
  // Wrap each nav link's onClick to also close the sidebar on mobile
  // Example for the Contacts link:
  return (
    <nav className="flex flex-col h-full py-6 px-3 space-y-1">
      <div className="px-3 mb-6">
        {/* Logo / brand */}
        <span className="text-white font-bold text-lg tracking-tight">ClearCRM</span>
      </div>

      <Link
        href="/contacts"
        onClick={onClose}
        className="flex items-center gap-2 px-3 py-2 text-sm text-gray-300 hover:text-white hover:bg-white/10 rounded-lg transition-colors min-h-[44px]"
      >
        Contacts
      </Link>

      {/* Repeat for all nav links — add onClick={onClose} to each */}
    </nav>
  )
}
```

The `onClick={onClose}` on every link means: when the user taps a link on mobile, the sidebar closes and the page navigates. Without this, the sidebar stays open after navigation and covers the content.

---

### Step 3 — How the slide animation works

The sidebar uses CSS transforms:

- `translate-x-0` — sidebar is visible (on screen)
- `-translate-x-full` — sidebar is off screen to the left (hidden)

The `transition-transform duration-300 ease-in-out` class animates between the two states. The animation only touches `transform` — not `left`, not `margin`, not `width`. This keeps the animation on the GPU compositor thread, which means it stays smooth at 60fps even on mid-range phones.

On `lg:` breakpoint (1024px and wider), the sidebar is always visible:

- `lg:relative` — takes it out of `fixed` positioning so it participates in the flex layout
- `lg:translate-x-0` — always on screen, ignoring the state variable
- `lg:block` — always visible

The hamburger button uses `lg:hidden` — it only appears on screens narrower than 1024px.

---

### Step 4 — Make the contacts table responsive with mobile card layout

The contacts table currently uses a `<table>` element. Tables do not reflow on narrow screens — they overflow horizontally. You solve this by rendering two different UIs: the table on `md` and wider, and a card list on smaller screens.

Open `app/contacts/page.tsx` (or your `ContactsTable` component). Find the `<table>` element and add `hidden md:table` to hide it on mobile. Below it, add a card list with `md:hidden`:

```tsx
{/* Table — hidden on mobile */}
<div className="hidden md:block overflow-x-auto rounded-lg border border-gray-200">
  <table className="w-full text-sm">
    <thead className="bg-gray-50 text-xs text-gray-500 uppercase">
      <tr>
        <th className="px-4 py-3 text-left">Name</th>
        <th className="px-4 py-3 text-left">Email</th>
        <th className="px-4 py-3 text-left">Phone</th>
        <th className="px-4 py-3 text-left">Company</th>
        <th className="px-4 py-3 text-right">Actions</th>
      </tr>
    </thead>
    <tbody className="divide-y divide-gray-100">
      {contacts.map(contact => (
        <tr key={contact.id} className="hover:bg-gray-50 transition-colors">
          {/* ... existing table cells ... */}
        </tr>
      ))}
    </tbody>
  </table>
</div>

{/* Card list — shown on mobile only */}
<div className="md:hidden space-y-3">
  {contacts.map(contact => (
    <div
      key={contact.id}
      className="bg-white border border-gray-200 rounded-xl p-4 space-y-2 shadow-sm"
    >
      {/* Name row */}
      <div className="flex items-start justify-between gap-2">
        <div>
          <p className="font-semibold text-gray-900 text-sm">
            {contact.first_name} {contact.last_name}
          </p>
          {contact.companies?.name && (
            <p className="text-xs text-gray-500 mt-0.5">{contact.companies.name}</p>
          )}
        </div>
        {/* Action button */}
        <Link
          href={`/contacts/${contact.id}`}
          className="shrink-0 text-xs text-emerald-600 font-medium hover:text-emerald-800 min-h-[44px] flex items-center"
        >
          View
        </Link>
      </div>

      {/* Email */}
      {contact.email && (
        <a
          href={`mailto:${contact.email}`}
          className="flex items-center gap-2 text-sm text-gray-600 hover:text-gray-900 min-h-[44px]"
        >
          <svg className="w-4 h-4 text-gray-400 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2}
              d="M3 8l7.89 5.26a2 2 0 002.22 0L21 8M5 19h14a2 2 0 002-2V7a2 2 0 00-2-2H5a2 2 0 00-2 2v10a2 2 0 002 2z"
            />
          </svg>
          {contact.email}
        </a>
      )}

      {/* Phone */}
      {contact.phone && (
        <a
          href={`tel:${contact.phone}`}
          className="flex items-center gap-2 text-sm text-gray-600 hover:text-gray-900 min-h-[44px]"
        >
          <svg className="w-4 h-4 text-gray-400 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2}
              d="M3 5a2 2 0 012-2h3.28a1 1 0 01.948.684l1.498 4.493a1 1 0 01-.502 1.21l-2.257 1.13a11.042 11.042 0 005.516 5.516l1.13-2.257a1 1 0 011.21-.502l4.493 1.498a1 1 0 01.684.948V19a2 2 0 01-2 2h-1C9.716 21 3 14.284 3 12V5z"
            />
          </svg>
          {contact.phone}
        </a>
      )}
    </div>
  ))}
</div>
```

The card design converts columns into labeled rows. Email and phone become tappable links (`mailto:` and `tel:`) — on a phone, tapping these opens the email client or dialer.

---

### Step 5 — Make all action buttons full-width on mobile

Any button that sits in an action bar should be full-width on mobile and auto-width on tablet and up:

```tsx
<button className="w-full md:w-auto px-4 py-2 min-h-[44px] ...">
  Add Contact
</button>
```

Apply this to: Add Contact, Export CSV, Import CSV, Add Deal, Add Company, Save on all forms.

---

### Step 6 — Audit with Chrome DevTools MCP

Use Chrome DevTools MCP from within Claude Code to catch overflow issues. Give Claude this exact prompt:

```
Use Chrome DevTools MCP to open http://localhost:3000/contacts in Chrome.
Set the viewport to 375px wide (iPhone SE size).
Take a screenshot and tell me what layout issues you see.
Fix any elements that overflow or are cut off.
```

Chrome DevTools MCP will:
1. Open a Chrome window and navigate to the URL
2. Set the viewport to 375px using the device emulation API
3. Take a screenshot and analyze it
4. Report any elements that are wider than the viewport

After the audit, give this follow-up prompt if issues are found:

```
Use Chrome DevTools MCP to check these specific pages at 375px viewport width:
- /contacts
- /deals
- /contacts/[pick any contact ID]

For each page, take a screenshot. Report any horizontal scroll, overflow, or elements cut off.
List the CSS class changes needed to fix each issue.
```

---

### Step 7 — Test on an actual mobile device with ngrok

Chrome DevTools emulation is not the same as a real phone. Test on a real device too.

Install ngrok (free tier — no account required for basic tunneling):

```bash
# macOS
brew install ngrok

# Windows (via Chocolatey)
choco install ngrok

# Or download directly from https://ngrok.com/download
```

Start your Next.js dev server:

```bash
# Terminal 1 — from your project root
npm run dev
# Running on http://localhost:3000
```

In a second terminal, expose port 3000:

```bash
# Terminal 2
ngrok http 3000
```

Expected output:

```
Session Status     online
Account            (not signed in)
Version            3.x.x
Region             United States (us)
Forwarding         https://a1b2-197-245-12-55.ngrok-free.app -> http://localhost:3000
```

Open `https://a1b2-197-245-12-55.ngrok-free.app` on your phone. This is a public HTTPS URL that tunnels to your local development server. Your phone accesses your Next.js app running on your laptop.

What ngrok does: it creates a secure tunnel from a public URL on ngrok's servers to your localhost. Traffic goes: your phone → ngrok server → your laptop via the ngrok agent → Next.js dev server. The tunnel is encrypted with HTTPS. It works on any network — your phone does not need to be on the same WiFi as your laptop.

The free tier gives you one tunnel with a random URL that changes each time you restart ngrok. For development testing, this is sufficient.

---

### Step 8 — Commit

```bash
git add .
git commit -m "feat: mobile responsive — collapsible sidebar, card tables, touch targets"
```

Expected output:

```
[main e8f5c03] feat: mobile responsive — collapsible sidebar, card tables, touch targets
 5 files changed, 189 insertions(+), 34 deletions(-)
```

---

## Practical workflow

Open your app in two browser windows side by side: one at full width (desktop) and one narrowed to 375px (drag the window edge). Build the mobile layout in the narrow window, then verify the desktop layout is unchanged in the wide window. This prevents you from fixing mobile by breaking desktop.

In Chrome, press `F12` → click the device toolbar icon (or `Ctrl+Shift+M`) → select "iPhone SE" from the device dropdown. This sets the viewport to 375×667px. Changes to the responsive layout show in real time.

---

## Common mistakes

**Sidebar shows on mobile even when closed.**
Check that `lg:relative` is applied to the sidebar wrapper. Without it, on large screens the sidebar is still `fixed` positioned and sits on top of the content. The `lg:` prefix means "apply this class at 1024px and wider."

**The backdrop overlay is behind the sidebar.**
The overlay needs `z-20` and the sidebar needs `z-30`. If both have the same z-index, the first one in the DOM wins — which may be the overlay, causing it to block the sidebar links.

**Clicking a nav link does not close the sidebar.**
You forgot to pass `onClick={onClose}` to the nav link, or the `onClose` prop was not passed from `AppShell` to `Sidebar`. Check the prop chain: `AppShell` → `Sidebar` → each `<Link>`.

**Cards appear on desktop in addition to the table.**
You used `md:hidden` on the card list but forgot `hidden md:block` on the table wrapper. Both end up visible. The pattern is: table uses `hidden md:block`, cards use `block md:hidden` (or `md:hidden`).

**Touch targets are smaller than 44px.**
Adding `min-h-[44px]` to a button sets the minimum height, but if the button has `flex items-center` with very small content, the button may still render smaller than 44px because `min-h` can be overridden by `flex` shrink. Add `shrink-0` to the button, or change to `h-11` (44px exact) if the height must be fixed.

**ngrok shows a "deceptive site" warning on first load.**
ngrok free tier adds a browser warning page before forwarding. Click "Visit Site" to proceed. This warning only appears once per session for a given ngrok URL. It does not appear on Vercel or any real domain.

---

## Your turn

1. Apply the card layout to the Companies table. Use the same pattern: `hidden md:block` on the table, `md:hidden` on the card list.
2. Apply the card layout to the Deals Kanban on mobile. A Kanban board at 375px is unusable — collapse it to a vertical list of deal cards, sorted by stage, each showing the stage name as a colored badge.
3. Make the dashboard chart scrollable horizontally on mobile: wrap it in `<div className="overflow-x-auto">`.
4. Test all form pages on mobile. Every `<input>` and `<textarea>` should be at least 44px tall. Add `min-h-[44px]` to all form inputs.

---

## Prompt / Template / Checklist pack

### Claude Code prompt — responsive sidebar and contact cards

```
I am building ClearCRM with Next.js 14 App Router and Tailwind CSS.

Make the app mobile responsive. Specifically:

1. Create components/AppShell.tsx — a 'use client' component with useState for sidebarOpen (default false).
   - Renders a fixed backdrop overlay (z-20, bg-black/40, backdrop-blur-sm) when sidebarOpen and on mobile (lg:hidden)
   - Clicking the overlay sets sidebarOpen to false
   - Renders the sidebar in a div with these classes:
     fixed inset-y-0 left-0 z-30 w-60 bg-[#0F172A]
     transform transition-transform duration-300 ease-in-out
     lg:relative lg:translate-x-0
     and conditionally: translate-x-0 (open) or -translate-x-full (closed)
   - Renders a top bar (lg:hidden) with a hamburger button (min-h-[44px] min-w-[44px]) that sets sidebarOpen to true
   - Renders a main content area with overflow-y-auto and padding p-4 lg:p-6

2. Update components/Sidebar.tsx to accept onClose?: () => void and call it on every nav link's onClick.

3. Update app/contacts/page.tsx to show:
   - A table wrapped in hidden md:block
   - A card list in md:hidden space-y-3
   Each card shows: contact name (font-semibold), company name (text-xs text-gray-500), email as mailto: link, phone as tel: link
   Email and phone links must have min-h-[44px]

4. Make all action buttons w-full md:w-auto and min-h-[44px].

Sidebar color: #0F172A. Accent: emerald-600. Do not install any new packages.
```

---

## Deliverable — Mobile responsiveness checklist

Work through this checklist on a real or emulated 375px viewport. Check off each item only when you have tested it — not just "I think it works."

- [ ] **Sidebar hidden on mobile**: At 375px, the sidebar is not visible on page load. No overflow, no content hidden behind it.
- [ ] **Hamburger button visible**: A hamburger icon appears in the top bar on mobile. It is at least 44px × 44px. Test: tap it (or click in DevTools).
- [ ] **Sidebar slides open**: Tapping the hamburger opens the sidebar with a slide-in animation. The sidebar covers the full height of the screen.
- [ ] **Backdrop overlay**: When the sidebar is open, a dark overlay covers the rest of the screen. It is not clickable-through — tapping it closes the sidebar.
- [ ] **Sidebar links close sidebar**: Tapping any nav link navigates to the page AND closes the sidebar. The content is immediately visible.
- [ ] **Contacts table hidden on mobile**: At 375px, the table is not visible. No horizontal scroll, no overflow.
- [ ] **Contact cards visible on mobile**: A card list replaces the table. Each card shows name, company, email, and phone. Email and phone are tappable links.
- [ ] **Deals Kanban usable on mobile**: The Kanban board does not overflow horizontally at 375px. Columns are scrollable or reformatted.
- [ ] **All buttons 44px tall**: Every button, including Add Contact, Export CSV, Import CSV, Save, Cancel, has at least 44px height. Verify with DevTools: inspect element → computed styles → height.
- [ ] **All form inputs 44px tall**: Every `<input>`, `<select>`, and `<textarea>` on form pages has min-height 44px. Verify on the Add Contact form.
- [ ] **No horizontal overflow on any page**: In Chrome DevTools at 375px, open each main page (contacts, companies, deals, dashboard, reports). None should show a horizontal scrollbar or overflow indicator.
- [ ] **Chrome DevTools MCP audit complete**: Run the Chrome DevTools MCP prompt from Step 6 on at least 3 pages. All reported issues are fixed.
- [ ] **ngrok tunnel working**: The app is accessible at an ngrok URL. You have opened it on a real phone and navigated at least 2 pages.
- [ ] **Sidebar hidden on desktop**: At 1024px and wider, the hamburger button is hidden. The sidebar is always visible. The layout matches the desktop design.
- [ ] **Commit pushed**: All mobile changes are committed. Run `git status` — no uncommitted changes remain.
