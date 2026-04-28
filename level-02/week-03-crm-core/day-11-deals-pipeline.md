# Day 11: Deals Pipeline — Kanban Board

## Outcome artifact

ClearCRM has a `/deals` page with a horizontal Kanban board showing all deals grouped into six stage columns: Lead, Qualified, Proposal, Negotiation, Won, and Lost. An "Add Deal" button opens a form to create a new deal linked to a contact. Each deal card shows the deal title, the linked contact's name, the value in South African Rand, and the close date.

---

## The core idea

A Kanban board is a visual pipeline. Each column represents a stage in your sales process. Each card represents a deal moving through that process. The board gives a sales team an instant read on where their pipeline sits — how many deals are in negotiation, what the total proposal value is, which deals closed as won or lost.

You do not need a drag-and-drop library to build this. CSS Flexbox handles the horizontal column layout. Grouping is done in JavaScript before any data reaches the component. The board is a read-only view today. Drag-and-drop comes on Day 12.

The Supabase query joins the `deals` table to the `contacts` table in a single request — no second query needed. Next.js server components fetch the data on the server before the page reaches the browser, so the user sees a fully populated board immediately with no loading spinner.

---

## Step-by-step walkthrough

### Step 1 — Create the deals table in Supabase

Open your Supabase project. Go to the SQL Editor. Run this exact SQL:

```sql
-- Create deals table
CREATE TABLE deals (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT NOT NULL,
  value NUMERIC(12, 2) NOT NULL DEFAULT 0,
  stage TEXT NOT NULL DEFAULT 'lead'
    CHECK (stage IN ('lead', 'qualified', 'proposal', 'negotiation', 'won', 'lost')),
  contact_id UUID REFERENCES contacts(id) ON DELETE SET NULL,
  close_date DATE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE deals ENABLE ROW LEVEL SECURITY;

-- Policy: authenticated users can read all deals
CREATE POLICY "Authenticated users can read deals"
  ON deals FOR SELECT
  TO authenticated
  USING (true);

-- Policy: authenticated users can insert deals
CREATE POLICY "Authenticated users can insert deals"
  ON deals FOR INSERT
  TO authenticated
  WITH CHECK (true);

-- Policy: authenticated users can update deals
CREATE POLICY "Authenticated users can update deals"
  ON deals FOR UPDATE
  TO authenticated
  USING (true);

-- Policy: authenticated users can delete deals
CREATE POLICY "Authenticated users can delete deals"
  ON deals FOR DELETE
  TO authenticated
  USING (true);
```

Expected output in the Supabase SQL Editor: a green success banner reading "Success. No rows returned."

Verify the table exists by clicking **Table Editor** in the left sidebar. You should see `deals` listed alongside `contacts` and `companies`.

---

### Step 2 — Create the deals page (server component)

Your terminal should be in the ClearCRM project root.

```
Directory: ~/clearcrm
Command:   mkdir -p app/deals
```

Create the file `app/deals/page.tsx`:

```tsx
// app/deals/page.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import KanbanBoard from './KanbanBoard'
import type { Database } from '@/types/database'

// The six stages in display order
const STAGES = ['lead', 'qualified', 'proposal', 'negotiation', 'won', 'lost'] as const
type Stage = typeof STAGES[number]

export type DealWithContact = {
  id: string
  title: string
  value: number
  stage: Stage
  close_date: string | null
  created_at: string
  contacts: {
    first_name: string
    last_name: string
  } | null
}

export default async function DealsPage() {
  const supabase = createServerComponentClient<Database>({ cookies })

  const { data: deals, error } = await supabase
    .from('deals')
    .select('*, contacts(first_name, last_name)')
    .order('created_at', { ascending: false })

  if (error) {
    console.error('Error fetching deals:', error)
  }

  // Group deals by stage
  const dealsByStage: Record<Stage, DealWithContact[]> = {
    lead: [],
    qualified: [],
    proposal: [],
    negotiation: [],
    won: [],
    lost: [],
  }

  for (const deal of (deals ?? []) as DealWithContact[]) {
    dealsByStage[deal.stage].push(deal)
  }

  return (
    <div className="p-6">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">Deals Pipeline</h1>
          <p className="text-sm text-gray-500 mt-1">
            {deals?.length ?? 0} deal{deals?.length !== 1 ? 's' : ''} total
          </p>
        </div>
      </div>

      <KanbanBoard dealsByStage={dealsByStage} stages={STAGES} />
    </div>
  )
}
```

