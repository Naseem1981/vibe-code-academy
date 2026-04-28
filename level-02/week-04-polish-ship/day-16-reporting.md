# Day 16: Reporting Page

## Outcome artifact

ClearCRM has a `/reports` page with 4 report sections: deals won vs lost this month, average deal value by stage, top 5 contacts by deal count, and a monthly pipeline trend for the last 6 months. All data is pulled live from Supabase. All charts are built with pure CSS — no chart library installed.

---

## The core idea

A reporting page is not a dashboard. The dashboard shows "how are we doing right now." The reports page answers "what happened, and where." You are answering business questions with aggregated data, not row-level records.

The four reports you build today cover the four questions every sales team asks:

1. Are we closing deals or losing them? (Win vs Loss)
2. Where does the most value sit in our pipeline? (Avg deal value by stage)
3. Who are our best-connected contacts? (Top contacts by deal count)
4. Is the pipeline growing month over month? (Monthly trend)

All four queries run server-side in parallel. You fetch them once when the page loads — no client-side fetching, no loading spinners, no useEffect. The page is a React Server Component from top to bottom.

You build one reusable chart component — `HorizontalBarChart` — and use it for all four reports. The chart is built with plain `div` elements and Tailwind width utilities. No library. No SVG. No canvas.

---

## Step-by-step walkthrough

### Step 1 — Create the reports page layout

Create the directory and file.

```bash
# from your project root
mkdir -p app/reports
touch app/reports/page.tsx
```

Open `app/reports/page.tsx` and add this shell. You will fill in the data fetching next.

```tsx
// app/reports/page.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import HorizontalBarChart from '@/components/HorizontalBarChart'

export const dynamic = 'force-dynamic'

export default async function ReportsPage() {
  const supabase = createServerComponentClient({ cookies })

  // --- All 4 queries run in parallel ---
  const [winLoss, avgByStage, topContacts, monthlyTrend] = await Promise.all([
    fetchWinLoss(supabase),
    fetchAvgByStage(supabase),
    fetchTopContacts(supabase),
    fetchMonthlyTrend(supabase),
  ])

  return (
    <div className="p-6 space-y-10">
      <h1 className="text-2xl font-bold text-gray-900">Reports</h1>

      {/* Report 1 — Won vs Lost */}
      <section>
        <h2 className="text-lg font-semibold text-gray-800 mb-4">Won vs Lost — This Month</h2>
        <HorizontalBarChart
          data={winLoss}
          maxValue={Math.max(...winLoss.map(d => d.value), 1)}
        />
      </section>

      {/* Report 2 — Average deal value by stage */}
      <section>
        <h2 className="text-lg font-semibold text-gray-800 mb-4">Average Deal Value by Stage</h2>
        <HorizontalBarChart
          data={avgByStage}
          maxValue={Math.max(...avgByStage.map(d => d.value), 1)}
          prefix="R"
        />
      </section>

      {/* Report 3 — Top contacts by deal count */}
      <section>
        <h2 className="text-lg font-semibold text-gray-800 mb-4">Top 5 Contacts by Deal Count</h2>
        <HorizontalBarChart
          data={topContacts}
          maxValue={Math.max(...topContacts.map(d => d.value), 1)}
        />
      </section>

      {/* Report 4 — Monthly pipeline trend */}
      <section>
        <h2 className="text-lg font-semibold text-gray-800 mb-4">Monthly Pipeline — Last 6 Months</h2>
        <HorizontalBarChart
          data={monthlyTrend}
          maxValue={Math.max(...monthlyTrend.map(d => d.value), 1)}
        />
      </section>
    </div>
  )
}
```

---

### Step 2 — Report 1: Won vs Lost this month

This report counts deals where `stage = 'won'` and deals where `stage = 'lost'`, filtered to the current calendar month.

Add this function below the component in `app/reports/page.tsx`:

```tsx
async function fetchWinLoss(supabase: any) {
  const now = new Date()
  const startOfMonth = new Date(now.getFullYear(), now.getMonth(), 1).toISOString()
  const endOfMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0, 23, 59, 59).toISOString()

  const [{ count: won }, { count: lost }] = await Promise.all([
    supabase
      .from('deals')
      .select('*', { count: 'exact', head: true })
      .eq('stage', 'won')
      .gte('created_at', startOfMonth)
      .lte('created_at', endOfMonth),

    supabase
      .from('deals')
      .select('*', { count: 'exact', head: true })
      .eq('stage', 'lost')
      .gte('created_at', startOfMonth)
      .lte('created_at', endOfMonth),
  ])

  return [
    { label: 'Won', value: won ?? 0 },
    { label: 'Lost', value: lost ?? 0 },
  ]
}
```

