# Day 10: Search, Filter, Sort, and Pagination

## Outcome artifact
By the end of this day the `/contacts` page has a working search box that filters contacts by name or email, a company filter dropdown, a sort control, and pagination that shows 10 contacts per page — all state stored in the URL so filters are shareable and bookmarkable.

## The core idea

Storing filter state in URL search params (e.g. `/contacts?search=jane&company=abc123&sort=name&page=2`) gives you three things for free: shareability (copy the URL and the filters persist), browser history (the back button restores the previous filter state), and server-side rendering (the server reads the params and runs the filtered query directly, with no loading flash). This is the correct approach for a data-heavy page in Next.js — never store filter state only in React state if the page is server-rendered.

The filter controls live in a client component (`ContactsFilters`) because they respond to user input. When the user types in the search box or picks a company, the component updates the URL using `useRouter` from `next/navigation`. Updating the URL triggers Next.js to re-render the server component (`app/contacts/page.tsx`) with the new search params — which runs a fresh Supabase query. The cycle is: user types → URL updates → server re-renders → Supabase query runs with filters → filtered data returned.

Supabase provides three key methods for filtering: `.ilike()` for case-insensitive text search using SQL `ILIKE`, `.eq()` for exact value matching, and `.range()` for pagination offset. The `.or()` method lets you combine two `.ilike()` calls so a single search term matches against both first name and email.

## Step-by-step walkthrough

### Step 1 — Create the ContactsFilters client component

Create the file `app/contacts/ContactsFilters.tsx`. This component renders the search input, company dropdown, and sort select. Every change immediately updates the URL.

File path: `clearcrm/app/contacts/ContactsFilters.tsx`

```typescript
'use client'

import { useRouter, useSearchParams, usePathname } from 'next/navigation'
import { useCallback, useEffect, useState } from 'react'
import { createClient } from '@/utils/supabase/client'

type Company = { id: string; name: string }

export default function ContactsFilters() {
  const router = useRouter()
  const pathname = usePathname()
  const searchParams = useSearchParams()

  const [search, setSearch] = useState(searchParams.get('search') ?? '')
  const [companyId, setCompanyId] = useState(searchParams.get('company') ?? '')
  const [sort, setSort] = useState(searchParams.get('sort') ?? 'newest')
  const [companies, setCompanies] = useState<Company[]>([])

  // Fetch companies for the filter dropdown
  useEffect(() => {
    const supabase = createClient()
    supabase
      .from('companies')
      .select('id, name')
      .order('name')
      .then(({ data }) => setCompanies(data ?? []))
  }, [])

  // Build a new URL with updated params and push it
  const updateParams = useCallback(
    (updates: Record<string, string>) => {
      const params = new URLSearchParams(searchParams.toString())

      Object.entries(updates).forEach(([key, value]) => {
        if (value) {
          params.set(key, value)
        } else {
          params.delete(key)
        }
      })

      // Reset to page 1 whenever filters change
      params.delete('page')

      router.push(`${pathname}?${params.toString()}`)
    },
    [searchParams, pathname, router]
  )

  // Debounce the search input so we don't push a new URL on every keystroke
  useEffect(() => {
    const timer = setTimeout(() => {
      updateParams({ search })
    }, 300)
    return () => clearTimeout(timer)
  }, [search, updateParams])

  function handleCompanyChange(value: string) {
    setCompanyId(value)
    updateParams({ company: value })
  }

  function handleSortChange(value: string) {
    setSort(value)
    updateParams({ sort: value })
  }

  return (
    <div className="flex flex-wrap items-center gap-3 mb-5">
      {/* Search */}
      <div className="relative flex-1 min-w-[200px]">
        <svg
          className="absolute left-3 top-1/2 -translate-y-1/2 text-gray-400 w-4 h-4"
          fill="none"
          stroke="currentColor"
          viewBox="0 0 24 24"
        >
          <path
            strokeLinecap="round"
            strokeLinejoin="round"
            strokeWidth={2}
            d="M21 21l-6-6m2-5a7 7 0 11-14 0 7 7 0 0114 0z"
          />
        </svg>
        <input
          type="text"
          placeholder="Search by name or email…"
          value={search}
          onChange={(e) => setSearch(e.target.value)}
          className="w-full pl-9 pr-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
        />
      </div>

      {/* Company filter */}
      <select
        value={companyId}
        onChange={(e) => handleCompanyChange(e.target.value)}
        className="px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent bg-white min-w-[160px]"
      >
        <option value="">All companies</option>
        {companies.map((c) => (
          <option key={c.id} value={c.id}>
            {c.name}
          </option>
        ))}
      </select>

      {/* Sort */}
      <select
        value={sort}
        onChange={(e) => handleSortChange(e.target.value)}
        className="px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent bg-white"
      >
        <option value="newest">Newest first</option>
        <option value="oldest">Oldest first</option>
        <option value="name_asc">Name A–Z</option>
        <option value="name_desc">Name Z–A</option>
      </select>

      {/* Clear filters — only show if any filter is active */}
      {(search || companyId || sort !== 'newest') && (
        <button
          onClick={() => {
            setSearch('')
            setCompanyId('')
            setSort('newest')
            router.push(pathname)
          }}
          className="text-sm text-gray-500 hover:text-gray-700 underline"
        >
          Clear filters
        </button>
      )}
    </div>
  )
}
```

