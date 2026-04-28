# Day 8: Contacts Module — List, Add, Edit, Delete

## Outcome artifact
By the end of this day ClearCRM has a `/contacts` page that fetches all contacts from Supabase and displays them in a table, an "Add Contact" button that opens a slide-over drawer form, edit and delete actions on each row, and all four operations (create, read, update, delete) saving to Supabase.

## The core idea

The Contacts module is built across four files that each have a single responsibility. The page component is a server component — it runs on the server, fetches data from Supabase with your server client, and passes the data to child components as props. The table and form are client components — they run in the browser and handle user interaction. The actions file contains server actions — functions that run on the server but are called from the client, handling database writes.

This split exists because reading from a database is best done on the server (no round-trip, no loading state, secure). Writing data requires a user interaction first, which happens in the browser. Server actions bridge this gap: you call them from a client component but they execute on the server, so your Supabase service role key never touches the browser.

The slide-over drawer pattern (a panel that slides in from the right side of the screen) is a standard CRM UI pattern for add/edit forms. It keeps the user in context — they can see the contacts table behind the form. You will build the drawer as a client component using React state to control its open/closed visibility. No external library needed.

## Step-by-step walkthrough

### Step 1 — Create the contacts page (server component)

Create the file `app/contacts/page.tsx`. This is a server component that fetches all contacts and renders the table.

File path: `clearcrm/app/contacts/page.tsx`

```typescript
import { createClient } from '@/utils/supabase/server'
import ContactsTable from './ContactsTable'

export default async function ContactsPage() {
  const supabase = createClient()

  const { data: contacts, error } = await supabase
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

  if (error) {
    return (
      <div className="p-8">
        <p className="text-red-600">Failed to load contacts: {error.message}</p>
      </div>
    )
  }

  return (
    <div className="p-8">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">Contacts</h1>
          <p className="text-sm text-gray-500 mt-0.5">{contacts.length} total</p>
        </div>
      </div>
      <ContactsTable contacts={contacts ?? []} />
    </div>
  )
}
```

---

### Step 2 — Create the contacts table (client component)

Create the file `app/contacts/ContactsTable.tsx`. This client component receives contacts as props and renders the table. It also owns the state for which contact is being edited and whether the form drawer is open.

File path: `clearcrm/app/contacts/ContactsTable.tsx`

