# Day 9: Companies Module — CRUD and Contact Relationship

## Outcome artifact
By the end of this day ClearCRM has a `/companies` page that lists all companies with a count of linked contacts, an add/edit/delete flow for companies, and the Contact form has a Company dropdown that correctly links contacts to companies.

## The core idea

The Companies module introduces a one-to-many relationship: one company can have many contacts. In the database, this is expressed as a foreign key — the `contacts` table has a `company_id` column that points to a row in the `companies` table. You set this up in Day 6. Today you build the UI on top of it.

When you display a list of companies, you want to show how many contacts belong to each one. In a traditional setup you would run two queries and join the results in JavaScript. With Supabase you can express this in a single select using nested count syntax, which Supabase translates to a single SQL query with an aggregation. This is more efficient and less code.

When you update the Contact form to include the Company dropdown, you are establishing the link between the two modules. The form fetches the companies list from Supabase and presents it as a `<select>` element. When the user saves the contact, the selected company's UUID is written to `contact.company_id`. Deleting a company does not delete its contacts — the `ON DELETE SET NULL` constraint you added in Day 6 means the contact's `company_id` becomes NULL, making it "unlinked" but preserved.

## Step-by-step walkthrough

### Step 1 — Create the companies page (server component)

Create the file `app/companies/page.tsx`. This fetches all companies and for each one counts how many contacts are linked.

File path: `clearcrm/app/companies/page.tsx`

```typescript
import { createClient } from '@/utils/supabase/server'
import CompaniesTable from './CompaniesTable'

export default async function CompaniesPage() {
  const supabase = createClient()

  // Fetch companies and count of linked contacts in one query
  const { data: companies, error } = await supabase
    .from('companies')
    .select(`
      id,
      name,
      website,
      industry,
      created_at,
      contacts ( count )
    `)
    .order('name')

  if (error) {
    return (
      <div className="p-8">
        <p className="text-red-600">Failed to load companies: {error.message}</p>
      </div>
    )
  }

  // Normalize the nested count — Supabase returns [{ count: N }]
  const companiesWithCount = (companies ?? []).map((company) => ({
    ...company,
    contact_count: (company.contacts as unknown as { count: number }[])[0]?.count ?? 0,
  }))

  return (
    <div className="p-8">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">Companies</h1>
          <p className="text-sm text-gray-500 mt-0.5">{companiesWithCount.length} total</p>
        </div>
      </div>
      <CompaniesTable companies={companiesWithCount} />
    </div>
  )
}
```

---

### Step 2 — Create the companies table (client component)

Create the file `app/companies/CompaniesTable.tsx`. It manages the drawer state and renders the table with edit/delete actions.

File path: `clearcrm/app/companies/CompaniesTable.tsx`

