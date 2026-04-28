# Day 14: Dashboard

## Outcome artifact

The `/dashboard` page shows four stat cards (total contacts, total companies, pipeline value, deals closed this month), a deals-by-stage bar chart built with pure CSS (no chart library), and a recent activities feed showing the last 10 activities across all contacts and deals. All data is fetched server-side in parallel using `Promise.all`. The pipeline value excludes Won and Lost deals. Currency is formatted in South African Rand.

---

## The core idea

A dashboard is a single page that answers the most important questions in the shortest time: How many contacts do I have? What is the total value of open deals? Where are deals stuck? What happened recently?

Two principles make this dashboard fast and simple:

**Parallel data fetching.** Five Supabase queries run at the same time using `Promise.all`. Each query is independent — fetching contacts does not depend on fetching deals. Running them in parallel means the total wait is the duration of the slowest query, not the sum of all five.

**No chart library.** The deals-by-stage bar chart is a set of `<div>` elements with widths set as percentages. The widest bar (the stage with the most deals) gets `width: 100%`. Every other bar gets `(count / maxCount) * 100%`. No D3, no Recharts, no bundle size cost.

---

## Step-by-step walkthrough

### Step 1 — Set up the dashboard route

If your app currently redirects the root `/` to `/dashboard`, use `app/dashboard/page.tsx`. If `/` is the dashboard directly, use `app/page.tsx`. This walkthrough uses `app/dashboard/page.tsx`. Adjust the path if your project uses `app/page.tsx`.

```
Directory: ~/clearcrm
Command:   mkdir -p app/dashboard
```

---

### Step 2 — The parallel Supabase queries

Create `app/dashboard/page.tsx`. Here are all five queries, each with a comment explaining what it returns:

```tsx
// app/dashboard/page.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import StatCard from '@/components/StatCard'
import DealsByStageChart from '@/components/DealsByStageChart'
import RecentActivity from '@/components/RecentActivity'
import type { Database } from '@/types/database'

const STAGES = ['lead', 'qualified', 'proposal', 'negotiation', 'won', 'lost'] as const

export default async function DashboardPage() {
  const supabase = createServerComponentClient<Database>({ cookies })

  // Run all 5 queries in parallel — total wait = slowest query, not sum of all
  const [
    contactsResult,
    companiesResult,
    pipelineResult,
    stageCountsResult,
    recentActivitiesResult,
  ] = await Promise.all([
    // 1. Total contacts count
    supabase
      .from('contacts')
      .select('id', { count: 'exact', head: true }),

    // 2. Total companies count
    supabase
      .from('companies')
      .select('id', { count: 'exact', head: true }),

    // 3. Sum of all open deal values (exclude Won and Lost)
    supabase
      .from('deals')
      .select('value')
      .not('stage', 'in', '(won,lost)'),

    // 4. All deals with their stage — we group client-side to count per stage
    supabase
      .from('deals')
      .select('stage'),

    // 5. Last 10 activities across all contacts and deals, with contact name joined
    supabase
      .from('activities')
      .select('*, contacts(first_name, last_name)')
      .order('created_at', { ascending: false })
      .limit(10),
  ])

  // Extract values with safe fallbacks
  const totalContacts = contactsResult.count ?? 0
  const totalCompanies = companiesResult.count ?? 0

  // Calculate pipeline value (sum of open deals)
  const pipelineValue = (pipelineResult.data ?? []).reduce(
    (sum, deal) => sum + (deal.value ?? 0),
    0
  )

  // Count deals that closed as Won this calendar month
  const now = new Date()
  const firstOfMonth = new Date(now.getFullYear(), now.getMonth(), 1).toISOString()
  const { data: wonThisMonth } = await supabase
    .from('deals')
    .select('id', { count: 'exact', head: true })
    .eq('stage', 'won')
    .gte('created_at', firstOfMonth)

  const dealsClosedThisMonth = (wonThisMonth as any)?.count ?? 0

  // Count deals per stage
  const stageCounts: Record<string, number> = {}
  for (const stage of STAGES) stageCounts[stage] = 0
  for (const deal of (stageCountsResult.data ?? [])) {
    if (deal.stage in stageCounts) {
      stageCounts[deal.stage]++
    }
  }

  const recentActivities = (recentActivitiesResult.data ?? []) as any[]

  return (
    <div className="p-6 max-w-6xl mx-auto">
      <h1 className="text-2xl font-bold text-gray-900 mb-6">Dashboard</h1>

      {/* Stat cards */}
      <div className="grid grid-cols-2 lg:grid-cols-4 gap-4 mb-8">
        <StatCard
          label="Total Contacts"
          value={totalContacts.toLocaleString()}
        />
        <StatCard
          label="Total Companies"
          value={totalCompanies.toLocaleString()}
        />
        <StatCard
          label="Pipeline Value"
          value={new Intl.NumberFormat('en-ZA', {
            style: 'currency',
            currency: 'ZAR',
            maximumFractionDigits: 0,
          }).format(pipelineValue)}
        />
        <StatCard
          label="Won This Month"
          value={dealsClosedThisMonth.toLocaleString()}
          suffix="deals"
        />
      </div>

      <div className="grid grid-cols-1 lg:grid-cols-2 gap-6">
        {/* Deals by stage chart */}
        <DealsByStageChart stageCounts={stageCounts} />

        {/* Recent activity feed */}
        <RecentActivity activities={recentActivities} />
      </div>
    </div>
  )
}
```