```typescript
'use client'

import { useState } from 'react'
import { useRouter } from 'next/navigation'
import ContactForm from './ContactForm'
import { deleteContact } from './actions'

type Company = { id: string; name: string }
type Contact = {
  id: string
  first_name: string
  last_name: string
  email: string | null
  phone: string | null
  created_at: string
  companies: Company | null
}

export default function ContactsTable({ contacts }: { contacts: Contact[] }) {
  const router = useRouter()
  const [drawerOpen, setDrawerOpen] = useState(false)
  const [editingContact, setEditingContact] = useState<Contact | null>(null)
  const [deletingId, setDeletingId] = useState<string | null>(null)
  const [showDeleteConfirm, setShowDeleteConfirm] = useState<string | null>(null)

  function openAddForm() {
    setEditingContact(null)
    setDrawerOpen(true)
  }

  function openEditForm(contact: Contact) {
    setEditingContact(contact)
    setDrawerOpen(true)
  }

  function closeDrawer() {
    setDrawerOpen(false)
    setEditingContact(null)
  }

  async function handleDelete(id: string) {
    setDeletingId(id)
    await deleteContact(id)
    setDeletingId(null)
    setShowDeleteConfirm(null)
    router.refresh()
  }

  return (
    <>
      {/* Add button lives inside the client component so it can open the drawer */}
      <div className="flex justify-end mb-4">
        <button
          onClick={openAddForm}
          className="bg-emerald-500 hover:bg-emerald-600 text-white text-sm font-medium px-4 py-2 rounded-lg transition-colors"
        >
          + Add Contact
        </button>
      </div>

      {contacts.length === 0 ? (
        <div className="text-center py-16 text-gray-400">
          <p className="text-lg font-medium">No contacts yet</p>
          <p className="text-sm mt-1">Click "Add Contact" to create your first one.</p>
        </div>
      ) : (
        <div className="bg-white border border-gray-200 rounded-xl overflow-hidden">
          <table className="w-full text-sm">
            <thead>
              <tr className="border-b border-gray-100 bg-gray-50">
                <th className="text-left px-4 py-3 font-medium text-gray-600">Name</th>
                <th className="text-left px-4 py-3 font-medium text-gray-600">Email</th>
                <th className="text-left px-4 py-3 font-medium text-gray-600">Phone</th>
                <th className="text-left px-4 py-3 font-medium text-gray-600">Company</th>
                <th className="text-left px-4 py-3 font-medium text-gray-600">Added</th>
                <th className="px-4 py-3"></th>
              </tr>
            </thead>
            <tbody>
              {contacts.map((contact) => (
                <tr key={contact.id} className="border-b border-gray-50 hover:bg-gray-50 transition-colors">
                  <td className="px-4 py-3 font-medium text-gray-900">
                    {contact.first_name} {contact.last_name}
                  </td>
                  <td className="px-4 py-3 text-gray-600">{contact.email ?? '—'}</td>
                  <td className="px-4 py-3 text-gray-600">{contact.phone ?? '—'}</td>
                  <td className="px-4 py-3 text-gray-600">
                    {contact.companies?.name ?? '—'}
                  </td>
                  <td className="px-4 py-3 text-gray-400">
                    {new Date(contact.created_at).toLocaleDateString()}
                  </td>
                  <td className="px-4 py-3">
                    <div className="flex items-center gap-2 justify-end">
                      <button
                        onClick={() => openEditForm(contact)}
                        className="text-gray-400 hover:text-gray-700 text-xs font-medium px-2 py-1 rounded hover:bg-gray-100 transition-colors"
                      >
                        Edit
                      </button>
                      {showDeleteConfirm === contact.id ? (
                        <div className="flex items-center gap-1">
                          <button
                            onClick={() => handleDelete(contact.id)}
                            disabled={deletingId === contact.id}
                            className="text-xs font-medium text-red-600 hover:text-red-700 px-2 py-1 rounded hover:bg-red-50 transition-colors disabled:opacity-50"
                          >
                            {deletingId === contact.id ? 'Deleting…' : 'Confirm'}
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
                          onClick={() => setShowDeleteConfirm(contact.id)}
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

      {/* Slide-over drawer */}
      {drawerOpen && (
        <>
          {/* Backdrop */}
          <div
            className="fixed inset-0 bg-black/30 z-40"
            onClick={closeDrawer}
          />
          {/* Drawer panel */}
          <div className="fixed right-0 top-0 h-full w-full max-w-md bg-white shadow-xl z-50 flex flex-col">
            <div className="flex items-center justify-between px-6 py-4 border-b border-gray-100">
              <h2 className="text-lg font-semibold text-gray-900">
                {editingContact ? 'Edit Contact' : 'Add Contact'}
              </h2>
              <button
                onClick={closeDrawer}
                className="text-gray-400 hover:text-gray-600 text-xl leading-none"
              >
                ×
              </button>
            </div>
            <div className="flex-1 overflow-y-auto p-6">
              <ContactForm
                contact={editingContact}
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

### Step 3 — Create the contact form (client component)

Create the file `app/contacts/ContactForm.tsx`. This client component handles both adding and editing a contact. It fetches the companies list from Supabase so the user can assign a company.

File path: `clearcrm/app/contacts/ContactForm.tsx`

```typescript
'use client'

import { useEffect, useState } from 'react'
import { createClient } from '@/utils/supabase/client'
import { addContact, updateContact } from './actions'

type Company = { id: string; name: string }
type Contact = {
  id: string
  first_name: string
  last_name: string
  email: string | null
  phone: string | null
  companies: Company | null
}

export default function ContactForm({
  contact,
  onSuccess,
}: {
  contact: Contact | null
  onSuccess: () => void
}) {
  const [firstName, setFirstName] = useState(contact?.first_name ?? '')
  const [lastName, setLastName] = useState(contact?.last_name ?? '')
  const [email, setEmail] = useState(contact?.email ?? '')
  const [phone, setPhone] = useState(contact?.phone ?? '')
  const [companyId, setCompanyId] = useState(contact?.companies?.id ?? '')
  const [companies, setCompanies] = useState<Company[]>([])
  const [loading, setLoading] = useState(false)
  const [error, setError] = useState<string | null>(null)

  useEffect(() => {
    const supabase = createClient()
    supabase
      .from('companies')
      .select('id, name')
      .order('name')
      .then(({ data }) => setCompanies(data ?? []))
  }, [])

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setLoading(true)
    setError(null)

    const payload = {
      first_name: firstName,
      last_name: lastName,
      email: email || null,
      phone: phone || null,
      company_id: companyId || null,
    }

    const result = contact
      ? await updateContact(contact.id, payload)
      : await addContact(payload)

    if (result?.error) {
      setError(result.error)
      setLoading(false)
      return
    }

    onSuccess()
  }

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div className="grid grid-cols-2 gap-4">
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">
            First name <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            required
            value={firstName}
            onChange={(e) => setFirstName(e.target.value)}
            className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
          />
        </div>
        <div>
          <label className="block text-sm font-medium text-gray-700 mb-1">
            Last name <span className="text-red-500">*</span>
          </label>
          <input
            type="text"
            required
            value={lastName}
            onChange={(e) => setLastName(e.target.value)}
            className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
          />
        </div>
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">Email</label>
        <input
          type="email"
          value={email}
          onChange={(e) => setEmail(e.target.value)}
          className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
          placeholder="contact@company.com"
        />
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">Phone</label>
        <input
          type="tel"
          value={phone}
          onChange={(e) => setPhone(e.target.value)}
          className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
          placeholder="+1 555 000 0000"
        />
      </div>

      <div>
        <label className="block text-sm font-medium text-gray-700 mb-1">Company</label>
        <select
          value={companyId}
          onChange={(e) => setCompanyId(e.target.value)}
          className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent bg-white"
        >
          <option value="">No company</option>
          {companies.map((c) => (
            <option key={c.id} value={c.id}>
              {c.name}
            </option>
          ))}
        </select>
      </div>

      {error && (
        <p className="text-sm text-red-600 bg-red-50 border border-red-200 rounded-lg px-3 py-2">
          {error}
        </p>
      )}

      <div className="flex gap-3 pt-2">
        <button
          type="submit"
          disabled={loading}
          className="flex-1 bg-emerald-500 hover:bg-emerald-600 disabled:opacity-50 text-white font-medium py-2 px-4 rounded-lg text-sm transition-colors"
        >
          {loading ? 'Saving…' : contact ? 'Save changes' : 'Add contact'}
        </button>
      </div>
    </form>
  )
}
```

---

### Step 4 — Create the server actions

Create the file `app/contacts/actions.ts`. This file contains three server actions: add, update, and delete. Server actions run on the server — they have access to the full server Supabase client and execute securely.

File path: `clearcrm/app/contacts/actions.ts`

```typescript
'use server'

