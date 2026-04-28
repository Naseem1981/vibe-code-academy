# Day 12: Deal Detail Page

## Outcome artifact

Each deal card in the Kanban board is a clickable link. Clicking it opens `/deals/[id]` — a full detail page for that deal. The page shows the deal's title, value, and close date. A stage selector displays all six stages as clickable buttons; clicking one immediately updates the stage in Supabase and highlights the new active stage in emerald green. A contact info card shows the linked contact's name, email, phone, and company. An activities section at the bottom shows a placeholder ready to be wired up fully on Day 13.

---

## The core idea

Next.js App Router uses the file system to define routes. A folder named `[id]` creates a dynamic segment — the `id` part of the URL becomes a variable your page can read. When a user visits `/deals/abc-123`, Next.js passes `{ id: 'abc-123' }` as the `params` prop to your page component.

The detail page is a server component that fetches one specific deal. The stage selector is a client component because it responds to user clicks. This is the same server/client split you used on Day 11 — server for data, client for interactivity.

Updating the stage calls a server action. The server action writes to Supabase, then calls `revalidatePath` so the next render of both the detail page and the pipeline board shows the updated stage without a full page reload.

---

## Step-by-step walkthrough

### Step 1 — Create the deal detail page

Your terminal should be in the ClearCRM project root.

```
Directory: ~/clearcrm
Command:   mkdir -p app/deals/[id]
```

Create `app/deals/[id]/page.tsx`:

```tsx
// app/deals/[id]/page.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { notFound } from 'next/navigation'
import Link from 'next/link'
import StageSelector from './StageSelector'
import type { Database } from '@/types/database'

type Props = {
  params: { id: string }
}

export default async function DealDetailPage({ params }: Props) {
  const supabase = createServerComponentClient<Database>({ cookies })

  // Fetch the deal with its linked contact and that contact's company
  const { data: deal, error } = await supabase
    .from('deals')
    .select(`
      *,
      contacts (
        id,
        first_name,
        last_name,
        email,
        phone,
        company_id,
        companies (
          name
        )
      )
    `)
    .eq('id', params.id)
    .single()

  if (error || !deal) {
    notFound()
  }

  const contact = deal.contacts as any
  const contactName = contact
    ? `${contact.first_name} ${contact.last_name}`
    : null

  const formattedValue = new Intl.NumberFormat('en-ZA', {
    style: 'currency',
    currency: 'ZAR',
    maximumFractionDigits: 0,
  }).format(deal.value ?? 0)

  const formattedCloseDate = deal.close_date
    ? new Date(deal.close_date).toLocaleDateString('en-ZA', {
        day: 'numeric',
        month: 'long',
        year: 'numeric',
      })
    : null

  return (
    <div className="p-6 max-w-4xl mx-auto">
      {/* Back link */}
      <Link
        href="/deals"
        className="inline-flex items-center gap-1 text-sm text-gray-500 hover:text-emerald-700 mb-6 transition-colors"
      >
        ← Back to Pipeline
      </Link>

      {/* Deal header */}
      <div className="bg-white rounded-2xl border border-gray-200 shadow-sm p-6 mb-6">
        <div className="flex items-start justify-between gap-4">
          <div>
            <h1 className="text-2xl font-bold text-gray-900">{deal.title}</h1>
            <p className="text-3xl font-bold text-emerald-700 mt-2">
              {formattedValue}
            </p>
            {formattedCloseDate && (
              <p className="text-sm text-gray-500 mt-1">
                Close date: {formattedCloseDate}
              </p>
            )}
          </div>
        </div>

        {/* Stage selector */}
        <div className="mt-6">
          <p className="text-xs font-semibold text-gray-500 uppercase tracking-wider mb-3">
            Stage
          </p>
          <StageSelector dealId={deal.id} currentStage={deal.stage} />
        </div>
      </div>

      {/* Contact info */}
      {contact && (
        <div className="bg-white rounded-2xl border border-gray-200 shadow-sm p-6 mb-6">
          <h2 className="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">
            Contact
          </h2>
          <div className="flex items-start gap-4">
            <div className="w-10 h-10 rounded-full bg-emerald-100 text-emerald-700 flex items-center justify-center font-bold text-sm flex-shrink-0">
              {contact.first_name[0]}{contact.last_name[0]}
            </div>
            <div className="flex-1 min-w-0">
              <Link
                href={`/contacts/${contact.id}`}
                className="text-base font-semibold text-gray-900 hover:text-emerald-700 transition-colors"
              >
                {contactName}
              </Link>
              {contact.companies?.name && (
                <p className="text-sm text-gray-500">{contact.companies.name}</p>
              )}
              <div className="mt-2 space-y-1">
                {contact.email && (
                  <p className="text-sm text-gray-600">
                    <span className="text-gray-400 mr-1">✉</span>
                    <a
                      href={`mailto:${contact.email}`}
                      className="hover:text-emerald-700 transition-colors"
                    >
                      {contact.email}
                    </a>
                  </p>
                )}
                {contact.phone && (
                  <p className="text-sm text-gray-600">
                    <span className="text-gray-400 mr-1">📞</span>
                    <a
                      href={`tel:${contact.phone}`}
                      className="hover:text-emerald-700 transition-colors"
                    >
                      {contact.phone}
                    </a>
                  </p>
                )}
              </div>
            </div>
          </div>
        </div>
      )}

      {/* Activities section — wired fully on Day 13 */}
      <div className="bg-white rounded-2xl border border-gray-200 shadow-sm p-6">
        <h2 className="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">
          Activities
        </h2>
        <p className="text-sm text-gray-400 text-center py-8">
          No activities yet. Activity log coming on Day 13.
        </p>
      </div>
    </div>
  )
}
```