```typescript
'use client'

import { useState } from 'react'
import { useRouter } from 'next/navigation'
import CompanyForm from './CompanyForm'
import { deleteCompany } from './actions'

type Company = {
  id: string
  name: string
  website: string | null
  industry: string | null
  created_at: string
  contact_count: number
}

export default function CompaniesTable({ companies }: { companies: Company[] }) {
  const router = useRouter()
  const [drawerOpen, setDrawerOpen] = useState(false)
  const [editingCompany, setEditingCompany] = useState<Company | null>(null)
  const [deletingId, setDeletingId] = useState<string | null>(null)
  const [showDeleteConfirm, setShowDeleteConfirm] = useState<string | null>(null)

  function openAddForm() {
    setEditingCompany(null)
    setDrawerOpen(true)
  }

  function openEditForm(company: Company) {
    setEditingCompany(company)
    setDrawerOpen(true)
  }

  function closeDrawer() {
    setDrawerOpen(false)
    setEditingCompany(null)
  }

  async function handleDelete(id: string) {
    setDeletingId(id)
    await deleteCompany(id)
    setDeletingId(null)
    setShowDeleteConfirm(null)
    router.refresh()
  }

  return (
    <>
      <div className="flex justify-end mb-4">
        <button
          onClick={openAddForm}
          className="bg-emerald-500 hover:bg-emerald-600 text-white text-sm font-medium px-4 py-2 rounded-lg transition-colors"
        >
          + Add Company
        </button>
      </div>

      {companies.length === 0 ? (
        <div className="text-center py-16 text-gray-400">
          <p className="text-lg font-medium">No companies yet</p>
          <p className="text-sm mt-1">Click "Add Company" to create your first one.</p>
        </div>
      ) : (
        <div className="bg-white border border-gray-200 rounded-xl overflow-hidden">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b border-gray-100 bg-gray-50">
                <th className="text-left px-4 py-3 font-medium text-gray-600">Company</th>
                <th className="text-left px-4 py-3 font-medium text-gray-600">Website</th>
                <th className="text-left px-4 py-3 font-medium text-gray-600">Industry</th>
                <th className="text-left px-4 py-3 font-medium text-gray-600">Contacts</th>
                <th className="px-4 py-3"></th>
              </tr>
            </thead>
            <tbody>
              {companies.map((company) => (
                <tr
                  key={company.id}
                  className="border-b border-gray-50 hover:bg-gray-50 transition-colors"
                >
                  <td className="px-4 py-3 font-medium text-gray-900">{company.name}</td>
                  <td className="px-4 py-3 text-gray-600">
                    {company.website ? (
                      <a
                        href={company.website}
                        target="_blank"
                        rel="noopener noreferrer"
                        className="text-emerald-600 hover:underline"
                      >
                        {company.website.replace(/^https?:\/\//, '')}
                      </a>
                    ) : (
                      '—'
                    )}
                  </td>
                  <td className="px-4 py-3 text-gray-600">{company.industry ?? '—'}</td>
                  <td className="px-4 py-3">
                    <span className="inline-flex items-center justify-center px-2 py-0.5 rounded-full text-xs font-medium bg-emerald-50 text-emerald-700">
                      {company.contact_count}
                    </span>
                  </td>
                  <td className="px-4 py-3">
                    <div className="flex items-center gap-2 justify-end">
                      <button
                        onClick={() => openEditForm(company)}
                        className="text-gray-400 hover:text-gray-700 text-xs font-medium px-2 py-1 rounded hover:bg-gray-100 transition-colors"
                      >
                        Edit
                      </button>
                      {showDeleteConfirm === company.id ? (
                        <div className="flex items-center gap-1">
                          <button
                            onClick={() => handleDelete(company.id)}
                            disabled={deletingId === company.id}
                            className="text-xs font-medium text-red-600 hover:text-red-700 px-2 py-1 rounded hover:bg-red-50 transition-colors disabled:opacity-50"
                          >
                            {deletingId === company.id ? 'Deleting…' : 'Confirm'}
                          </button>
                          <button
                            onClick={() => setShowDeleteConfirm(null)}
                            className="text-xs text-gray-400 hover:text-gray-600 px-2 py-1 rounded hover:bg-gray-100 transition-colors"
                          >
                            Cancel
                          </button>
                        </div>
                      ) : (
                        <button
                          onClick={() => setShowDeleteConfirm(company.id)}
                          className="text-gray-400 hover:text-red-500 text-xs font-medium px-2 py-1 rounded hover:bg-red-50 transition-colors"
                        >
                          Delete
                        </button>
                      )}
                    </div>
                  </td>
                </tr>
              ))}
            </tbody>
          </table>
        </div>
      )}

      {drawerOpen && (
        <>
          <div className="fixed inset-0 bg-black/30 z-40" onClick={closeDrawer} />
          <div className="fixed right-0 top-0 h-full w-full max-w-md bg-white shadow-xl z-50 flex flex-col">
            <div className="flex items-center justify-between px-6 py-4 border-b border-gray-100">
              <h2 className="text-lg font-semibold text-gray-900">
                {editingCompany ? 'Edit Company' : 'Add Company'}
              </h2>
              <button
                onClick={closeDrawer}
                className="text-gray-400 hover:text-gray-600 text-xl leading-none"
              >
                ×
              </button>
            </div>
            <div className="flex-1 overflow-y-auto p-6">
              <CompanyForm
                company={editingCompany}
                onSuccess={() => {
                  closeDrawer()
                  router.refresh()
                }}
              />
            </div>
          </div>
        </>
      )}
    </>
  )
}
```

---

### Step 3 — Create the company form (client component)

Create the file `app/companies/CompanyForm.tsx`. The industry field is a fixed dropdown — no need to store industries in the database.

File path: `clearcrm/app/companies/CompanyForm.tsx`

