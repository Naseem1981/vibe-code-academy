# Day 13: Activity Log

## Outcome artifact

Every contact detail page and every deal detail page has a live Activity Log section. You can add a note, log a call, or record an email — each linked to the contact or deal you are viewing. Activities are saved to Supabase and displayed in reverse chronological order with a type icon, description, and time ago (e.g., "2 hours ago"). The components are reusable: the same `ActivityLog` and `AddActivity` components work on both the contact page and the deal page.

---

## The core idea

The activity log is a chronological record of every interaction with a contact or deal. In a real sales team, this is how salespeople hand off work to each other, track follow-ups, and remember what was promised. A note is an internal observation. A call is a logged phone conversation. An email is a record of correspondence sent.

Three components solve this:

1. `ActivityLog` — reads activities from Supabase and renders the feed. Reusable.
2. `AddActivity` — a form for adding a new activity. Reusable.
3. `addActivity` — a server action that writes to Supabase.

Both components accept optional `contactId` and `dealId` props. Pass only the relevant ID depending on which page you are on. The Supabase query filters on whichever ID is provided.

Today you also build the contact detail page at `/contacts/[id]` — because activities need somewhere to live on the contact side.

---

## Step-by-step walkthrough

### Step 1 — Create the activities table in Supabase

Open the Supabase SQL Editor. Run this exact SQL:

```sql
-- Create activities table
CREATE TABLE activities (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  type TEXT NOT NULL CHECK (type IN ('note', 'call', 'email')),
  description TEXT NOT NULL,
  contact_id UUID REFERENCES contacts(id) ON DELETE CASCADE,
  deal_id UUID REFERENCES deals(id) ON DELETE CASCADE,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- At least one of contact_id or deal_id must be set
ALTER TABLE activities
  ADD CONSTRAINT activity_has_context
  CHECK (contact_id IS NOT NULL OR deal_id IS NOT NULL);

-- Enable Row Level Security
ALTER TABLE activities ENABLE ROW LEVEL SECURITY;

-- Policy: authenticated users can read all activities
CREATE POLICY "Authenticated users can read activities"
  ON activities FOR SELECT
  TO authenticated
  USING (true);

-- Policy: authenticated users can insert activities
CREATE POLICY "Authenticated users can insert activities"
  ON activities FOR INSERT
  TO authenticated
  WITH CHECK (true);

-- Policy: authenticated users can delete activities
CREATE POLICY "Authenticated users can delete activities"
  ON activities FOR DELETE
  TO authenticated
  USING (true);

-- Index for fast lookups by contact or deal
CREATE INDEX idx_activities_contact_id ON activities(contact_id);
CREATE INDEX idx_activities_deal_id ON activities(deal_id);
```

Expected output: a green success banner reading "Success. No rows returned."

The constraint `activity_has_context` ensures you can never accidentally save an activity with no link to anything. The indexes speed up queries when you filter by `contact_id` or `deal_id`.

---

### Step 2 — Create the contact detail page

```
Directory: ~/clearcrm
Command:   mkdir -p app/contacts/[id]
```

Create `app/contacts/[id]/page.tsx`:

```tsx
// app/contacts/[id]/page.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { notFound } from 'next/navigation'
import Link from 'next/link'
import ActivityLog from '@/components/ActivityLog'
import type { Database } from '@/types/database'

type Props = {
  params: { id: string }
}

export default async function ContactDetailPage({ params }: Props) {
  const supabase = createServerComponentClient<Database>({ cookies })

  const { data: contact, error } = await supabase
    .from('contacts')
    .select(`
      *,
      companies (
        id,
        name,
        website
      )
    `)
    .eq('id', params.id)
    .single()

  if (error || !contact) {
    notFound()
  }

  const fullName = `${contact.first_name} ${contact.last_name}`

  return (
    <div className="p-6 max-w-4xl mx-auto">
      {/* Back link */}
      <Link
        href="/contacts"
        className="inline-flex items-center gap-1 text-sm text-gray-500 hover:text-emerald-700 mb-6 transition-colors"
      >
        ← Back to Contacts
      </Link>

      {/* Contact header */}
      <div className="bg-white rounded-2xl border border-gray-200 shadow-sm p-6 mb-6">
        <div className="flex items-start gap-4">
          <div className="w-14 h-14 rounded-full bg-emerald-100 text-emerald-700 flex items-center justify-center font-bold text-lg flex-shrink-0">
            {contact.first_name[0]}{contact.last_name[0]}
          </div>
          <div className="flex-1 min-w-0">
            <h1 className="text-2xl font-bold text-gray-900">{fullName}</h1>
            {contact.companies && (
              <p className="text-sm text-gray-500 mt-0.5">
                {(contact.companies as any).name}
              </p>
            )}
            <div className="mt-3 flex flex-wrap gap-4">
              {contact.email && (
                <a
                  href={`mailto:${contact.email}`}
                  className="text-sm text-gray-600 hover:text-emerald-700 transition-colors flex items-center gap-1"
                >
                  ✉ {contact.email}
                </a>
              )}
              {contact.phone && (
                <a
                  href={`tel:${contact.phone}`}
                  className="text-sm text-gray-600 hover:text-emerald-700 transition-colors flex items-center gap-1"
                >
                  📞 {contact.phone}
                </a>
              )}
            </div>
          </div>
        </div>
      </div>

      {/* Activity log */}
      <ActivityLog contactId={contact.id} />
    </div>
  )
}
```

---

### Step 3 — Create the ActivityLog component

```
Directory: ~/clearcrm
Command:   mkdir -p components
```

Create `components/ActivityLog.tsx`:

```tsx
// components/ActivityLog.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import AddActivity from './AddActivity'

const TYPE_ICON: Record<string, string> = {
  note: '📝',
  call: '📞',
  email: '📧',
}

const TYPE_LABEL: Record<string, string> = {
  note: 'Note',
  call: 'Call',
  email: 'Email',
}

function timeAgo(date: string): string {
  const seconds = Math.floor((Date.now() - new Date(date).getTime()) / 1000)
  if (seconds < 60) return 'just now'
  if (seconds < 3600) return `${Math.floor(seconds / 60)} minutes ago`
  if (seconds < 86400) return `${Math.floor(seconds / 3600)} hours ago`
  return `${Math.floor(seconds / 86400)} days ago`
}

type Props = {
  contactId?: string
  dealId?: string
}

export default async function ActivityLog({ contactId, dealId }: Props) {
  const supabase = createServerComponentClient({ cookies })

  // Build the query depending on which ID was provided
  let query = supabase
    .from('activities')
    .select('*')
    .order('created_at', { ascending: false })
    .limit(50)

  if (contactId) {
    query = query.eq('contact_id', contactId)
  } else if (dealId) {
    query = query.eq('deal_id', dealId)
  }

  const { data: activities } = await query

  return (
    <div className="bg-white rounded-2xl border border-gray-200 shadow-sm p-6">
      <h2 className="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">
        Activity Log
      </h2>

      {/* Add activity form */}
      <AddActivity contactId={contactId} dealId={dealId} />

      {/* Activity feed */}
      <div className="mt-6">
        {!activities || activities.length === 0 ? (
          <p className="text-sm text-gray-400 text-center py-6">
            No activities yet. Add your first note, call, or email above.
          </p>
        ) : (
          <div className="space-y-4">
            {activities.map((activity) => (
              <div key={activity.id} className="flex gap-3">
                {/* Type icon */}
                <div className="w-8 h-8 rounded-full bg-gray-100 flex items-center justify-center flex-shrink-0 text-sm mt-0.5">
                  {TYPE_ICON[activity.type]}
                </div>

                {/* Content */}
                <div className="flex-1 min-w-0">
                  <div className="flex items-center gap-2 mb-1">
                    <span className="text-xs font-semibold text-gray-600">
                      {TYPE_LABEL[activity.type]}
                    </span>
                    <span className="text-xs text-gray-400">
                      {timeAgo(activity.created_at)}
                    </span>
                  </div>
                  <p className="text-sm text-gray-700 leading-relaxed whitespace-pre-wrap">
                    {activity.description}
                  </p>
                </div>
              </div>
            ))}
          </div>
        )}
      </div>
    </div>
  )
}
```