---

### Step 3 — Create the Kanban board (client component)

Create `app/deals/KanbanBoard.tsx`:

```tsx
// app/deals/KanbanBoard.tsx
'use client'

import { useState } from 'react'
import DealCard from './DealCard'
import DealForm from './DealForm'
import type { DealWithContact } from './page'

const STAGE_LABELS: Record<string, string> = {
  lead: 'Lead',
  qualified: 'Qualified',
  proposal: 'Proposal',
  negotiation: 'Negotiation',
  won: 'Won',
  lost: 'Lost',
}

// Header colour per stage
const STAGE_HEADER_CLASS: Record<string, string> = {
  lead: 'bg-gray-100 text-gray-700',
  qualified: 'bg-blue-50 text-blue-700',
  proposal: 'bg-purple-50 text-purple-700',
  negotiation: 'bg-amber-50 text-amber-700',
  won: 'bg-emerald-100 text-emerald-800',
  lost: 'bg-red-100 text-red-700',
}

type Props = {
  dealsByStage: Record<string, DealWithContact[]>
  stages: readonly string[]
}

export default function KanbanBoard({ dealsByStage, stages }: Props) {
  const [showForm, setShowForm] = useState(false)

  return (
    <>
      {/* Add Deal button */}
      <div className="mb-4 flex justify-end">
        <button
          onClick={() => setShowForm(true)}
          className="bg-emerald-600 text-white text-sm font-medium px-4 py-2 rounded-lg hover:bg-emerald-700 transition-colors"
        >
          + Add Deal
        </button>
      </div>

      {/* Kanban columns — horizontal scroll on small screens */}
      <div className="flex gap-4 overflow-x-auto pb-6">
        {stages.map((stage) => {
          const stageDeals = dealsByStage[stage] ?? []
          const stageTotal = stageDeals.reduce((sum, d) => sum + (d.value ?? 0), 0)

          return (
            <div
              key={stage}
              className="flex-shrink-0 w-64 flex flex-col rounded-xl border border-gray-200 bg-gray-50 overflow-hidden"
            >
              {/* Column header */}
              <div className={`px-4 py-3 ${STAGE_HEADER_CLASS[stage]}`}>
                <div className="flex items-center justify-between">
                  <span className="font-semibold text-sm">{STAGE_LABELS[stage]}</span>
                  <span className="text-xs font-medium bg-white/60 rounded-full px-2 py-0.5">
                    {stageDeals.length}
                  </span>
                </div>
                {stageDeals.length > 0 && (
                  <p className="text-xs mt-1 opacity-70">
                    {new Intl.NumberFormat('en-ZA', {
                      style: 'currency',
                      currency: 'ZAR',
                      maximumFractionDigits: 0,
                    }).format(stageTotal)}
                  </p>
                )}
              </div>

              {/* Deal cards */}
              <div className="flex flex-col gap-3 p-3 flex-1">
                {stageDeals.length === 0 ? (
                  <p className="text-xs text-gray-400 text-center py-4">No deals</p>
                ) : (
                  stageDeals.map((deal) => (
                    <DealCard key={deal.id} deal={deal} />
                  ))
                )}
              </div>
            </div>
          )
        })}
      </div>

      {/* Add Deal modal */}
      {showForm && (
        <div className="fixed inset-0 bg-black/40 flex items-center justify-center z-50 p-4">
          <div className="bg-white rounded-2xl shadow-xl w-full max-w-md p-6">
            <div className="flex items-center justify-between mb-4">
              <h2 className="text-lg font-bold text-gray-900">New Deal</h2>
              <button
                onClick={() => setShowForm(false)}
                className="text-gray-400 hover:text-gray-600 text-xl leading-none"
              >
                ×
              </button>
            </div>
            <DealForm onSuccess={() => setShowForm(false)} />
          </div>
        </div>
      )}
    </>
  )
}
```