---

### Step 3 — Create the StatCard component

Create `components/StatCard.tsx`:

```tsx
// components/StatCard.tsx
type Props = {
  label: string
  value: string
  suffix?: string
}

export default function StatCard({ label, value, suffix }: Props) {
  return (
    <div className="bg-white rounded-2xl border border-gray-200 shadow-sm p-5">
      <p className="text-xs font-semibold text-gray-500 uppercase tracking-wider">
        {label}
      </p>
      <div className="mt-2 flex items-end gap-1">
        <p className="text-3xl font-bold text-emerald-700 leading-none">
          {value}
        </p>
        {suffix && (
          <p className="text-sm text-gray-500 mb-0.5 leading-none">{suffix}</p>
        )}
      </div>
    </div>
  )
}
```

---

### Step 4 — Create the DealsByStageChart component

Create `components/DealsByStageChart.tsx`:

```tsx
// components/DealsByStageChart.tsx

const STAGE_LABELS: Record<string, string> = {
  lead: 'Lead',
  qualified: 'Qualified',
  proposal: 'Proposal',
  negotiation: 'Negotiation',
  won: 'Won',
  lost: 'Lost',
}

const STAGE_BAR_CLASS: Record<string, string> = {
  lead: 'bg-gray-400',
  qualified: 'bg-blue-400',
  proposal: 'bg-purple-400',
  negotiation: 'bg-amber-400',
  won: 'bg-emerald-500',
  lost: 'bg-red-400',
}

const STAGES = ['lead', 'qualified', 'proposal', 'negotiation', 'won', 'lost']

type Props = {
  stageCounts: Record<string, number>
}

export default function DealsByStageChart({ stageCounts }: Props) {
  const maxCount = Math.max(...Object.values(stageCounts), 1)

  return (
    <div className="bg-white rounded-2xl border border-gray-200 shadow-sm p-6">
      <h2 className="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-5">
        Deals by Stage
      </h2>

      <div className="space-y-3">
        {STAGES.map((stage) => {
          const count = stageCounts[stage] ?? 0
          const widthPercent = Math.round((count / maxCount) * 100)

          return (
            <div key={stage} className="flex items-center gap-3">
              {/* Stage label */}
              <div className="w-24 flex-shrink-0 text-right">
                <span className="text-xs font-medium text-gray-600">
                  {STAGE_LABELS[stage]}
                </span>
              </div>

              {/* Bar track */}
              <div className="flex-1 bg-gray-100 rounded-full h-2.5 overflow-hidden">
                <div
                  className={`h-full rounded-full transition-all duration-500 ${STAGE_BAR_CLASS[stage]}`}
                  style={{ width: count === 0 ? '0%' : `${Math.max(widthPercent, 4)}%` }}
                />
              </div>

              {/* Count */}
              <div className="w-6 flex-shrink-0 text-left">
                <span className="text-xs font-bold text-gray-700">{count}</span>
              </div>
            </div>
          )
        })}
      </div>

      {maxCount === 1 && Object.values(stageCounts).every((c) => c === 0) && (
        <p className="text-sm text-gray-400 text-center mt-4">
          No deals yet. Add deals in the pipeline.
        </p>
      )}
    </div>
  )
}
```

The bar width calculation: `(count / maxCount) * 100`. The stage with the most deals gets 100% width. All others scale proportionally. The `Math.max(widthPercent, 4)` ensures that a bar with a non-zero count never renders as completely invisible — it shows at minimum 4% width so you can see it exists.