`ActivityLog` is a server component. It fetches data directly from Supabase on the server. This means the activity feed is server-rendered — no loading state, no useEffect. When you need to refresh after adding an activity, `revalidatePath` handles it.

---

### Step 4 — Create the AddActivity component

Create `components/AddActivity.tsx`:

```tsx
// components/AddActivity.tsx
'use client'

import { useState, useTransition, useRef } from 'react'
import { addActivity } from '@/app/actions/activities'
import { useRouter } from 'next/navigation'

const TYPES = [
  { value: 'note', label: 'Note', icon: '📝' },
  { value: 'call', label: 'Call', icon: '📞' },
  { value: 'email', label: 'Email', icon: '📧' },
]

type Props = {
  contactId?: string
  dealId?: string
}

export default function AddActivity({ contactId, dealId }: Props) {
  const [selectedType, setSelectedType] = useState<'note' | 'call' | 'email'>('note')
  const [isPending, startTransition] = useTransition()
  const [error, setError] = useState<string | null>(null)
  const formRef = useRef<HTMLFormElement>(null)
  const router = useRouter()

  async function handleSubmit(formData: FormData) {
    setError(null)
    formData.set('type', selectedType)
    if (contactId) formData.set('contact_id', contactId)
    if (dealId) formData.set('deal_id', dealId)

    startTransition(async () => {
      const result = await addActivity(formData)
      if (result?.error) {
        setError(result.error)
      } else {
        formRef.current?.reset()
        router.refresh()
      }
    })
  }

  return (
    <form ref={formRef} action={handleSubmit} className="space-y-3">
      {/* Type selector */}
      <div className="flex gap-2">
        {TYPES.map((t) => (
          <button
            key={t.value}
            type="button"
            onClick={() => setSelectedType(t.value as typeof selectedType)}
            className={`
              flex items-center gap-1.5 px-3 py-1.5 rounded-full text-sm font-medium border transition-all
              ${selectedType === t.value
                ? 'bg-emerald-600 text-white border-emerald-600'
                : 'bg-white text-gray-600 border-gray-300 hover:border-gray-400'
              }
            `}
          >
            <span>{t.icon}</span>
            {t.label}
          </button>
        ))}
      </div>

      {/* Description textarea */}
      <textarea
        name="description"
        required
        rows={3}
        placeholder={
          selectedType === 'note'
            ? 'Write a note…'
            : selectedType === 'call'
            ? 'What was discussed on the call?'
            : 'What email was sent or received?'
        }
        className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 resize-none"
      />

      {error && (
        <p className="text-sm text-red-600 bg-red-50 border border-red-200 rounded-lg px-3 py-2">
          {error}
        </p>
      )}

      <button
        type="submit"
        disabled={isPending}
        className="bg-emerald-600 text-white text-sm font-medium px-4 py-2 rounded-lg hover:bg-emerald-700 disabled:opacity-60 transition-colors"
      >
        {isPending ? 'Saving…' : 'Log activity'}
      </button>
    </form>
  )
}
```

---

### Step 5 — Create the addActivity server action

```
Directory: ~/clearcrm
Command:   mkdir -p app/actions
```

Create `app/actions/activities.ts`:

```ts
// app/actions/activities.ts
'use server'

import { createServerActionClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { revalidatePath } from 'next/cache'

const VALID_TYPES = ['note', 'call', 'email']

export async function addActivity(formData: FormData) {
  const supabase = createServerActionClient({ cookies })

  const type = formData.get('type') as string
  const description = (formData.get('description') as string)?.trim()
  const contact_id = formData.get('contact_id') as string | null
  const deal_id = formData.get('deal_id') as string | null

  // Validate
  if (!VALID_TYPES.includes(type)) {
    return { error: 'Invalid activity type.' }
  }

  if (!description) {
    return { error: 'Description is required.' }
  }

  if (!contact_id && !deal_id) {
    return { error: 'Activity must be linked to a contact or a deal.' }
  }

  const { error } = await supabase.from('activities').insert({
    type,
    description,
    contact_id: contact_id || null,
    deal_id: deal_id || null,
  })

  if (error) {
    console.error('addActivity error:', error)
    return { error: error.message }
  }

  // Revalidate the relevant pages so the feed updates
  if (contact_id) revalidatePath(`/contacts/${contact_id}`)
  if (deal_id) revalidatePath(`/deals/${deal_id}`)
}
```

---

### Step 6 — The time-ago formatter

The `timeAgo` function is already included in `ActivityLog.tsx`. Here is the complete standalone function for reference — use it anywhere else you need relative time:

```ts
function timeAgo(date: string): string {
  const seconds = Math.floor((Date.now() - new Date(date).getTime()) / 1000)
  if (seconds < 60) return 'just now'
  if (seconds < 3600) return `${Math.floor(seconds / 60)} minutes ago`
  if (seconds < 86400) return `${Math.floor(seconds / 3600)} hours ago`
  return `${Math.floor(seconds / 86400)} days ago`
}
```

No library. No dependency. The logic: convert the difference between now and the stored timestamp to seconds. Check which threshold the result falls under and return the appropriate string. This covers "just now" (under 60 seconds), minutes, hours, and days. Extend it with weeks or months if you need longer time ranges.

---

### Step 7 — Wire ActivityLog into the deal detail page

Open `app/deals/[id]/page.tsx`. Replace the placeholder activities section with the real component:

```tsx
// app/deals/[id]/page.tsx — update the import at the top
import ActivityLog from '@/components/ActivityLog'

// Replace this placeholder section:
// <div className="bg-white rounded-2xl ...">
//   <p>No activities yet. Activity log coming on Day 13.</p>
// </div>

// With this:
<ActivityLog dealId={deal.id} />
```

The component is the same one used on the contact page. It accepts either `contactId` or `dealId` — here you pass `dealId`.

---

### Step 8 — Test both pages

1. Navigate to `/contacts`. Click any contact name (or add a link to the contact list pointing to `/contacts/[id]` if you have not already — see the note below).

2. On the contact detail page, select **Note**, type a test note, click **Log activity**. The note should appear immediately in the feed below.

3. Navigate to `/deals`. Click any deal card. On the deal detail page, select **Call**, describe the call, submit. The call appears in the deal's activity feed.

4. Check that the activity log on the contact page does not show the deal's activity (the query filters by `contact_id` only when `contactId` is passed).

**Note on the contacts list.** If your contacts list page (`app/contacts/page.tsx`) shows contact names as plain text, update each row to be a link:

```tsx
// In your contacts list — make the name a link
<Link href={`/contacts/${contact.id}`} className="font-medium text-gray-900 hover:text-emerald-700">
  {contact.first_name} {contact.last_name}
</Link>
```

---

### Step 9 — Commit

```
Directory: ~/clearcrm
Command:   git add . && git commit -m "feat: activity log — notes, calls, emails on contacts and deals"
Expected:  [main xxxxxxx] feat: activity log — notes, calls, emails on contacts and deals
```

---

## Practical workflow

**Why `ActivityLog` is a server component but `AddActivity` is a client component.** `ActivityLog` only reads data — it fetches from Supabase and renders HTML. No state, no event handlers. Server component. `AddActivity` manages form state, type selection, and calls `router.refresh()` after submission. All of that requires the browser. Client component.

**Why `router.refresh()` instead of another approach.** After `addActivity` runs, the `ActivityLog` server component needs to re-fetch its data. `router.refresh()` tells Next.js to re-run the server components for the current route without losing client state. The activity feed updates, the form resets, and nothing else changes.