import { createClient } from '@/utils/supabase/server'

type ContactPayload = {
  first_name: string
  last_name: string
  email: string | null
  phone: string | null
  company_id: string | null
}

export async function addContact(payload: ContactPayload) {
  const supabase = createClient()
  const { error } = await supabase.from('contacts').insert(payload)
  if (error) return { error: error.message }
}

export async function updateContact(id: string, payload: ContactPayload) {
  const supabase = createClient()
  const { error } = await supabase.from('contacts').update(payload).eq('id', id)
  if (error) return { error: error.message }
}

export async function deleteContact(id: string) {
  const supabase = createClient()
  const { error } = await supabase.from('contacts').delete().eq('id', id)
  if (error) return { error: error.message }
}
```

---

### Step 5 — Wire up the sidebar link

Open your `components/Sidebar.tsx`. Find the "Contacts" nav link. Make sure its `href` is set to `/contacts`:

```typescript
import Link from 'next/link'

// In your sidebar nav links:
<Link
  href="/contacts"
  className="flex items-center gap-3 px-4 py-2 text-sm text-gray-400 hover:text-white hover:bg-white/10 rounded-lg transition-colors"
>
  Contacts
</Link>
```

---

### Step 6 — Test each operation

1. Run `npm run dev` and open `http://localhost:3000/contacts`
2. Click **+ Add Contact** — the drawer should slide in from the right
3. Fill in first name, last name, and email — click **Add contact**
4. The drawer closes and the contact appears in the table
5. Click **Edit** on that row — the drawer opens with the contact's values pre-filled
6. Change the last name — click **Save changes** — the table updates
7. Click **Delete** on a row — a confirm/cancel prompt appears inline
8. Click **Confirm** — the row disappears

---

### Step 7 — Commit

```
git add . && git commit -m "feat: contacts CRUD — list, add, edit, delete"
```

Directory: `clearcrm/`

---

## Practical workflow

1. Create `app/contacts/page.tsx` — async server component, fetch contacts with company JOIN, render ContactsTable
2. Create `app/contacts/ContactsTable.tsx` — client component, manage drawer state, render table rows with edit/delete buttons
3. Create `app/contacts/ContactForm.tsx` — client component, controlled form, fetch companies on mount, call addContact or updateContact
4. Create `app/contacts/actions.ts` — three server actions: addContact, updateContact, deleteContact
5. Update sidebar Contacts link href to `/contacts`
6. Run `npm run dev`, test add → edit → delete flow
7. Commit

---

## Common mistakes

**1. After adding or editing a contact, the table does not update — the old data still shows**

The page is a server component. After a successful form submission, you must call `router.refresh()` to tell Next.js to re-fetch the server component data. The `ContactsTable` already calls `router.refresh()` in the `onSuccess` callback passed to `ContactForm`. If the table is not updating, check that `onSuccess` is actually being called after the action returns — add a `console.log('onSuccess called')` inside the callback to verify.

**2. The company dropdown in ContactForm shows no options**

The `useEffect` that fetches companies runs after the component mounts. If it shows empty, open the browser console and look for a Supabase error. The most common cause is an RLS policy missing on the `companies` table — the client-side Supabase call uses the anon key, and if no `SELECT` policy exists on `companies`, the query returns an empty array silently. Go to Supabase → Authentication → Policies and confirm the `companies` table has a SELECT policy for authenticated users.