---

### Step 2 — The Supabase query explained

The query for a single deal looks like this:

```ts
const { data: deal, error } = await supabase
  .from('deals')
  .select(`
    *,
    contacts (
      id,
      first_name,
      last_name,
      email,
      phone,
      company_id,
      companies (
        name
      )
    )
  `)
  .eq('id', params.id)
  .single()
```

Three things to understand here:

1. `.eq('id', params.id)` — filters to only the row where `id` matches the URL parameter.
2. `.single()` — instead of returning an array, this returns one object. If zero rows match (because the ID does not exist), Supabase returns an error, which triggers `notFound()`.
3. The nested `companies(name)` inside `contacts(...)` is a second-level join — Supabase follows `contacts.company_id → companies.id` automatically because the foreign key exists.

---

### Step 3 — Create the StageSelector component

Create `app/deals/[id]/StageSelector.tsx`:

```tsx
// app/deals/[id]/StageSelector.tsx
'use client'

import { useState, useTransition } from 'react'
import { updateStage } from './actions'

const STAGES: { value: string; label: string }[] = [
  { value: 'lead', label: 'Lead' },
  { value: 'qualified', label: 'Qualified' },
  { value: 'proposal', label: 'Proposal' },
  { value: 'negotiation', label: 'Negotiation' },
  { value: 'won', label: 'Won' },
  { value: 'lost', label: 'Lost' },
]

const STAGE_ACTIVE_CLASS: Record<string, string> = {
  lead: 'bg-gray-700 text-white border-gray-700',
  qualified: 'bg-blue-600 text-white border-blue-600',
  proposal: 'bg-purple-600 text-white border-purple-600',
  negotiation: 'bg-amber-500 text-white border-amber-500',
  won: 'bg-emerald-600 text-white border-emerald-600',
  lost: 'bg-red-600 text-white border-red-600',
}

type Props = {
  dealId: string
  currentStage: string
}

export default function StageSelector({ dealId, currentStage }: Props) {
  const [activeStage, setActiveStage] = useState(currentStage)
  const [isPending, startTransition] = useTransition()

  function handleStageClick(stage: string) {
    if (stage === activeStage) return

    // Optimistic update — update the UI immediately
    setActiveStage(stage)

    startTransition(async () => {
      const result = await updateStage(dealId, stage)
      if (result?.error) {
        // Revert on error
        setActiveStage(currentStage)
        alert(`Failed to update stage: ${result.error}`)
      }
    })
  }

  return (
    <div className="flex flex-wrap gap-2">
      {STAGES.map((stage) => {
        const isActive = stage.value === activeStage
        return (
          <button
            key={stage.value}
            onClick={() => handleStageClick(stage.value)}
            disabled={isPending}
            className={`
              px-3 py-1.5 rounded-full text-sm font-medium border transition-all
              ${isActive
                ? STAGE_ACTIVE_CLASS[stage.value]
                : 'bg-white text-gray-600 border-gray-300 hover:border-gray-400 hover:bg-gray-50'
              }
              ${isPending ? 'opacity-60 cursor-not-allowed' : 'cursor-pointer'}
            `}
          >
            {stage.label}
          </button>
        )
      })}
    </div>
  )
}
```

The `useTransition` hook is important here. It marks the stage update as a non-urgent transition, so the UI stays responsive while the server action runs. The optimistic update pattern (update local state immediately, revert on error) makes the stage change feel instant to the user.

---

### Step 4 — Create the updateStage server action

Create `app/deals/[id]/actions.ts`:

```ts
// app/deals/[id]/actions.ts
'use server'

import { createServerActionClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { revalidatePath } from 'next/cache'

const VALID_STAGES = ['lead', 'qualified', 'proposal', 'negotiation', 'won', 'lost']

export async function updateStage(dealId: string, stage: string) {
  if (!VALID_STAGES.includes(stage)) {
    return { error: 'Invalid stage value.' }
  }

  const supabase = createServerActionClient({ cookies })

  const { error } = await supabase
    .from('deals')
    .update({ stage })
    .eq('id', dealId)

  if (error) {
    console.error('updateStage error:', error)
    return { error: error.message }
  }

  // Revalidate both the detail page and the pipeline board
  revalidatePath(`/deals/${dealId}`)
  revalidatePath('/deals')
}
```

The server validates the `stage` value against the allowed list before writing to Supabase. This prevents a malicious request from writing an invalid stage value directly.

---

### Step 5 — Confirm DealCard links to the detail page

Open `app/deals/DealCard.tsx`. The card should already be wrapped in a `<Link>` pointing to `/deals/${deal.id}`. Verify this is in place:

```tsx
// app/deals/DealCard.tsx — confirm this wrapper exists
import Link from 'next/link'

export default function DealCard({ deal }: Props) {
  return (
    <Link href={`/deals/${deal.id}`}>
      <div className="bg-white rounded-lg border border-gray-200 p-3 ...">
        {/* card content */}
      </div>
    </Link>
  )
}
```

If you built DealCard exactly as shown on Day 11, this is already correct.

---

### Step 6 — Test the deal detail page

1. Confirm your dev server is running (`npm run dev`).

2. Navigate to `/deals`. Click any deal card. You should land on `/deals/[some-uuid]`.

3. Verify the page shows:
   - The deal title as an `<h1>`
   - The value formatted as ZAR currency
   - The close date (if set)
   - Six stage buttons, with the current stage highlighted
   - The contact card with name, email, phone

4. Click a different stage button. The button should highlight immediately (optimistic update). Wait one second. Navigate back to `/deals`. The card in the Kanban board should now show the updated stage column — because `revalidatePath('/deals')` was called.

5. Click the **← Back to Pipeline** link. Confirm it takes you back to `/deals`.

---

### Step 7 — Commit

```
Directory: ~/clearcrm
Command:   git add . && git commit -m "feat: deal detail page with stage selector and contact info"
Expected:  [main xxxxxxx] feat: deal detail page with stage selector and contact info
```

---

## Practical workflow

**How to access `params` in a server component.** In Next.js 14 App Router, dynamic route params are passed as a prop to the default export of a `page.tsx`:

```tsx
// The params prop shape mirrors the folder structure
// app/deals/[id]/page.tsx → params.id
// app/contacts/[id]/activities/[activityId]/page.tsx → params.id, params.activityId
type Props = {
  params: { id: string }
}

export default async function Page({ params }: Props) {
  console.log(params.id) // the UUID from the URL
}
```

The `id` is always a string, even if it looks like a number. Parse it if you need it as a number.

**Why optimistic updates matter.** Without optimistic updates, the user clicks a stage button, waits for the round trip to Supabase (300–600ms), and then sees the button highlight. With optimistic updates, the button highlights instantly and the server call happens in the background. If the server call fails, you revert. The UX is dramatically better.

**`notFound()` vs returning null.** When a deal ID does not exist in the database, calling `notFound()` from `next/navigation` returns a proper 404 page. Returning `null` or rendering an empty page would silently succeed with HTTP 200 — which is wrong. Always use `notFound()` when a required resource is missing.

---

## Common mistakes

**Mistake: `params.id` is undefined.**
Cause: The folder is named `[id]` but the component reads `params.dealId` or another name.
Fix: The variable name in `params` must exactly match the folder name in brackets. Folder `[id]` → `params.id`. Folder `[dealId]` → `params.dealId`.

**Mistake: `.single()` throws an error even though the deal exists.**
Cause: The RLS policy is missing or the user session is not being read correctly in the server component.
Fix: Use `createServerComponentClient({ cookies })` — not `createClient()`. The `cookies()` call reads the session from the HTTP request.

**Mistake: Stage buttons do not visually update until page refresh.**
Cause: Local state update is missing. The `updateStage` action runs but the component does not call `setActiveStage`.
Fix: Call `setActiveStage(stage)` before `startTransition`. This is the optimistic update — it must happen synchronously before the async transition.

**Mistake: The contact card shows "undefined undefined" for the name.**
Cause: The `contacts` join returns `null` when `contact_id` is null on the deal.
Fix: Guard with `if (!contact) return null` or use the conditional render shown in Step 1: `{contact && (<div>...</div>)}`.