```typescript
'use client'

import { useState } from 'react'
import { addCompany, updateCompany } from './actions'

const INDUSTRIES = ['Technology', 'Finance', 'Healthcare', 'Retail', 'Other']

type Company = {
  id: string
  name: string
  website: string | null
  industry: string | null
}

export default function CompanyForm({
  company,
  onSuccess,
}: {
  company: Company | null
  onSuccess: () => void
}) {
  const [name, setName] = useState(company?.name ?? '')
  const [website, setWebsite] = useState(company?.website ?? '')
  const [industry, setIndustry] = useState(company?.industry ?? '')
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setLoading(true)
    setError(null)

    const payload = {
      name,
      website: website || null,
      industry: industry || null,
    }

    const result = company
      ? await updateCompany(company.id, payload)
      : await addCompany(payload)

    if (result?.error) {
      setError(result.error)
      setLoading(false)
      return
    }

    onSuccess()
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">
          Company name <span className="text-red-500">*</span>
        </label>
        <input
          type="text"
          required
          value={name}
          onChange={(e) => setName(e.target.value)}
          className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
          placeholder="Acme Corp"
        />
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">Website</label>
        <input
          type="url"
          value={website}
          onChange={(e) => setWebsite(e.target.value)}
          className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
          placeholder="https://acme.com"
        />
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">Industry</label>
        <select
          value={industry}
          onChange={(e) => setIndustry(e.target.value)}
          className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent bg-white"
        >
          <option value="">Select industry</option>
          {INDUSTRIES.map((ind) => (
            <option key={ind} value={ind}>
              {ind}
            </option>
          ))}
        </select>
      </div>

      {error && (
        <p className="text-sm text-red-600 bg-red-50 border border-red-200 rounded-lg px-3 py-2">
          {error}
        </p>
      )}

      <button
        type="submit"
        disabled={loading}
        className="w-full bg-emerald-500 hover:bg-emerald-600 disabled:opacity-50 text-white font-medium py-2 px-4 rounded-lg text-sm transition-colors"
      >
        {loading ? 'Saving…' : company ? 'Save changes' : 'Add company'}
      </button>
    </form>
  )
}
```

---

### Step 4 — Create the company server actions

Create the file `app/companies/actions.ts`.

File path: `clearcrm/app/companies/actions.ts`

```typescript
'use server'

import { createClient } from '@/utils/supabase/server'

type CompanyPayload = {
  name: string
  website: string | null
  industry: string | null
}

export async function addCompany(payload: CompanyPayload) {
  const supabase = createClient()
  const { error } = await supabase.from('companies').insert(payload)
  if (error) return { error: error.message }
}

export async function updateCompany(id: string, payload: CompanyPayload) {
  const supabase = createClient()
  const { error } = await supabase.from('companies').update(payload).eq('id', id)
  if (error) return { error: error.message }
}

export async function deleteCompany(id: string) {
  const supabase = createClient()
  const { error } = await supabase.from('companies').delete().eq('id', id)
  if (error) return { error: error.message }
}
```

---

### Step 5 — Update the contacts table to show company name

The contacts table already fetches company name via the nested select `companies ( id, name )` from Day 8. Open `app/contacts/page.tsx` and confirm the select includes it. The `ContactsTable` component already renders `contact.companies?.name`. No additional changes needed.

The contacts table columns already show the Company column. Verify by opening `http://localhost:3000/contacts` — the Company column should now show company names for contacts you assigned a company to.

---

### Step 6 — Wire up the sidebar Companies link

Open `components/Sidebar.tsx`. Find the "Companies" nav link. Set its href to `/companies`:

```typescript
<Link
  href="/companies"
  className="flex items-center gap-3 px-4 py-2 text-sm text-gray-400 hover:text-white hover:bg-white/10 rounded-lg transition-colors"
>
  Companies
</Link>
```

---

### Step 7 — Commit

```
git add . && git commit -m "feat: companies module + contacts-company relationship"
```

Directory: `clearcrm/`

---

## Practical workflow

1. Create `app/companies/page.tsx` — fetch companies with nested contact count, normalize count, render CompaniesTable
2. Create `app/companies/CompaniesTable.tsx` — client component, same drawer pattern as contacts
3. Create `app/companies/CompanyForm.tsx` — name, website, URL input, industry dropdown with fixed options
4. Create `app/companies/actions.ts` — addCompany, updateCompany, deleteCompany
5. Verify `app/contacts/page.tsx` select still includes `companies ( id, name )`
6. Update sidebar Companies link href to `/companies`
7. Test: add two companies → go to contacts → add a contact and assign a company → verify company name shows in contacts table → go to companies → verify contact count shows 1
8. Commit

---

## Common mistakes

**1. The contact count on the companies page always shows 0**

The nested count syntax `contacts ( count )` returns an array like `[{ count: 2 }]`, not a plain number. You must extract it with `company.contacts[0]?.count`. If you try to render `company.contacts` directly, TypeScript will also complain because Supabase infers it as a nested object. The normalization step in the page component (the `.map()` that creates `contact_count`) is required. Do not skip it.

**2. Deleting a company causes a Supabase error about foreign key constraints**

Your `contacts` table must have `company_id UUID REFERENCES companies(id) ON DELETE SET NULL`. If you created the table without this clause, deleting a company that has linked contacts will fail. Fix: run this in the Supabase SQL Editor to drop and re-add the foreign key constraint:

```sql
ALTER TABLE contacts DROP CONSTRAINT IF EXISTS contacts_company_id_fkey;
ALTER TABLE contacts ADD CONSTRAINT contacts_company_id_fkey
  FOREIGN KEY (company_id) REFERENCES companies(id) ON DELETE SET NULL;
```