**The `formRef.current?.reset()` pattern.** HTML forms remember their values after submission unless you reset them. `formRef` holds a reference to the `<form>` element. After a successful submission, calling `.reset()` on it clears all input fields — no need to manage each field with useState.

**Filtering activities by both contact and deal.** The current implementation filters by either `contact_id` or `deal_id`. If you ever need activities that belong to a specific contact AND a specific deal (for a combined timeline view), change the query to use `.or()`:

```ts
query = query.or(`contact_id.eq.${contactId},deal_id.eq.${dealId}`)
```

---

## Common mistakes

**Mistake: The activity feed does not update after submission.**
Cause: `router.refresh()` is not being called, or the `ActivityLog` component is not a server component (so it does not re-render on refresh).
Fix: Confirm `AddActivity` calls `router.refresh()` after `addActivity` resolves with no error. Confirm `ActivityLog` has no `'use client'` directive.

**Mistake: Activities from one context appear in another (e.g., deal activities show on the contact page).**
Cause: The query is missing the `.eq()` filter, or the wrong ID is being passed as a prop.
Fix: Check the `ActivityLog` call on each page. Contact page: `<ActivityLog contactId={contact.id} />`. Deal page: `<ActivityLog dealId={deal.id} />`. The query uses `if (contactId)` and `else if (dealId)` — make sure you did not accidentally pass both.

**Mistake: The CHECK constraint `activity_has_context` causes an insert error.**
Cause: Both `contact_id` and `deal_id` are being sent as empty strings instead of `null`.
Fix: In the server action, use `contact_id || null` — not `contact_id || ''`. An empty string is not null.

**Mistake: "just now" shows for activities that are 5 minutes old.**
Cause: The `created_at` value from Supabase is UTC, but `Date.now()` is also UTC — they should match. The real cause is usually a Supabase column default that is set as `now()` in local time without timezone.
Fix: Confirm the column type is `TIMESTAMPTZ` (with timezone), not `TIMESTAMP`. `TIMESTAMPTZ` stores UTC. `TIMESTAMP` stores whatever local time the server happens to use.

**Mistake: TypeScript error on `contact.companies?.name`.**
Cause: Supabase infers the type as `Json | null` when the join goes through multiple levels.
Fix: Cast the joined data with `as any` or add a proper type in your `Database` type definition. For now, `(contact.companies as any)?.name` is the quickest fix.

---

## Your turn

1. Add a "delete" button next to each activity in the feed. Create a `deleteActivity(id)` server action, call it on click, and revalidate the path.
2. Add a `deal_id` display to activities in the contact's feed — if the activity is linked to a deal, show the deal title next to the time-ago label.
3. Add an activity count badge to each deal card in the Kanban board. Fetch the count in the deals page server component with a separate Supabase query and pass it down to DealCard.

---

## Prompt / Template / Checklist pack

### Claude Code prompt 1 — Build the reusable ActivityLog and AddActivity components