**3. `deleteContact` deletes the contact but activities or deals linked to it throw a foreign key error**

Your `activities` and `deals` tables have `contact_id` as a foreign key with `ON DELETE SET NULL`. This means deleting a contact sets `contact_id` to NULL on linked records rather than blocking the delete. If you see a foreign key error, check your `CREATE TABLE` statements from Day 6 — the `ON DELETE SET NULL` clause must be present on both `activities.contact_id` and `deals.contact_id`.

---

## Your turn

Add at least three contacts through the UI. Then open the Supabase dashboard → Table Editor → contacts table. Verify all three contacts appear there with the correct field values.

Expected output: The `contacts` table in Supabase shows three rows. The `company_id` column shows `null` for contacts you did not assign a company to.

**Failure state:** You click Add Contact and the form submits but no new row appears in Supabase and no error shows in the UI.
Fix: Add a `console.log(result)` in the `handleSubmit` function in `ContactForm.tsx` right after the `addContact` or `updateContact` call. Check the browser console after submitting. If `result` contains an error object, the Supabase insert is failing — likely an RLS policy issue. If `result` is `undefined` (success), the problem is that `router.refresh()` is not triggering a re-fetch — ensure your page component is not cached by adding `export const dynamic = 'force-dynamic'` at the top of `app/contacts/page.tsx`.

---

## Prompt / Template / Checklist pack

### Claude Code Prompt 1 — Generate contacts list page

```
In this Next.js 14 App Router project (clearcrm), create app/contacts/page.tsx as an async server component. It should:
- Import the server Supabase client from utils/supabase/server.ts
- Fetch all rows from the contacts table, selecting id, first_name, last_name, email, phone, created_at, and join companies (id, name) using Supabase's nested select syntax
- Order by created_at descending
- Pass the contacts array as a prop to a ContactsTable client component (import from ./ContactsTable)
- Show a page heading "Contacts" with a subheading showing total count
- If there is a fetch error, display the error message in a red paragraph

Use Tailwind CSS. No placeholder data — fetch from Supabase only.
```

### Claude Code Prompt 2 — Add slide-over drawer form

```
In the ClearCRM Next.js 14 project, create app/contacts/ContactsTable.tsx as a 'use client' component.

It receives a contacts prop (array of contact objects including companies: { id, name } | null).

It should:
- Render a table with columns: Name, Email, Phone, Company, Added, and an actions column
- Show an "Add Contact" button above the table (top right)
- Manage state for: drawerOpen (boolean), editingContact (Contact | null), showDeleteConfirm (string | null — holds the id of the contact pending delete confirmation)
- When Add Contact is clicked: set editingContact to null, open drawer
- When Edit is clicked on a row: set editingContact to that contact, open drawer
- When Delete is clicked: show inline Confirm/Cancel buttons on that row (not a modal)
- The drawer is a right-side slide-over panel with a semi-transparent backdrop that closes when clicked
- Inside the drawer render a ContactForm component (import from ./ContactForm) with the editingContact prop and an onSuccess callback that closes the drawer and calls router.refresh()

Use Tailwind CSS. Emerald-500 as accent. Table has a white background, border, and rounded corners.
```

### Claude Code Prompt 3 — Delete with confirmation

```
In the ClearCRM Next.js 14 project, update app/contacts/ContactsTable.tsx to handle deletion with an inline confirmation pattern.

When the user clicks Delete on a row:
1. Set showDeleteConfirm to that contact's id
2. Replace the Delete button with two buttons: "Confirm" (red) and "Cancel" (gray)
3. When Confirm is clicked: call the deleteContact server action from ./actions, set a deletingId state to show "Deleting…" text, clear showDeleteConfirm after completion, then call router.refresh()
4. When Cancel is clicked: clear showDeleteConfirm without deleting

Import deleteContact from ./actions. The deleteContact function signature is: deleteContact(id: string) => Promise<{ error: string } | undefined>

Do not use a modal or dialog. The confirmation must appear inline in the table row's actions cell.
```

---

### Contacts module quick reference

The three server actions and their function signatures:

```typescript
// app/contacts/actions.ts

// Add a new contact — returns { error: string } on failure, undefined on success
export async function addContact(payload: {
  first_name: string
  last_name: string
  email: string | null
  phone: string | null
  company_id: string | null
}): Promise<{ error: string } | undefined>

// Update an existing contact by id
export async function updateContact(
  id: string,
  payload: {
    first_name: string
    last_name: string
    email: string | null
    phone: string | null
    company_id: string | null
  }
): Promise<{ error: string } | undefined>

// Delete a contact by id
export async function deleteContact(
  id: string
): Promise<{ error: string } | undefined>
```