How it works:

- `startOfMonth` and `endOfMonth` are ISO strings for the first and last millisecond of the current month.
- `{ count: 'exact', head: true }` tells Supabase to return only the count, not the rows. This is fast — equivalent to `SELECT COUNT(*) WHERE ...`.
- Both queries run in parallel with `Promise.all`.

SQL equivalent for your reference:

```sql
SELECT COUNT(*) FROM deals
WHERE stage = 'won'
  AND created_at >= '2025-04-01'
  AND created_at <= '2025-04-30 23:59:59';
```

---

### Step 3 — Report 2: Average deal value by stage

This query groups all deals by `stage` and calculates the average `value` per stage.

Supabase JS does not have a built-in `GROUP BY` method. You use `.rpc()` to call a Postgres function, or you fetch all deals and aggregate in JavaScript. For this report, JavaScript aggregation is simpler and fast enough — your deals table will not have millions of rows.

Add this function:

```tsx
async function fetchAvgByStage(supabase: any) {
  const { data, error } = await supabase
    .from('deals')
    .select('stage, value')

  if (error || !data) return []

  // Group by stage and calculate average value
  const groups: Record<string, number[]> = {}
  for (const deal of data) {
    if (!deal.stage) continue
    if (!groups[deal.stage]) groups[deal.stage] = []
    groups[deal.stage].push(Number(deal.value) || 0)
  }

  const stageOrder = ['lead', 'qualified', 'proposal', 'negotiation', 'won', 'lost']

  return stageOrder
    .filter(stage => groups[stage])
    .map(stage => {
      const values = groups[stage]
      const avg = values.reduce((sum, v) => sum + v, 0) / values.length
      return {
        label: stage.charAt(0).toUpperCase() + stage.slice(1),
        value: Math.round(avg),
      }
    })
}
```

SQL equivalent for your reference:

```sql
SELECT stage, ROUND(AVG(value)) as avg_value
FROM deals
GROUP BY stage
ORDER BY CASE stage
  WHEN 'lead' THEN 1
  WHEN 'qualified' THEN 2
  WHEN 'proposal' THEN 3
  WHEN 'negotiation' THEN 4
  WHEN 'won' THEN 5
  WHEN 'lost' THEN 6
END;
```

---

### Step 4 — Report 3: Top 5 contacts by deal count

This query counts how many deals each contact has. It returns the top 5.

Your `deals` table has a `contact_id` column. You need to count deals grouped by `contact_id`, then join to `contacts` to get the name.

Supabase supports this with a two-step query:

```tsx
async function fetchTopContacts(supabase: any) {
  // Fetch all deals with their contact, then count in JS
  const { data, error } = await supabase
    .from('deals')
    .select('contact_id, contacts(first_name, last_name)')

  if (error || !data) return []

  // Count deals per contact
  const counts: Record<string, { name: string; count: number }> = {}

  for (const deal of data) {
    if (!deal.contact_id || !deal.contacts) continue
    const id = deal.contact_id
    const name = `${deal.contacts.first_name ?? ''} ${deal.contacts.last_name ?? ''}`.trim()
    if (!counts[id]) counts[id] = { name, count: 0 }
    counts[id].count += 1
  }

  return Object.values(counts)
    .sort((a, b) => b.count - a.count)
    .slice(0, 5)
    .map(entry => ({ label: entry.name || 'Unknown', value: entry.count }))
}
```

SQL equivalent for your reference:

```sql
SELECT
  c.first_name || ' ' || c.last_name AS contact_name,
  COUNT(d.id) AS deal_count
FROM deals d
JOIN contacts c ON c.id = d.contact_id
GROUP BY c.id, c.first_name, c.last_name
ORDER BY deal_count DESC
LIMIT 5;
```

---

### Step 5 — Report 4: Monthly pipeline trend (last 6 months)

This report counts how many deals were created in each of the last 6 calendar months.