---

### Step 2 — Update the contacts page to read searchParams and run a filtered Supabase query

Replace the contents of `app/contacts/page.tsx` with this updated version. The page now reads search params, applies them to the Supabase query, and passes pagination data to both the table and the pagination component.

File path: `clearcrm/app/contacts/page.tsx`

```typescript
import { createClient } from '@/utils/supabase/server'
import ContactsTable from './ContactsTable'
import ContactsFilters from './ContactsFilters'
import Pagination from './Pagination'

const PAGE_SIZE = 10

type SearchParams = {
  search?: string
  company?: string
  sort?: string
  page?: string
}

export default async function ContactsPage({
  searchParams,
}: {
  searchParams: SearchParams
}) {
  const supabase = createClient()

  const search = searchParams.search ?? ''
  const companyId = searchParams.company ?? ''
  const sort = searchParams.sort ?? 'newest'
  const page = parseInt(searchParams.page ?? '1', 10)
  const offset = (page - 1) * PAGE_SIZE

  // Build the query dynamically
  let query = supabase
    .from('contacts')
    .select(
      `
      id,
      first_name,
      last_name,
      email,
      phone,
      created_at,
      companies ( id, name )
    `,
      { count: 'exact' }  // ask Supabase to return the total row count for pagination
    )

  // Search: match first_name OR last_name OR email (case-insensitive)
  if (search) {
    query = query.or(
      `first_name.ilike.%${search}%,last_name.ilike.%${search}%,email.ilike.%${search}%`
    )
  }

  // Company filter: exact match on company_id
  if (companyId) {
    query = query.eq('company_id', companyId)
  }

  // Sorting
  switch (sort) {
    case 'oldest':
      query = query.order('created_at', { ascending: true })
      break
    case 'name_asc':
      query = query.order('first_name', { ascending: true })
      break
    case 'name_desc':
      query = query.order('first_name', { ascending: false })
      break
    default:
      query = query.order('created_at', { ascending: false })
  }

  // Pagination: return only PAGE_SIZE rows starting at offset
  query = query.range(offset, offset + PAGE_SIZE - 1)

  const { data: contacts, error, count } = await query

  if (error) {
    return (
      <div className="p-8">
        <p className="text-red-600">Failed to load contacts: {error.message}</p>
      </div>
    )
  }

  const totalPages = Math.ceil((count ?? 0) / PAGE_SIZE)

  return (
    <div className="p-8">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">Contacts</h1>
          <p className="text-sm text-gray-500 mt-0.5">
            {count ?? 0} {search || companyId ? 'matching' : 'total'}
          </p>
        </div>
      </div>

      <ContactsFilters />
      <ContactsTable contacts={contacts ?? []} />
      <Pagination currentPage={page} totalPages={totalPages} />
    </div>
  )
}
```