---

### Step 5 — Create the RecentActivity component

Create `components/RecentActivity.tsx`:

```tsx
// components/RecentActivity.tsx

const TYPE_ICON: Record<string, string> = {
  note: '📝',
  call: '📞',
  email: '📧',
}

function timeAgo(date: string): string {
  const seconds = Math.floor((Date.now() - new Date(date).getTime()) / 1000)
  if (seconds < 60) return 'just now'
  if (seconds < 3600) return `${Math.floor(seconds / 60)}m ago`
  if (seconds < 86400) return `${Math.floor(seconds / 3600)}h ago`
  return `${Math.floor(seconds / 86400)}d ago`
}

type Activity = {
  id: string
  type: string
  description: string
  created_at: string
  contacts: {
    first_name: string
    last_name: string
  } | null
}

type Props = {
  activities: Activity[]
}

export default function RecentActivity({ activities }: Props) {
  return (
    <div className="bg-white rounded-2xl border border-gray-200 shadow-sm p-6">
      <h2 className="text-sm font-semibold text-gray-500 uppercase tracking-wider mb-4">
        Recent Activity
      </h2>

      {activities.length === 0 ? (
        <p className="text-sm text-gray-400 text-center py-6">
          No activities yet.
        </p>
      ) : (
        <div className="space-y-4">
          {activities.map((activity) => {
            const contactName = activity.contacts
              ? `${activity.contacts.first_name} ${activity.contacts.last_name}`
              : null

            return (
              <div key={activity.id} className="flex gap-3">
                {/* Icon */}
                <div className="w-8 h-8 rounded-full bg-gray-100 flex items-center justify-center flex-shrink-0 text-sm mt-0.5">
                  {TYPE_ICON[activity.type] ?? '📌'}
                </div>

                {/* Content */}
                <div className="flex-1 min-w-0">
                  <div className="flex items-center justify-between gap-2 mb-0.5">
                    {contactName && (
                      <span className="text-xs font-semibold text-gray-600 truncate">
                        {contactName}
                      </span>
                    )}
                    <span className="text-xs text-gray-400 flex-shrink-0">
                      {timeAgo(activity.created_at)}
                    </span>
                  </div>
                  <p className="text-sm text-gray-600 line-clamp-2 leading-relaxed">
                    {activity.description}
                  </p>
                </div>
              </div>
            )
          })}
        </div>
      )}
    </div>
  )
}
```

`RecentActivity` is a pure presentational component — it receives data as props and renders it. No Supabase call inside this component. The data fetch happens in the page server component and gets passed down. This makes the component easy to test and reuse anywhere.

---

### Step 6 — Format currency in South African Rand

The `Intl.NumberFormat` API handles all locale-specific currency formatting. No library needed:

```ts
// ZAR currency formatting — use this pattern everywhere in ClearCRM
function formatZAR(value: number): string {
  return new Intl.NumberFormat('en-ZA', {
    style: 'currency',
    currency: 'ZAR',
    maximumFractionDigits: 0,
  }).format(value)
}

// Examples:
// formatZAR(45000)    → "R 45 000"
// formatZAR(1250.50)  → "R 1 251"
// formatZAR(0)        → "R 0"
```

The locale `en-ZA` applies South African number formatting: spaces as thousand separators (`45 000`) and the `R` symbol. `maximumFractionDigits: 0` removes the cents for cleaner display on the dashboard.

---

### Step 7 — Wire the sidebar

Add a Dashboard link to your sidebar navigation:

```tsx
// In your sidebar navLinks array
const navLinks = [
  { href: '/dashboard', label: 'Dashboard', icon: '🏠' },
  { href: '/contacts', label: 'Contacts', icon: '👥' },
  { href: '/companies', label: 'Companies', icon: '🏢' },
  { href: '/deals', label: 'Deals', icon: '💼' },
]
```

If your root `/` redirects to `/dashboard`, confirm the redirect in `app/page.tsx`:

```tsx
// app/page.tsx — redirect root to dashboard
import { redirect } from 'next/navigation'

export default function RootPage() {
  redirect('/dashboard')
}
```

---

### Step 8 — Test all four stats

1. Run your dev server (`npm run dev`).

2. Navigate to `http://localhost:3000/dashboard`.