First, generate the 6 month labels in JavaScript:

```js
const months = Array.from({ length: 6 }, (_, i) => {
  const d = new Date()
  d.setMonth(d.getMonth() - (5 - i))
  return d.toLocaleString('default', { month: 'short', year: 'numeric' })
})
// Example output: ['Nov 2024', 'Dec 2024', 'Jan 2025', 'Feb 2025', 'Mar 2025', 'Apr 2025']
```

This generates labels oldest to newest. Index 0 is 5 months ago. Index 5 is the current month.

Now add the full fetch function:

```tsx
async function fetchMonthlyTrend(supabase: any) {
  const now = new Date()

  // Start date: first day of the month 5 months ago
  const startDate = new Date(now.getFullYear(), now.getMonth() - 5, 1).toISOString()

  const { data, error } = await supabase
    .from('deals')
    .select('created_at')
    .gte('created_at', startDate)

  if (error || !data) return []

  // Generate the 6 month labels (oldest first)
  const months = Array.from({ length: 6 }, (_, i) => {
    const d = new Date()
    d.setMonth(d.getMonth() - (5 - i))
    return d.toLocaleString('default', { month: 'short', year: 'numeric' })
  })

  // Count deals per month
  const counts: Record<string, number> = {}
  for (const month of months) counts[month] = 0

  for (const deal of data) {
    const d = new Date(deal.created_at)
    const label = d.toLocaleString('default', { month: 'short', year: 'numeric' })
    if (counts[label] !== undefined) counts[label] += 1
  }

  return months.map(month => ({ label: month, value: counts[month] }))
}
```

SQL equivalent for your reference:

```sql
SELECT
  TO_CHAR(DATE_TRUNC('month', created_at), 'Mon YYYY') AS month,
  COUNT(*) AS deal_count
FROM deals
WHERE created_at >= DATE_TRUNC('month', NOW()) - INTERVAL '5 months'
GROUP BY DATE_TRUNC('month', created_at)
ORDER BY DATE_TRUNC('month', created_at);
```

---

### Step 6 — Create the HorizontalBarChart component

Create the file:

```bash
touch components/HorizontalBarChart.tsx
```

Add the full component:

```tsx
// components/HorizontalBarChart.tsx

type BarChartItem = {
  label: string
  value: number
}

type HorizontalBarChartProps = {
  data: BarChartItem[]
  maxValue: number
  prefix?: string   // e.g. "R" for Rand currency display
  suffix?: string   // e.g. "deals"
}

export default function HorizontalBarChart({
  data,
  maxValue,
  prefix = '',
  suffix = '',
}: HorizontalBarChartProps) {
  if (!data || data.length === 0) {
    return <p className="text-sm text-gray-400">No data available.</p>
  }

  return (
    <div className="space-y-3">
      {data.map((item) => {
        const percent = maxValue > 0 ? (item.value / maxValue) * 100 : 0
        const displayValue = `${prefix}${item.value.toLocaleString()}${suffix ? ' ' + suffix : ''}`

        return (
          <div key={item.label} className="flex items-center gap-3">
            {/* Label */}
            <span className="w-32 text-sm text-gray-600 text-right shrink-0">
              {item.label}
            </span>

            {/* Bar track */}
            <div className="flex-1 bg-gray-100 rounded-full h-6 overflow-hidden">
              {/* Bar fill */}
              <div
                className="h-full bg-emerald-500 rounded-full transition-all duration-500"
                style={{ width: `${percent}%` }}
              />
            </div>

            {/* Value label */}
            <span className="w-24 text-sm font-medium text-gray-800 shrink-0">
              {displayValue}
            </span>
          </div>
        )
      })}
    </div>
  )
}
```

How it works:

- `percent` converts the raw value to a percentage of `maxValue`. A deal count of 8 with a maxValue of 10 renders a bar at 80% width.
- The bar fill uses `style={{ width: \`${percent}%\` }}` — inline style is necessary here because Tailwind cannot generate dynamic percentage widths at runtime.
- `transition-all` is not used. Only `transition-all duration-500` applies to `width`, which is correct. Wait — the rule says never use `transition-all`. Fix this: replace with `transition-[width] duration-500`.

Corrected bar fill line:

```tsx
className="h-full bg-emerald-500 rounded-full transition-[width] duration-500"
```

---