---

### Step 3 — Create the Pagination client component

Create the file `app/contacts/Pagination.tsx`.

File path: `clearcrm/app/contacts/Pagination.tsx`

```typescript
'use client'

import { useRouter, useSearchParams, usePathname } from 'next/navigation'

export default function Pagination({
  currentPage,
  totalPages,
}: {
  currentPage: number
  totalPages: number
}) {
  const router = useRouter()
  const pathname = usePathname()
  const searchParams = useSearchParams()

  if (totalPages <= 1) return null

  function goToPage(page: number) {
    const params = new URLSearchParams(searchParams.toString())
    if (page === 1) {
      params.delete('page')
    } else {
      params.set('page', String(page))
    }
    router.push(`${pathname}?${params.toString()}`)
  }

  return (
    <div className="flex items-center justify-between mt-4">
      <p className="text-sm text-gray-500">
        Page {currentPage} of {totalPages}
      </p>
      <div className="flex items-center gap-2">
        <button
          onClick={() => goToPage(currentPage - 1)}
          disabled={currentPage <= 1}
          className="px-3 py-1.5 text-sm border border-gray-300 rounded-lg disabled:opacity-40 hover:bg-gray-50 transition-colors disabled:cursor-not-allowed"
        >
          Previous
        </button>

        {/* Page number buttons — show up to 5 pages */}
        {Array.from({ length: Math.min(totalPages, 5) }, (_, i) => {
          const pageNum = i + 1
          return (
            <button
              key={pageNum}
              onClick={() => goToPage(pageNum)}
              className={`px-3 py-1.5 text-sm border rounded-lg transition-colors ${
                pageNum === currentPage
                  ? 'bg-emerald-500 text-white border-emerald-500'
                  : 'border-gray-300 hover:bg-gray-50'
              }`}
            >
              {pageNum}
            </button>
          )
        })}

        <button
          onClick={() => goToPage(currentPage + 1)}
          disabled={currentPage >= totalPages}
          className="px-3 py-1.5 text-sm border border-gray-300 rounded-lg disabled:opacity-40 hover:bg-gray-50 transition-colors disabled:cursor-not-allowed"
        >
          Next
        </button>
      </div>
    </div>
  )
}
```

---

### Step 4 — Verify the Supabase query with all filters

The complete filtered query in context (for reference — this is what runs inside the page component):

```typescript
// Build the query
let query = supabase
  .from('contacts')
  .select(
    `id, first_name, last_name, email, phone, created_at, companies ( id, name )`,
    { count: 'exact' }
  )

// Text search — matches first_name, last_name, or email (case-insensitive)
if (search) {
  query = query.or(
    `first_name.ilike.%${search}%,last_name.ilike.%${search}%,email.ilike.%${search}%`
  )
}

// Filter by company — exact match on UUID
if (companyId) {
  query = query.eq('company_id', companyId)
}

// Sort — switch on the sort param value
query = query.order('created_at', { ascending: false })  // default: newest

// Pagination — return rows from offset to offset + 9 (10 rows per page)
query = query.range(offset, offset + PAGE_SIZE - 1)

const { data, count, error } = await query
// count is the total number of matching rows (before pagination)
// data is only the PAGE_SIZE rows for this page
```

---

### Step 5 — Test the full filter flow