```
I'm building ClearCRM with Next.js 14 App Router, Supabase, and Tailwind CSS.

The activities table exists in Supabase with these columns:
id (UUID), type (TEXT: note/call/email), description (TEXT), contact_id (UUID nullable FK → contacts), deal_id (UUID nullable FK → deals), created_at (TIMESTAMPTZ)

Build two reusable components:

COMPONENT 1: components/ActivityLog.tsx — SERVER COMPONENT (no 'use client')
- Props: contactId?: string, dealId?: string
- Fetch activities from Supabase using createServerComponentClient({ cookies })
- If contactId is provided, filter: .eq('contact_id', contactId)
- If dealId is provided, filter: .eq('deal_id', dealId)
- Order: created_at descending, limit 50
- Render a white card (rounded-2xl border border-gray-200 shadow-sm p-6) with heading "Activity Log"
- Render the AddActivity component at the top, passing the same props
- Below that, render the activity feed. Each item: a flex row with a 32px circle icon container on the left showing the type emoji (📝 note, 📞 call, 📧 email), and on the right a label ("Note"/"Call"/"Email"), a time-ago string, and the description text.
- Empty state: centered text "No activities yet. Add your first note, call, or email above."
- Time-ago function (no library): calculate seconds since created_at; return "just now" (<60s), "X minutes ago" (<3600s), "X hours ago" (<86400s), "X days ago" (>=86400s)

COMPONENT 2: components/AddActivity.tsx — CLIENT COMPONENT ('use client')
- Props: contactId?: string, dealId?: string
- Local state: selectedType ('note' | 'call' | 'email', default 'note')
- Render 3 pill toggle buttons (Note/Call/Email). Active type: bg-emerald-600 text-white. Inactive: white bg gray border.
- Textarea (rows=3) with placeholder that changes based on selectedType
- On submit: call addActivity server action from @/app/actions/activities, passing type, description, contact_id, deal_id via FormData. On success: reset the form (use useRef on the form element, call formRef.current?.reset()), call router.refresh(). On error: show error message in a red box.
- Submit button: "Log activity" / "Saving…" when pending. Emerald filled style.

SERVER ACTION: app/actions/activities.ts — 'use server'
- addActivity(formData: FormData): validate type is in ['note','call','email'], description is not empty, at least one of contact_id or deal_id is set. Insert into activities table. revalidatePath for contact or deal page. Return { error } on failure.

Tailwind only. No external UI libraries. Show complete file contents for all 3 files.
```

---

### Claude Code prompt 2 — Build the contact detail page

```
I'm building ClearCRM with Next.js 14 App Router, Supabase, and Tailwind CSS.

Build the contact detail page at app/contacts/[id]/page.tsx.

ROUTE: Dynamic route. Access the contact ID via params.id.

SUPABASE QUERY:
supabase
  .from('contacts')
  .select('*, companies(id, name, website)')
  .eq('id', params.id)
  .single()

If error or !contact, call notFound().

PAGE LAYOUT (server component, no 'use client'):
1. A "← Back to Contacts" link pointing to /contacts (text-sm text-gray-500, hover:text-emerald-700)
2. A white card (rounded-2xl border border-gray-200 shadow-sm p-6) containing:
   - A 56px avatar circle (bg-emerald-100 text-emerald-700) showing the contact's initials (first letter of first name + first letter of last name)
   - Contact full name as h1 (text-2xl font-bold text-gray-900)
   - Company name below (text-sm text-gray-500) — show only if companies join is not null
   - Email as a mailto: anchor link (text-sm text-gray-600 hover:text-emerald-700)
   - Phone as a tel: anchor link (text-sm text-gray-600 hover:text-emerald-700)
3. An ActivityLog component from @/components/ActivityLog, passing contactId={contact.id}

IMPORTANT: Do not add any features beyond this list. Show complete file contents.
```

---

### Activity types reference card

| Type | Icon | When to use | Example descriptions |
|------|------|-------------|---------------------|
| **Note** | 📝 | Any internal observation, reminder, or context you want to record | "Sarah mentioned they are reviewing 3 vendors. Decision by end of month." / "Budget approved internally, waiting for IT sign-off." / "Left a voicemail, follow up next Tuesday." |
| **Call** | 📞 | A phone or video call that happened — log what was discussed and agreed | "15-min discovery call. Confirmed budget of R50k. Agreed to send proposal by Friday." / "Follow-up call re: proposal. They want to reduce scope by 20%. Sending revised quote." |
| **Email** | 📧 | An email sent or received that is worth recording in the CRM | "Sent introduction email + capability deck." / "Received RFP document — see email thread." / "Proposal email sent with quote attached." |

**When to log vs when to skip.** Log every meaningful interaction. Skip generic automated emails (newsletters, system notifications) and internal admin tasks. If it contains information that would help a colleague pick up the deal tomorrow, log it.