**3. The website field shows a URL error when saving because the input type is "url" and the user typed without "https://"**

The `<input type="url">` requires a valid URL including the protocol. If users frequently type `acme.com` instead of `https://acme.com`, change the input type to `text` and add a placeholder that says `https://...` to guide them. Or change the type to `text` and add a note below the field: "Include https://".

---

## Your turn

Add two companies (e.g. "Acme Corp" in Technology and "Blue River" in Finance). Then go to Contacts and edit an existing contact — assign it to "Acme Corp". Go back to Companies — the Contacts count for Acme Corp should show 1.

Expected output: The Companies page shows two rows. Acme Corp has a green "1" badge in the Contacts column. Blue River has "0".

**Failure state:** The Companies page loads but the Contacts column shows 0 for all companies even after linking a contact.
Fix: The nested count requires an RLS SELECT policy on the `contacts` table. When Supabase runs the count aggregation, it still applies RLS. If you are seeing 0 counts, go to Supabase → Authentication → Policies → contacts table and confirm a `SELECT` policy exists for `authenticated` users. Also check your Supabase query is using `contacts ( count )` (the string `"count"` specifically, not `"id"` or `"*"`).

---

## Prompt / Template / Checklist pack

### Claude Code prompt — full Companies module with contact relationship

```
In this Next.js 14 App Router project (clearcrm), build the full Companies module.

Create these files:

1. app/companies/page.tsx — async server component that:
   - Imports the server Supabase client from utils/supabase/server.ts
   - Fetches all companies with a nested count of contacts using: .select('id, name, website, industry, created_at, contacts ( count )')
   - Normalizes the nested count array into a contact_count number on each company object
   - Orders by name ascending
   - Renders a page heading "Companies" with total count
   - Passes the companies array to a CompaniesTable client component

2. app/companies/CompaniesTable.tsx — 'use client' component that:
   - Receives companies with contact_count
   - Manages drawer state and editingCompany state
   - Renders an "+ Add Company" button (top right, emerald-500)
   - Renders a table with columns: Company, Website (as a clickable link), Industry, Contacts (as a green badge), and an actions column
   - Renders inline delete confirmation (same pattern as ContactsTable — Confirm/Cancel in the table cell)
   - Renders a right-side slide-over drawer with a backdrop on click-outside

3. app/companies/CompanyForm.tsx — 'use client' component with:
   - Text input for name (required)
   - URL input for website (optional)
   - Select dropdown for industry with options: Technology, Finance, Healthcare, Retail, Other
   - Calls addCompany or updateCompany based on whether a company prop is passed

4. app/companies/actions.ts — 'use server' file with three server actions:
   - addCompany(payload: { name, website, industry }) 
   - updateCompany(id, payload)
   - deleteCompany(id)
   Each returns { error: string } on failure, undefined on success.

Use Tailwind CSS. Emerald-500 accent. Sidebar background #0F172A. Match the visual style of the existing contacts module.
```

---

### Supabase relationships reference card

**The foreign key relationship between contacts and companies**

SQL definition (set up in Day 6):
```sql
-- contacts.company_id references companies.id
-- ON DELETE SET NULL means if you delete a company, linked contacts keep their data but company_id becomes NULL
CREATE TABLE contacts (
  ...
  company_id UUID REFERENCES companies(id) ON DELETE SET NULL,
  ...
);
```

**Fetching contacts with company name — the JOIN query in Supabase JS**

```typescript
const { data: contacts } = await supabase
  .from('contacts')
  .select(`
    id,
    first_name,
    last_name,
    email,
    phone,
    created_at,
    companies ( id, name )
  `)
  .order('created_at', { ascending: false })

// contacts[0].companies is either null (no company) or { id: '...', name: 'Acme Corp' }
// Access the name: contact.companies?.name ?? '—'
```

**Fetching companies with contact count**

```typescript
const { data: companies } = await supabase
  .from('companies')
  .select(`
    id,
    name,
    website,
    industry,
    contacts ( count )
  `)
  .order('name')

// Normalize the count — Supabase returns [{ count: 3 }], not a plain number
const normalized = companies.map((c) => ({
  ...c,
  contact_count: c.contacts[0]?.count ?? 0,
}))
```

**Linking a contact to a company (insert or update)**

```typescript
// Insert with company link
await supabase.from('contacts').insert({
  first_name: 'Jane',
  last_name: 'Smith',
  company_id: 'uuid-of-the-company',  // or null if no company
})

// Update to change company
await supabase
  .from('contacts')
  .update({ company_id: 'new-company-uuid' })
  .eq('id', 'contact-uuid')

// Unlink from company
await supabase
  .from('contacts')
  .update({ company_id: null })
  .eq('id', 'contact-uuid')
```