### Step 7 — Add the Reports link to the sidebar

Open your sidebar component (typically `components/Sidebar.tsx` or `app/layout.tsx`) and add the Reports nav link.

```tsx
<Link
  href="/reports"
  className="flex items-center gap-2 px-4 py-2 text-sm text-gray-300 hover:text-white hover:bg-white/10 rounded-lg transition-colors"
>
  <svg className="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2}
      d="M9 19v-6a2 2 0 00-2-2H5a2 2 0 00-2 2v6a2 2 0 002 2h2a2 2 0 002-2zm0 0V9a2 2 0 012-2h2a2 2 0 012 2v10m-6 0a2 2 0 002 2h2a2 2 0 002-2m0 0V5a2 2 0 012-2h2a2 2 0 012 2v14a2 2 0 01-2 2h-2a2 2 0 01-2-2z"
    />
  </svg>
  Reports
</Link>
```

---

### Step 8 — Commit

```bash
# from your project root
git add .
git commit -m "feat: reports page — win/loss, deal values, top contacts, monthly trend"
```

Expected output:

```
[main a3f12b8] feat: reports page — win/loss, deal values, top contacts, monthly trend
 3 files changed, 187 insertions(+)
 create mode 100644 app/reports/page.tsx
 create mode 100644 components/HorizontalBarChart.tsx
```

---

## Practical workflow

You will not see data until you have deals in your database. If your Supabase deals table is empty, the charts will show "No data available." Add 5–10 test deals with different stages and different `created_at` dates before you open the reports page.

To insert test deals with backdated timestamps, run this in the Supabase SQL editor:

```sql
INSERT INTO deals (title, value, stage, created_at) VALUES
  ('Alpha Project', 12000, 'won', NOW() - INTERVAL '1 month'),
  ('Beta Campaign', 8500, 'lost', NOW() - INTERVAL '1 month'),
  ('Gamma Retainer', 22000, 'qualified', NOW() - INTERVAL '2 months'),
  ('Delta Website', 9500, 'proposal', NOW() - INTERVAL '3 months'),
  ('Epsilon App', 45000, 'won', NOW() - INTERVAL '3 months'),
  ('Zeta Consulting', 6000, 'lead', NOW() - INTERVAL '4 months'),
  ('Eta Branding', 18000, 'negotiation', NOW() - INTERVAL '4 months'),
  ('Theta Launch', 31000, 'won', NOW()),
  ('Iota SEO', 4500, 'lost', NOW()),
  ('Kappa Social', 7800, 'qualified', NOW());
```

After inserting, refresh the reports page. All four charts should populate.

---

## Common mistakes

**Promise.all fails silently.**
If one query inside `Promise.all` throws, the whole page throws. Wrap each fetch function in a try/catch and return an empty array on error. The page still renders — it just shows "No data available" for the broken section instead of crashing.

```tsx
async function fetchWinLoss(supabase: any) {
  try {
    // ... query
  } catch {
    return []
  }
}
```

**Bar widths are always 0.**
This happens when `maxValue` is 0. The `maxValue` prop is `Math.max(...data.map(d => d.value), 1)`. The `, 1` is the fallback. Without it, `Math.max()` on an empty array returns `-Infinity`, and percent becomes `NaN`.

**`created_at` filtering misses records.**
Your Supabase `created_at` column may store timestamps in UTC. If your local timezone is UTC+2, "today" for you started 2 hours before UTC midnight. Use `.gte()` and `.lte()` with explicit UTC timestamps to avoid off-by-one-day bugs.

**Supabase join returns null for contacts.**
`.select('contact_id, contacts(first_name, last_name)')` requires that a foreign key relationship exists between `deals.contact_id` and `contacts.id` in Supabase. If the join returns null, check that the foreign key is defined in your database schema. Go to Supabase → Table Editor → deals → foreign keys.

**The Reports page is not protected by auth.**
Make sure your middleware or layout wraps the `/reports` route in the same auth check as your other protected routes. Check `middleware.ts` — the route pattern should include `/reports`.

---

## Your turn