1. Run `npm run dev` and open `http://localhost:3000/contacts`
2. Add at least 15 contacts through the add form (or insert them directly in the Supabase Table Editor) to test pagination
3. Search for a name — the URL should change to `/contacts?search=jane` and the table should update
4. Select a company in the company dropdown — the URL should become `/contacts?search=jane&company=<uuid>`
5. Change the sort to "Name A–Z" — URL becomes `/contacts?search=jane&company=<uuid>&sort=name_asc`
6. Navigate to page 2 — URL becomes `/contacts?search=jane&company=<uuid>&sort=name_asc&page=2`
7. Copy the full URL and paste it in a new tab — the same filters and page should load (this confirms URL-based state)
8. Click "Clear filters" — the URL resets to `/contacts`

---

### Step 6 — Commit

```
git add . && git commit -m "feat: contacts search, filter, sort and pagination"
```

Directory: `clearcrm/`

---

## Practical workflow

1. Create `app/contacts/ContactsFilters.tsx` — search input with 300ms debounce, company select, sort select, all updating URL params via `useRouter`
2. Update `app/contacts/page.tsx` — read `searchParams` prop, build Supabase query with conditional `.or()`, `.eq()`, `.order()`, `.range()`, pass `count` to Pagination
3. Create `app/contacts/Pagination.tsx` — previous/next buttons, page number buttons, reads current page from URL, updates URL on click
4. Add at least 15 test contacts to verify pagination
5. Test: search → filter → sort → paginate → copy URL and verify it restores state
6. Commit

---

## Common mistakes

**1. The search input causes a new URL push on every single keystroke, making the page feel laggy**

This is the missing debounce. The `ContactsFilters` component uses a `useEffect` with a 300ms `setTimeout` to batch rapid keystrokes into a single URL update. If you wire the search input directly to `updateParams` in the `onChange` handler without the effect, you push a URL on every character. Always debounce text inputs that trigger navigation. The `useEffect` + `clearTimeout` pattern in Step 1 handles this correctly.

**2. Pagination shows the wrong total or page 2 returns no results**

Two separate issues. First: `{ count: 'exact' }` must be passed as the second argument to `.select()` — without it, `count` is always `null` and `totalPages` computes as 0. Second: `.range(offset, offset + PAGE_SIZE - 1)` uses zero-based indexing — page 1 maps to range(0, 9), page 2 maps to range(10, 19). If you accidentally use `range(offset, offset + PAGE_SIZE)` instead of `range(offset, offset + PAGE_SIZE - 1)`, you get 11 rows per page instead of 10. Both ends of the range are inclusive in Supabase.

**3. Changing a filter does not reset to page 1 — you get an empty page**

When you are on page 3 and change the search term, the result set shrinks to fewer pages. Without resetting the page number, the URL still says `page=3` but the new result set has only 1 page — so Supabase returns zero rows. The `updateParams` function in `ContactsFilters` handles this with `params.delete('page')` before pushing the new URL. If you write a custom `updateParams` without this line, you will hit this bug every time.

---

## Your turn

Open the Supabase Table Editor and manually insert 12 contacts (you can copy-paste the same row multiple times and change just the name). Then go to `/contacts`. Verify:
- The total count shows 12+
- The table shows 10 rows on page 1
- Clicking "Next" takes you to page 2 with the remaining rows
- Searching for a name that matches 2 contacts shows only those 2 rows, with no pagination controls (since there is only 1 page)

Expected output: Page 1 shows exactly 10 rows. The pagination footer shows "Page 1 of 2". Page 2 shows the remaining rows. Searching collapses the results correctly.

**Failure state:** Search returns no results even though matching contacts exist.
Fix: The `.or()` filter syntax in Supabase requires the field names and ilike operator to be formatted exactly as `field_name.ilike.%value%` with no spaces. Check that your search string is not empty (a blank search with `ilike.%%` matches everything, which is correct — but if search is a single space, it will match nothing useful). Also verify that none of your contacts have a `null` first_name — a null field will not match an ilike. First name is required in the form, but rows inserted directly in Supabase may have nulls.