**Mistake: Navigating back to `/deals` shows the old stage on the card.**
Cause: `revalidatePath('/deals')` was not called in the server action.
Fix: Add `revalidatePath('/deals')` alongside `revalidatePath('/deals/${dealId}')` in `updateStage`.

---

## Your turn

1. Add a "Delete Deal" button on the detail page. Clicking it should call a `deleteDeal` server action, delete the deal from Supabase, then redirect to `/deals` using `redirect('/deals')` from `next/navigation`.
2. Add an "Edit Deal" section — a small form below the stage selector that lets you change the deal title, value, and close date.
3. Add a breadcrumb trail at the top: `Deals → [deal title]` where "Deals" links back to `/deals`.

---

## Prompt / Template / Checklist pack

### Claude Code prompt — Build the deal detail page

```
I'm building ClearCRM with Next.js 14 App Router, Supabase, and Tailwind CSS.

Build the deal detail page at app/deals/[id]/page.tsx with these exact requirements:

ROUTE: app/deals/[id]/page.tsx — dynamic route. Access the deal ID via params.id.

SUPABASE QUERY: Fetch the deal with a nested contact + company join:
supabase
  .from('deals')
  .select(`*, contacts(id, first_name, last_name, email, phone, companies(name))`)
  .eq('id', params.id)
  .single()

If error or no data, call notFound() from 'next/navigation'.

PAGE LAYOUT (top to bottom):
1. A "← Back to Pipeline" link pointing to /deals (text-sm text-gray-500, hover:text-emerald-700)
2. A white card (rounded-2xl border border-gray-200 shadow-sm p-6) containing:
   - Deal title as h1 (text-2xl font-bold text-gray-900)
   - Deal value formatted as ZAR (text-3xl font-bold text-emerald-700) using Intl.NumberFormat('en-ZA', { style: 'currency', currency: 'ZAR' })
   - Close date as formatted string below the value (text-sm text-gray-500)
   - A StageSelector client component below (see below)
3. A contact info card (white, same styling) showing: avatar initials, full name (linked to /contacts/[contact.id]), company name, email (mailto link), phone (tel link). Hide this card entirely if contact is null.
4. An activities placeholder card with heading "Activities" and an empty state message "No activities yet."

STAGE SELECTOR (app/deals/[id]/StageSelector.tsx — client component):
- Props: dealId (string), currentStage (string)
- Show 6 pill buttons: Lead, Qualified, Proposal, Negotiation, Won, Lost
- Active stage has a filled background colour (emerald for won, red for lost, others use their own colour)
- Inactive stages: white bg, gray border, subtle hover
- On click: call setActiveStage immediately (optimistic update), then call updateStage server action inside useTransition. On error, revert setActiveStage to currentStage.

SERVER ACTION (app/deals/[id]/actions.ts — 'use server'):
export async function updateStage(dealId: string, stage: string) — validate stage is in the allowed list, update deals table, revalidatePath for both /deals/[id] and /deals, return { error } on failure.

Do not use any UI library. Tailwind only. Show complete file contents for all 3 files.
```

---

### Next.js dynamic routes cheat sheet

**File structure for a dynamic route:**

```
app/
  deals/
    page.tsx              → /deals
    [id]/
      page.tsx            → /deals/abc-123
      actions.ts          → server actions for this route
      StageSelector.tsx   → client component used by the page
```

**Accessing params in a server component:**

```tsx
// app/deals/[id]/page.tsx
type Props = {
  params: { id: string }          // matches the [id] folder name exactly
}

export default async function DealDetailPage({ params }: Props) {
  const id = params.id            // string, always — even if it looks like a number
  // use id in your Supabase query
}
```

**Fetching a single record from Supabase by ID:**

```ts
// Pattern for fetching one record
const { data, error } = await supabase
  .from('table_name')
  .select('*')
  .eq('id', params.id)   // filter to one row
  .single()              // return object instead of array; error if 0 or >1 rows

if (error || !data) {
  notFound()             // renders the Next.js 404 page
}
```

**Nested joins in a single query:**

```ts
// Follow two levels of foreign keys in one request
const { data } = await supabase
  .from('deals')
  .select(`
    *,
    contacts (
      id,
      first_name,
      last_name,
      companies (
        name
      )
    )
  `)
  .eq('id', params.id)
  .single()

// Result shape:
// {
//   id: '...',
//   title: '...',
//   contacts: {
//     id: '...',
//     first_name: 'Sarah',
//     companies: { name: 'Acme Corp' }
//   }
// }
```

**Redirecting from a server action:**

```ts
// In a server action (actions.ts)
import { redirect } from 'next/navigation'

export async function deleteDeal(dealId: string) {
  // ... delete from Supabase ...
  redirect('/deals')      // redirects the user after the action completes
}
```