---

### Step 4 — Create the DealCard component

Create `app/deals/DealCard.tsx`:

```tsx
// app/deals/DealCard.tsx
import Link from 'next/link'
import type { DealWithContact } from './page'

type Props = {
  deal: DealWithContact
}

function formatCurrency(value: number): string {
  return new Intl.NumberFormat('en-ZA', {
    style: 'currency',
    currency: 'ZAR',
    maximumFractionDigits: 0,
  }).format(value)
}

function formatDate(dateStr: string | null): string {
  if (!dateStr) return ''
  return new Date(dateStr).toLocaleDateString('en-ZA', {
    day: 'numeric',
    month: 'short',
    year: 'numeric',
  })
}

export default function DealCard({ deal }: Props) {
  const contactName = deal.contacts
    ? `${deal.contacts.first_name} ${deal.contacts.last_name}`
    : 'No contact'

  return (
    <Link href={`/deals/${deal.id}`}>
      <div className="bg-white rounded-lg border border-gray-200 p-3 shadow-sm hover:shadow-md hover:border-emerald-300 transition-all cursor-pointer group">
        {/* Deal title */}
        <p className="text-sm font-semibold text-gray-900 group-hover:text-emerald-700 transition-colors leading-snug">
          {deal.title}
        </p>

        {/* Contact name */}
        <p className="text-xs text-gray-500 mt-1 flex items-center gap-1">
          <span>👤</span>
          {contactName}
        </p>

        {/* Value + close date */}
        <div className="flex items-center justify-between mt-2">
          <span className="text-sm font-bold text-emerald-700">
            {formatCurrency(deal.value)}
          </span>
          {deal.close_date && (
            <span className="text-xs text-gray-400">
              {formatDate(deal.close_date)}
            </span>
          )}
        </div>
      </div>
    </Link>
  )
}
```

---

### Step 5 — Create the DealForm component

Create `app/deals/DealForm.tsx`:

```tsx
// app/deals/DealForm.tsx
'use client'

import { useEffect, useState, useTransition } from 'react'
import { createClientComponentClient } from '@supabase/auth-helpers-nextjs'
import { addDeal } from './actions'
import { useRouter } from 'next/navigation'

const STAGES = [
  { value: 'lead', label: 'Lead' },
  { value: 'qualified', label: 'Qualified' },
  { value: 'proposal', label: 'Proposal' },
  { value: 'negotiation', label: 'Negotiation' },
  { value: 'won', label: 'Won' },
  { value: 'lost', label: 'Lost' },
]

type Contact = {
  id: string
  first_name: string
  last_name: string
}

type Props = {
  onSuccess: () => void
}

export default function DealForm({ onSuccess }: Props) {
  const [contacts, setContacts] = useState<Contact[]>([])
  const [isPending, startTransition] = useTransition()
  const [error, setError] = useState<string | null>(null)
  const router = useRouter()
  const supabase = createClientComponentClient()

  // Fetch contacts for the dropdown
  useEffect(() => {
    supabase
      .from('contacts')
      .select('id, first_name, last_name')
      .order('first_name')
      .then(({ data }) => {
        if (data) setContacts(data)
      })
  }, [supabase])

  async function handleSubmit(formData: FormData) {
    setError(null)
    startTransition(async () => {
      const result = await addDeal(formData)
      if (result?.error) {
        setError(result.error)
      } else {
        router.refresh()
        onSuccess()
      }
    })
  }

  return (
    <form action={handleSubmit} className="space-y-4">
      {/* Title */}
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Deal title <span className="text-red-500">*</span>
        </label>
        <input
          type="text"
          name="title"
          required
          placeholder="e.g. Website redesign for Acme Corp"
          className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
        />
      </div>

      {/* Value */}
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Value (ZAR) <span className="text-red-500">*</span>
        </label>
        <input
          type="number"
          name="value"
          required
          min="0"
          step="0.01"
          placeholder="0.00"
          className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
        />
      </div>

      {/* Stage */}
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Stage
        </label>
        <select
          name="stage"
          defaultValue="lead"
          className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
        >
          {STAGES.map((s) => (
            <option key={s.value} value={s.value}>
              {s.label}
            </option>
          ))}
        </select>
      </div>

      {/* Contact */}
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Contact
        </label>
        <select
          name="contact_id"
          className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
        >
          <option value="">— No contact —</option>
          {contacts.map((c) => (
            <option key={c.id} value={c.id}>
              {c.first_name} {c.last_name}
            </option>
          ))}
        </select>
      </div>

      {/* Close date */}
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Close date
        </label>
        <input
          type="date"
          name="close_date"
          className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
        />
      </div>

      {error && (
        <p className="text-sm text-red-600 bg-red-50 border border-red-200 rounded-lg px-3 py-2">
          {error}
        </p>
      )}

      <div className="flex gap-3 pt-2">
        <button
          type="submit"
          disabled={isPending}
          className="flex-1 bg-emerald-600 text-white text-sm font-medium py-2 rounded-lg hover:bg-emerald-700 disabled:opacity-60 transition-colors"
        >
          {isPending ? 'Saving…' : 'Add Deal'}
        </button>
      </div>
    </form>
  )
}
```

---

### Step 6 — Create the server actions

Create `app/deals/actions.ts`:

```ts
// app/deals/actions.ts
'use server'

import { createServerActionClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { revalidatePath } from 'next/cache'

export async function addDeal(formData: FormData) {
  const supabase = createServerActionClient({ cookies })

  const title = formData.get('title') as string
  const value = parseFloat(formData.get('value') as string)
  const stage = formData.get('stage') as string
  const contact_id = formData.get('contact_id') as string || null
  const close_date = formData.get('close_date') as string || null

  if (!title || isNaN(value)) {
    return { error: 'Title and value are required.' }
  }

  const { error } = await supabase.from('deals').insert({
    title,
    value,
    stage,
    contact_id: contact_id || null,
    close_date: close_date || null,
  })

  if (error) {
    console.error('addDeal error:', error)
    return { error: error.message }
  }

  revalidatePath('/deals')
}

export async function updateDeal(id: string, updates: {
  title?: string
  value?: number
  stage?: string
  contact_id?: string | null
  close_date?: string | null
}) {
  const supabase = createServerActionClient({ cookies })

  const { error } = await supabase
    .from('deals')
    .update(updates)
    .eq('id', id)

  if (error) {
    console.error('updateDeal error:', error)
    return { error: error.message }
  }

  revalidatePath('/deals')
  revalidatePath(`/deals/${id}`)
}

export async function deleteDeal(id: string) {
  const supabase = createServerActionClient({ cookies })

  const { error } = await supabase
    .from('deals')
    .delete()
    .eq('id', id)

  if (error) {
    console.error('deleteDeal error:', error)
    return { error: error.message }
  }

  revalidatePath('/deals')
}
```

---

### Step 7 — Wire the sidebar

Open your sidebar component (typically `components/Sidebar.tsx` or `app/layout.tsx`). Find the navigation links array and add the Deals entry:

```tsx
// Add to your nav links array
{ href: '/deals', label: 'Deals', icon: '💼' }
```

If your sidebar uses a `navLinks` array, it should look like this after the edit:

```tsx
const navLinks = [
  { href: '/dashboard', label: 'Dashboard', icon: '🏠' },
  { href: '/contacts', label: 'Contacts', icon: '👥' },
  { href: '/companies', label: 'Companies', icon: '🏢' },
  { href: '/deals', label: 'Deals', icon: '💼' },
]
```

---

### Step 8 — Test the pipeline

1. Run your dev server:

```
Directory: ~/clearcrm
Command:   npm run dev
Expected:  Server running on http://localhost:3000
```

2. Navigate to `http://localhost:3000/deals`. You should see six empty columns.

3. Click **+ Add Deal**. Fill in:
   - Title: `Website redesign for Acme Corp`
   - Value: `45000`
   - Stage: `Proposal`
   - Contact: pick any contact from the dropdown
   - Close date: a date one month from today

4. Submit. The modal closes and the deal appears in the **Proposal** column.

5. Add two more deals in different stages (e.g., Lead and Negotiation).

6. Verify: each deal appears in the correct column, the contact name shows correctly, and the value is formatted as ZAR currency.

---

### Step 9 — Commit

```
Directory: ~/clearcrm
Command:   git add . && git commit -m "feat: deals Kanban pipeline — 6 stages, add deal form"
Expected:  [main xxxxxxx] feat: deals Kanban pipeline — 6 stages, add deal form
```

---

## Practical workflow

**How the Supabase join works.** The query `select('*, contacts(first_name, last_name)')` tells Supabase to follow the foreign key from `deals.contact_id` to the `contacts` table and return only the two name columns. This is Supabase's PostgREST syntax for a join — no SQL JOIN keyword required. The result shape is:

```json
{
  "id": "...",
  "title": "Website redesign",
  "value": 45000,
  "stage": "proposal",
  "contacts": {
    "first_name": "Sarah",
    "last_name": "Johnson"
  }
}
```

**How grouping works.** In the server component you iterate over all deals and push each one into the correct bucket of the `dealsByStage` object. This runs on the server, so the client receives pre-grouped data — no grouping logic in the browser.

**Why server component for the page, client component for the board.** The page fetches data — that needs a server component. The board manages `showForm` state — that needs a client component. The split is: server for data, client for interactivity.

---

## Common mistakes

**Mistake: The contact dropdown in DealForm is empty.**
Cause: The `useEffect` fetch runs after render but the component uses the server-side Supabase client by mistake.
Fix: `DealForm` is a client component. Use `createClientComponentClient()` (not `createServerComponentClient`). Confirm the import is from `@supabase/auth-helpers-nextjs`.

**Mistake: "Error fetching deals" in the console, but the table exists.**
Cause: RLS policies are missing or the user is not authenticated.
Fix: Check the Supabase SQL Editor. Run `SELECT * FROM deals LIMIT 1;` — if it returns zero rows with no error, the table is fine. If it returns a permissions error, re-run the RLS policy SQL from Step 1.

**Mistake: The board is too wide and overflows without scrolling.**
Cause: The parent container has `overflow: hidden` or a fixed width that prevents `overflow-x-auto` from triggering.
Fix: Ensure the page wrapper does not have `overflow: hidden`. The `overflow-x-auto` class must be on a direct parent of the flex columns row.

**Mistake: `revalidatePath` does not update the board after adding a deal.**
Cause: The server action runs, revalidates, but the client component does not call `router.refresh()`.
Fix: After the server action resolves with no error, call `router.refresh()` in the client component. This tells Next.js to re-fetch the server component data.

**Mistake: Currency shows as `NaN` or `undefined`.**
Cause: `deal.value` is a string coming from the database, not a number, or the form field sends an empty string.
Fix: In the server action, wrap the value in `parseFloat()`. In `formatCurrency`, add a guard: `if (!value && value !== 0) return '—'`.

---

## Your turn