**Failure state:** Typing in the search box and then immediately clicking the company dropdown causes a race condition — one filter overwrites the other.
Fix: The debounce `useEffect` reads from `search` state and calls `updateParams({ search })`. But `updateParams` is built from `useSearchParams` at call time. If the company dropdown fires `updateParams({ company: value })` before the debounce timer fires, both updates read the same original `searchParams` and one overwrites the other. The fix is to wrap `updateParams` in `useCallback` with `searchParams` as a dependency (already done in the code above) — this ensures each call to `updateParams` sees the most current URL params. If you are seeing this bug, confirm your `useCallback` dependency array includes `searchParams`.

---

## Prompt / Template / Checklist pack

### Claude Code prompt — search, filter, sort, and pagination

```
In this Next.js 14 App Router project (clearcrm), add search, company filter, sort, and pagination to the contacts page. All filter state must be stored in URL search params (not React state alone).

Create app/contacts/ContactsFilters.tsx as a 'use client' component that:
- Has a text input for name/email search with 300ms debounce before pushing to URL
- Has a select dropdown for company filter (fetch companies from Supabase client on mount)
- Has a sort select with options: newest (created_at desc), oldest (created_at asc), name_asc (first_name asc), name_desc (first_name desc)
- Uses useRouter from next/navigation to push updated URL params on any filter change
- Resets the page param to 1 whenever a filter changes
- Shows a "Clear filters" button when any filter is active

Update app/contacts/page.tsx to:
- Accept a searchParams prop typed as { search?: string; company?: string; sort?: string; page?: string }
- Build a Supabase query that applies: .or() for search across first_name, last_name, email using ilike; .eq('company_id') for company filter; .order() based on sort param; .range() for pagination with PAGE_SIZE = 10
- Pass { count: 'exact' } to .select() to get the total row count
- Pass contacts, currentPage, and totalPages to child components

Create app/contacts/Pagination.tsx as a 'use client' component with:
- Previous and Next buttons that are disabled at boundaries
- Page number buttons for up to 5 pages
- Uses useRouter and useSearchParams to update the page param in the URL
- Returns null if totalPages <= 1

Use Tailwind CSS. Emerald-500 as accent color.
```

---

### Supabase filter query reference card

The complete Supabase JS query for the contacts page with all filters applied:

```typescript
const PAGE_SIZE = 10
const offset = (page - 1) * PAGE_SIZE

let query = supabase
  .from('contacts')
  .select(
    `id, first_name, last_name, email, phone, created_at, companies ( id, name )`,
    { count: 'exact' }   // required — returns total matching rows in count field
  )

// Text search: case-insensitive match on first_name, last_name, or email
// .or() takes a comma-separated string of filter expressions
if (search) {
  query = query.or(
    `first_name.ilike.%${search}%,last_name.ilike.%${search}%,email.ilike.%${search}%`
  )
}

// Company filter: exact UUID match
// Only applies when a company is selected (companyId is non-empty string)
if (companyId) {
  query = query.eq('company_id', companyId)
}

// Sorting: switch on the sort param value from URL
// Default is newest first (created_at descending)
switch (sort) {
  case 'oldest':
    query = query.order('created_at', { ascending: true })
    break
  case 'name_asc':
    query = query.order('first_name', { ascending: true })
    break
  case 'name_desc':
    query = query.order('first_name', { ascending: false })
    break
  default:
    query = query.order('created_at', { ascending: false })
}

// Pagination: .range(from, to) — both ends are inclusive
// Page 1: range(0, 9)   — rows 1–10
// Page 2: range(10, 19) — rows 11–20
query = query.range(offset, offset + PAGE_SIZE - 1)

const { data: contacts, count, error } = await query

// count = total matching rows (before pagination) — use for totalPages calculation
// data  = only the rows for this page (max PAGE_SIZE rows)
const totalPages = Math.ceil((count ?? 0) / PAGE_SIZE)
```