3. Verify each stat card shows the correct number:
   - **Total Contacts**: go to Supabase Table Editor → contacts → count the rows. Numbers must match.
   - **Total Companies**: same check on the companies table.
   - **Pipeline Value**: go to deals table, manually sum the `value` column for all rows NOT in `won` or `lost` stage. Must match.
   - **Won This Month**: count deals with `stage = 'won'` created this calendar month.

4. Add a new contact via the contacts page. Refresh the dashboard. The contacts count should increment by 1.

5. Add a deal and set it to `won`. Refresh the dashboard. The Won This Month count should increment.

---

### Step 9 — Commit

```
Directory: ~/clearcrm
Command:   git add . && git commit -m "feat: dashboard — stats, deals-by-stage chart, recent activity"
Expected:  [main xxxxxxx] feat: dashboard — stats, deals-by-stage chart, recent activity
```

---

## Practical workflow

**Why `Promise.all` and not sequential awaits.** Sequential awaits block — each query waits for the previous one to finish. Five queries taking 100ms each = 500ms total. `Promise.all` fires all five simultaneously. Total wait = 100ms (the slowest). On a dashboard that users open every morning, this difference is immediately felt.

**Why `{ count: 'exact', head: true }` for counts.** `count: 'exact'` tells Supabase to run a `COUNT(*)` and return it in the response header. `head: true` sends a HEAD request — no row data is returned at all, just the count. This is faster and uses less bandwidth than selecting all rows and calling `.length` on the result.

**Why the bar chart does not use a library.** Recharts, Chart.js, and similar libraries add 50–200KB to your bundle and require you to learn their API. The CSS bar chart you built today is 30 lines of JSX, fully controllable, and renders identically on every device. For a simple bar chart, a library is overkill.

**The `line-clamp-2` class.** This Tailwind class limits a text element to 2 lines and adds an ellipsis (`…`) when it overflows. It uses the CSS `-webkit-line-clamp` property. The activity feed descriptions vary in length — this keeps the feed layout consistent regardless of how long the description is.

---

## Common mistakes

**Mistake: `Promise.all` returns an error for one query and the whole dashboard crashes.**
Cause: One of the five queries fails (usually an RLS issue or missing table), and the error is not handled.
Fix: Destructure the result of each query with `const [{ data, error }] = await Promise.all([...])` and log errors individually. Add `?? []` or `?? 0` fallbacks for all values.

**Mistake: Pipeline value is zero even though open deals exist.**
Cause: The `.not('stage', 'in', '(won,lost)')` filter syntax is wrong. Supabase uses PostgREST syntax.
Fix: The correct syntax is exactly `.not('stage', 'in', '(won,lost)')` — including the parentheses inside the string. Alternatively use two `.neq()` calls: `.neq('stage', 'won').neq('stage', 'lost')`.

**Mistake: All bars in the chart are the same width.**
Cause: `maxCount` is being calculated incorrectly, or the bar width style is hardcoded.
Fix: `const maxCount = Math.max(...Object.values(stageCounts), 1)`. The `, 1` ensures the max is never 0 (which would cause division by zero). Then each bar: `style={{ width: (count / maxCount) * 100 + '%' }}`.

**Mistake: Recent activities show no contact name.**
Cause: The activities query includes `contacts(first_name, last_name)` but the activity was linked only to a deal (not a contact), so the join returns `null`.
Fix: This is correct behaviour. The contact name column should be optional. In `RecentActivity`, the `contactName &&` guard handles the null case — show nothing rather than "null null".

**Mistake: Won This Month count is always 0.**
Cause: The `firstOfMonth` date filter uses `created_at` but your deals close in a different column (`close_date`).
Fix: Decide which column you are filtering. If you want deals that were created as "Won" (stage set to won at creation), filter `created_at`. If you want deals whose close date falls in this month, filter `close_date`. Adjust the query accordingly.

---

## Your turn

1. Add a fifth stat card: "Open Deals" — the total number of deals not in Won or Lost stage.
2. Make the DealsByStageChart bars animate on load using CSS transitions (`transition-all duration-700`).
3. Add a "View all activities" link below the Recent Activity feed that points to a future `/activities` page.

---

## Prompt / Template / Checklist pack

### Claude Code prompt — Build the dashboard