1. Add a total pipeline value display above the board — sum of all deals not in Won or Lost.
2. Change the Won column card background to a very light green (`bg-emerald-50`) and the Lost column card to light red (`bg-red-50`).
3. Add a "deals count" badge to the sidebar Deals link, showing the total number of open deals.

---

## Prompt / Template / Checklist pack

### Claude Code prompt — Build the deals Kanban pipeline

```
I'm building ClearCRM, a CRM app using Next.js 14 App Router, Supabase, and Tailwind CSS.

Build a Kanban pipeline board at /deals with these exact requirements:

DATABASE: The deals table already exists with these columns:
- id (UUID), title (TEXT), value (NUMERIC), stage (TEXT), contact_id (UUID FK → contacts), close_date (DATE), created_at (TIMESTAMPTZ)
- Stage values: lead, qualified, proposal, negotiation, won, lost

FILES TO CREATE:
1. app/deals/page.tsx — server component. Fetch all deals with contact name joined using: supabase.from('deals').select('*, contacts(first_name, last_name)').order('created_at', { ascending: false }). Group deals by stage into an object keyed by stage name. Pass the grouped data to KanbanBoard.

2. app/deals/KanbanBoard.tsx — client component ('use client'). Render 6 columns in a horizontal flex row with overflow-x-auto. Each column is 256px wide (w-64), flex-shrink-0. Column header: neutral for lead/qualified/proposal/negotiation, emerald green for won, red for lost. Show a deal count badge in the header. Show a column total (sum of deal values) in the header. Include an "+ Add Deal" button in the top-right that toggles a DealForm modal.

3. app/deals/DealCard.tsx — shows deal title, contact name (or "No contact"), value formatted as South African Rand using Intl.NumberFormat('en-ZA', { style: 'currency', currency: 'ZAR' }), and close date formatted as "15 Jan 2025". Wrap the entire card in a Next.js Link to /deals/[deal.id]. Hover state: border turns emerald, shadow increases.

4. app/deals/DealForm.tsx — client component. Fields: title (text, required), value (number, required), stage (select with all 6 options, default 'lead'), contact_id (select dropdown — fetch contacts client-side with useEffect using createClientComponentClient), close_date (date input). On submit call addDeal server action, on success call router.refresh() then call onSuccess prop.

5. app/deals/actions.ts — 'use server'. Three functions: addDeal(formData), updateDeal(id, updates), deleteDeal(id). All use createServerActionClient({ cookies }). All call revalidatePath('/deals') and revalidatePath('/deals/[id]') after success. Return { error: message } on failure.

STYLING:
- Use Tailwind CSS only. No external UI libraries.
- Main background: white. Sidebar already exists.
- Column card container background: bg-gray-50
- Deal card: white bg, border border-gray-200, rounded-lg, p-3, shadow-sm
- Emerald accent: #10B981 (emerald-600 in Tailwind)

Do not use any drag-and-drop library. Do not add features beyond what is listed. Show complete file contents for all 5 files.
```

---

### Deal stages reference card

| Stage | Definition | Move forward when… |
|-------|-----------|-------------------|
| **Lead** | An unqualified prospect — someone who showed interest but you have not yet confirmed they are a real opportunity. | You have spoken to them, confirmed budget exists, and there is a real decision-making process. |
| **Qualified** | You have confirmed the lead has a budget, a need, and authority to buy. The opportunity is real. | You have presented a solution and they have asked for a proposal or formal quote. |
| **Proposal** | A formal quote or proposal has been sent. The prospect is reviewing your offer. | They have responded with feedback, questions, or have started negotiating terms. |
| **Negotiation** | Both parties are actively discussing price, scope, or terms. The deal is close to closing. | Agreement is reached on all material terms and you are waiting for a signature or final approval. |
| **Won** | The deal is closed. The prospect signed, paid, or confirmed they are proceeding. | — (terminal stage) |
| **Lost** | The deal did not close. The prospect went with a competitor, cancelled, or went silent permanently. | — (terminal stage) |