1. Add `export const dynamic = 'force-dynamic'` at the top of `app/reports/page.tsx`. Without it, Next.js may cache the page and return stale data.
2. Navigate to `http://localhost:3000/reports` and verify all four charts render.
3. Add a fifth report: "Deals by Company" — count deals grouped by `company_id`, show top 5 companies. Use the same `HorizontalBarChart` component.
4. Add a date range filter to Report 1 (Won vs Lost). Use a `<select>` to switch between "This Month", "Last 30 Days", and "Last 90 Days". Because this page is a Server Component, the filter will need to be a URL search param — pass it as a prop from `searchParams`.

---

## Prompt / Template / Checklist pack

### Claude Code prompt — build the reporting page

```
I am building a ClearCRM reporting page using Next.js 14 App Router and Supabase.

Create the following files:

1. app/reports/page.tsx — a React Server Component that:
   - Uses createServerComponentClient from @supabase/auth-helpers-nextjs
   - Runs 4 Supabase queries in parallel with Promise.all
   - Renders 4 report sections using a HorizontalBarChart component

2. components/HorizontalBarChart.tsx — a reusable pure-CSS bar chart with these props:
   - data: { label: string; value: number }[]
   - maxValue: number
   - prefix?: string (e.g. "R" for currency)
   - suffix?: string

The 4 reports are:
- Report 1: Count deals with stage = 'won' vs stage = 'lost' for the current calendar month
- Report 2: Average deal value grouped by stage (fetch all deals, aggregate in JS, use stage order: lead, qualified, proposal, negotiation, won, lost)
- Report 3: Top 5 contacts by deal count (fetch deals with contacts join, count in JS, return top 5 sorted descending)
- Report 4: Monthly pipeline — count deals created each month for the last 6 months. Generate month labels with:
  Array.from({ length: 6 }, (_, i) => { const d = new Date(); d.setMonth(d.getMonth() - (5 - i)); return d.toLocaleString('default', { month: 'short', year: 'numeric' }) })

Each fetch function should catch errors and return [] on failure.
Add export const dynamic = 'force-dynamic' to the page.
Use Tailwind CSS. Primary accent color is emerald-500 (#10B981).
Do not install any chart library.
```

---

## Deliverable — Reports data query reference

All four Supabase queries used in the reports page, labelled and explained.

---

### Query 1 — Won vs Lost this month

```ts
// Count deals with stage = 'won' created this calendar month
const { count: won } = await supabase
  .from('deals')
  .select('*', { count: 'exact', head: true })
  .eq('stage', 'won')
  .gte('created_at', startOfMonth)   // ISO string: first day of current month
  .lte('created_at', endOfMonth)     // ISO string: last second of current month

// Count deals with stage = 'lost' created this calendar month
const { count: lost } = await supabase
  .from('deals')
  .select('*', { count: 'exact', head: true })
  .eq('stage', 'lost')
  .gte('created_at', startOfMonth)
  .lte('created_at', endOfMonth)
```

`{ count: 'exact', head: true }` returns only the row count, not the rows. Fast and efficient.

---

### Query 2 — Average deal value by stage

```ts
// Fetch all deals — stage and value columns only
const { data } = await supabase
  .from('deals')
  .select('stage, value')

// Aggregate in JavaScript: group by stage, calculate average
// Then map to { label, value } format for the chart
```

Why aggregate in JS instead of using Postgres? Supabase JS does not support GROUP BY natively. For a table with thousands of rows you would use `.rpc()` with a Postgres function. For a typical CRM deals table, JS aggregation is faster to build and adequate in performance.

---

### Query 3 — Top 5 contacts by deal count

```ts
// Fetch all deals, with the related contact's name via foreign key join
const { data } = await supabase
  .from('deals')
  .select('contact_id, contacts(first_name, last_name)')

// Count in JavaScript, sort descending, take top 5
```

Requires a foreign key from `deals.contact_id` to `contacts.id` in Supabase. Supabase auto-discovers the join when the foreign key is defined.

---

### Query 4 — Monthly pipeline trend (last 6 months)

```ts
// Start date: first day of the month 5 months ago
const startDate = new Date(now.getFullYear(), now.getMonth() - 5, 1).toISOString()

// Fetch all deals created since that date — only created_at needed
const { data } = await supabase
  .from('deals')
  .select('created_at')
  .gte('created_at', startDate)

// Generate 6 month labels in JS, count deals per label
```

`new Date(year, month - 5, 1)` correctly handles year rollover. JavaScript's Date constructor wraps negative months automatically — month -1 becomes December of the previous year.