```
I'm building ClearCRM with Next.js 14 App Router, Supabase, and Tailwind CSS.

Build a dashboard page at app/dashboard/page.tsx. All requirements are listed below. Do not add anything beyond this list.

DATA: Fetch all data server-side using Promise.all with these 5 Supabase queries:

1. Contact count:
   supabase.from('contacts').select('id', { count: 'exact', head: true })
   → use result.count

2. Company count:
   supabase.from('companies').select('id', { count: 'exact', head: true })
   → use result.count

3. Pipeline value (open deals only, excluding won and lost):
   supabase.from('deals').select('value').not('stage', 'in', '(won,lost)')
   → sum result.data[].value

4. Stage counts (count deals per stage):
   supabase.from('deals').select('stage')
   → group in JS: iterate over data, count per stage

5. Recent activities (last 10, all contacts):
   supabase.from('activities').select('*, contacts(first_name, last_name)').order('created_at', { ascending: false }).limit(10)

LAYOUT (page.tsx — server component):
- Heading "Dashboard" (text-2xl font-bold text-gray-900)
- 4 StatCard components in a responsive grid (2 cols on mobile, 4 on desktop):
  - "Total Contacts" / totalContacts
  - "Total Companies" / totalCompanies
  - "Pipeline Value" / formatted as ZAR using Intl.NumberFormat('en-ZA', { style: 'currency', currency: 'ZAR', maximumFractionDigits: 0 })
  - "Won This Month" / count of deals with stage='won' created this calendar month (run a 6th query for this), with suffix "deals"
- Below the stat cards: a 2-column grid on desktop (1 col on mobile) with DealsByStageChart on the left and RecentActivity on the right.

COMPONENT 1: components/StatCard.tsx
- Props: label (string), value (string), suffix? (string)
- White card (rounded-2xl border border-gray-200 shadow-sm p-5)
- Label: text-xs font-semibold text-gray-500 uppercase tracking-wider
- Value: text-3xl font-bold text-emerald-700

COMPONENT 2: components/DealsByStageChart.tsx
- Props: stageCounts (Record<string, number>)
- Stages in order: lead, qualified, proposal, negotiation, won, lost
- White card (rounded-2xl border border-gray-200 shadow-sm p-6), heading "Deals by Stage"
- Each stage: a row with stage label (right-aligned, w-24), a bar track (bg-gray-100 rounded-full h-2.5), and a count number
- Bar fill width: (count / maxCount) * 100%. maxCount = Math.max(...all counts, 1)
- Minimum bar width for non-zero counts: 4% (so small counts are still visible)
- Bar colours: lead=gray, qualified=blue, proposal=purple, negotiation=amber, won=emerald, lost=red

COMPONENT 3: components/RecentActivity.tsx
- Props: activities (array with id, type, description, created_at, contacts object)
- White card (rounded-2xl border border-gray-200 shadow-sm p-6), heading "Recent Activity"
- Each item: icon circle on left (📝📞📧), contact name + time-ago on right, description below
- Time-ago function (no library): <60s="just now", <3600="Xm ago", <86400="Xh ago", else="Xd ago"
- description: line-clamp-2 (shows max 2 lines)
- Empty state: "No activities yet."

Do not use any chart library, UI library, or date formatting library. Tailwind and Intl only. Show complete file contents for all 4 files.
```

---

### Dashboard data query reference

```ts
// 1. Total contacts count
// Returns: { count: number } — the total number of rows in the contacts table
const { count: totalContacts } = await supabase
  .from('contacts')
  .select('id', { count: 'exact', head: true })

// 2. Total companies count
// Returns: { count: number } — the total number of rows in the companies table
const { count: totalCompanies } = await supabase
  .from('companies')
  .select('id', { count: 'exact', head: true })

// 3. Pipeline value (open deals only)
// Returns: { data: Array<{ value: number }> } — one object per open deal
// Pipeline value = data.reduce((sum, d) => sum + d.value, 0)
const { data: openDeals } = await supabase
  .from('deals')
  .select('value')
  .not('stage', 'in', '(won,lost)')

// 4. Deals per stage
// Returns: { data: Array<{ stage: string }> } — one object per deal
// Group in JS: iterate over data and increment stageCounts[deal.stage]
const { data: allDeals } = await supabase
  .from('deals')
  .select('stage')

// 5. Recent activities with contact name
// Returns: { data: Array<Activity & { contacts: { first_name, last_name } | null }> }
// Shows last 10 activities across the entire CRM
const { data: recentActivities } = await supabase
  .from('activities')
  .select('*, contacts(first_name, last_name)')
  .order('created_at', { ascending: false })
  .limit(10)
```
