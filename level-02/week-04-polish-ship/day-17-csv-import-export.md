# Day 17: CSV Import and Export

## Outcome artifact

The Contacts page has an "Export CSV" button that downloads all contacts as a `.csv` file. It also has an "Import CSV" button that opens a file picker, parses the uploaded CSV client-side, previews the first 3 rows, and bulk-inserts all valid rows into Supabase. The import shows a summary: "12 contacts imported, 2 skipped (missing email)."

---

## The core idea

CSV export and import are table stakes for any CRM. Every sales team eventually needs to move data in and out — to share with colleagues, import from another tool, or back up records. You are building both directions today.

Export is simple: an API route fetches all contacts from Supabase, builds a CSV string, and returns it with the right HTTP headers so the browser treats it as a file download. No client-side JavaScript needed — the browser handles everything when it receives `Content-Disposition: attachment`.

Import is client-side: the user picks a `.csv` file, the browser reads it with the FileReader API, you parse the text into rows, validate each row (skip any missing an email), preview the first 3, then bulk-insert the valid rows via a server action. No library needed — CSV parsing with `split('\n')` handles well-formed CSVs correctly.

---

## Step-by-step walkthrough — Export

### Step 1 — Create the export API route

Next.js App Router supports Route Handlers — files named `route.ts` inside the `app/` directory. Create one at `app/contacts/export/route.ts`.

```bash
# from your project root
mkdir -p app/contacts/export
touch app/contacts/export/route.ts
```

Add the complete route handler:

```ts
// app/contacts/export/route.ts
import { createRouteHandlerClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { NextResponse } from 'next/server'

export const dynamic = 'force-dynamic'

export async function GET() {
  const supabase = createRouteHandlerClient({ cookies })

  // Check auth — redirect to login if not signed in
  const { data: { session } } = await supabase.auth.getSession()
  if (!session) {
    return NextResponse.redirect('/login')
  }

  // Fetch all contacts
  const { data: contacts, error } = await supabase
    .from('contacts')
    .select('first_name, last_name, email, phone')
    .order('last_name', { ascending: true })

  if (error) {
    return new Response('Failed to fetch contacts', { status: 500 })
  }

  // Build CSV string
  const header = 'first_name,last_name,email,phone'

  const rows = (contacts ?? []).map(contact => {
    // Escape fields that might contain commas or quotes
    const escape = (val: string | null) => {
      const str = val ?? ''
      if (str.includes(',') || str.includes('"') || str.includes('\n')) {
        return `"${str.replace(/"/g, '""')}"`
      }
      return str
    }

    return [
      escape(contact.first_name),
      escape(contact.last_name),
      escape(contact.email),
      escape(contact.phone),
    ].join(',')
  })

  const csv = [header, ...rows].join('\n')

  // Return as file download
  return new Response(csv, {
    headers: {
      'Content-Type': 'text/csv; charset=utf-8',
      'Content-Disposition': 'attachment; filename="contacts.csv"',
    },
  })
}
```

How `Content-Disposition: attachment` works: when a browser receives a response with this header, it treats the response body as a file download instead of displaying it on screen. The `filename="contacts.csv"` value sets the default save name. No JavaScript required on the client side.

The `escape()` function handles the one edge case that breaks naive CSV generation: fields that contain commas. If a name is "Smith, Jr." and you do not quote it, the CSV parser will split it into two fields. Wrapping in double quotes fixes this. Any literal double-quotes inside the field are escaped by doubling them (`"` becomes `""`).

---

### Step 2 — Add the Export CSV button to the contacts page

Open `app/contacts/page.tsx`. Find the heading or action bar at the top of the page and add the export button.

```tsx
{/* In app/contacts/page.tsx — add near the "Add Contact" button */}
<a
  href="/contacts/export"
  className="inline-flex items-center gap-2 px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors min-h-[44px]"
>
  <svg className="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
    <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2}
      d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-4l-4 4m0 0l-4-4m4 4V4"
    />
  </svg>
  Export CSV
</a>
```

Use a plain `<a>` tag — not a `<button>` with a click handler, not `router.push()`. The browser's native link navigation triggers the download automatically when it receives the `Content-Disposition: attachment` header.

Test it now before building the import. Click the button. Your browser should download a file named `contacts.csv`. Open it in a text editor or spreadsheet app to verify the content.

---

## Step-by-step walkthrough — Import

### Step 3 — Create the CSVImport component

Create the file:

```bash
touch components/CSVImport.tsx
```

Add the complete component:

```tsx
// components/CSVImport.tsx
'use client'

import { useRef, useState } from 'react'
import { importContacts } from '@/app/contacts/actions'

type ParsedRow = Record<string, string>

function parseCSV(text: string): ParsedRow[] {
  const lines = text.trim().split('\n')
  const headers = lines[0].split(',').map(h => h.trim().toLowerCase())
  return lines.slice(1).map(line => {
    const values = line.split(',').map(v => v.trim())
    return Object.fromEntries(headers.map((h, i) => [h, values[i] ?? '']))
  })
}

export default function CSVImport() {
  const fileInputRef = useRef<HTMLInputElement>(null)
  const [preview, setPreview] = useState<ParsedRow[]>([])
  const [allRows, setAllRows] = useState<ParsedRow[]>([])
  const [status, setStatus] = useState<'idle' | 'previewing' | 'importing' | 'done'>('idle')
  const [summary, setSummary] = useState('')

  function handleFileChange(e: React.ChangeEvent<HTMLInputElement>) {
    const file = e.target.files?.[0]
    if (!file) return

    const reader = new FileReader()
    reader.onload = (event) => {
      const text = event.target?.result as string
      const rows = parseCSV(text)
      setAllRows(rows)
      setPreview(rows.slice(0, 3))
      setStatus('previewing')
    }
    reader.readAsText(file)
  }

  async function handleImport() {
    setStatus('importing')

    // Validate rows — skip any missing an email
    const validRows = allRows.filter(row => row.email && row.email.trim() !== '')
    const skippedCount = allRows.length - validRows.length

    const result = await importContacts(validRows)

    if (result.error) {
      setSummary(`Import failed: ${result.error}`)
    } else {
      setSummary(`${result.imported} contacts imported, ${skippedCount} skipped (missing email)`)
    }

    setStatus('done')
    setAllRows([])
    setPreview([])
    if (fileInputRef.current) fileInputRef.current.value = ''
  }

  function handleReset() {
    setStatus('idle')
    setSummary('')
    setAllRows([])
    setPreview([])
    if (fileInputRef.current) fileInputRef.current.value = ''
  }

  return (
    <div className="space-y-4">
      {/* File picker trigger */}
      {status === 'idle' && (
        <div>
          <input
            ref={fileInputRef}
            type="file"
            accept=".csv"
            onChange={handleFileChange}
            className="hidden"
            id="csv-file-input"
          />
          <label
            htmlFor="csv-file-input"
            className="inline-flex items-center gap-2 px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 cursor-pointer transition-colors min-h-[44px]"
          >
            <svg className="w-4 h-4" fill="none" viewBox="0 0 24 24" stroke="currentColor">
              <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2}
                d="M4 16v1a3 3 0 003 3h10a3 3 0 003-3v-1m-4-8l4-4m0 0l4 4m-4-4v12"
              />
            </svg>
            Import CSV
          </label>
        </div>
      )}

      {/* Preview panel */}
      {status === 'previewing' && preview.length > 0 && (
        <div className="border border-gray-200 rounded-lg overflow-hidden">
          <div className="bg-gray-50 px-4 py-2 border-b border-gray-200">
            <p className="text-sm font-medium text-gray-700">
              Preview — first {preview.length} of {allRows.length} rows
            </p>
          </div>
          <div className="overflow-x-auto">
            <table className="w-full text-sm">
              <thead className="bg-gray-50 text-xs text-gray-500 uppercase">
                <tr>
                  <th className="px-4 py-2 text-left">First Name</th>
                  <th className="px-4 py-2 text-left">Last Name</th>
                  <th className="px-4 py-2 text-left">Email</th>
                  <th className="px-4 py-2 text-left">Phone</th>
                </tr>
              </thead>
              <tbody className="divide-y divide-gray-100">
                {preview.map((row, i) => (
                  <tr key={i} className={!row.email ? 'bg-red-50' : ''}>
                    <td className="px-4 py-2 text-gray-800">{row.first_name || '—'}</td>
                    <td className="px-4 py-2 text-gray-800">{row.last_name || '—'}</td>
                    <td className="px-4 py-2 text-gray-800">
                      {row.email || <span className="text-red-500">missing</span>}
                    </td>
                    <td className="px-4 py-2 text-gray-800">{row.phone || '—'}</td>
                  </tr>
                ))}
              </tbody>
            </table>
          </div>
          <div className="px-4 py-3 bg-gray-50 border-t border-gray-200 flex gap-3">
            <button
              onClick={handleImport}
              className="px-4 py-2 text-sm font-medium text-white bg-emerald-600 rounded-lg hover:bg-emerald-700 transition-colors min-h-[44px]"
            >
              Import {allRows.length} contacts
            </button>
            <button
              onClick={handleReset}
              className="px-4 py-2 text-sm font-medium text-gray-700 bg-white border border-gray-300 rounded-lg hover:bg-gray-50 transition-colors min-h-[44px]"
            >
              Cancel
            </button>
          </div>
        </div>
      )}

      {/* Importing state */}
      {status === 'importing' && (
        <p className="text-sm text-gray-600">Importing contacts...</p>
      )}

      {/* Done state */}
      {status === 'done' && (
        <div className="flex items-center gap-3 p-4 bg-emerald-50 border border-emerald-200 rounded-lg">
          <svg className="w-5 h-5 text-emerald-600 shrink-0" fill="none" viewBox="0 0 24 24" stroke="currentColor">
            <path strokeLinecap="round" strokeLinejoin="round" strokeWidth={2} d="M5 13l4 4L19 7" />
          </svg>
          <p className="text-sm font-medium text-emerald-800">{summary}</p>
          <button
            onClick={handleReset}
            className="ml-auto text-sm text-emerald-700 underline hover:text-emerald-900"
          >
            Import another
          </button>
        </div>
      )}
    </div>
  )
}
```

---

### Step 4 — The CSV parse function, explained

The `parseCSV` function handles the import with no library:

```js
function parseCSV(text: string): Record<string, string>[] {
  const lines = text.trim().split('\n')
  const headers = lines[0].split(',').map(h => h.trim().toLowerCase())
  return lines.slice(1).map(line => {
    const values = line.split(',').map(v => v.trim())
    return Object.fromEntries(headers.map((h, i) => [h, values[i] ?? '']))
  })
}
```

Line by line:

- `text.trim().split('\n')` — remove leading/trailing whitespace, split into lines. Works on both Unix (`\n`) and Windows (`\r\n`) line endings because `.trim()` strips `\r` from the last field.
- `lines[0].split(',').map(h => h.trim().toLowerCase())` — the first line is always the header row. Convert to lowercase so `First_Name` and `first_name` both work.
- `lines.slice(1)` — skip the header, process only data rows.
- `Object.fromEntries(headers.map((h, i) => [h, values[i] ?? '']))` — zip headers and values into an object. `values[i] ?? ''` handles rows with fewer columns than headers.

Result for one data row: `{ first_name: 'Sarah', last_name: 'Johnson', email: 'sarah@techcorp.co.za', phone: '021-555-0101' }`

Limitation: this parser does not handle quoted fields containing commas (e.g. `"Smith, Jr."`). For the standard CRM contact CSV format — first name, last name, email, phone — this is not a problem. If you need to handle quoted fields in production, use the `papaparse` library.

---

### Step 5 — Create the importContacts server action

Open (or create) `app/contacts/actions.ts`:

```bash
touch app/contacts/actions.ts
```

Add the `importContacts` server action:

```ts
// app/contacts/actions.ts
'use server'

import { createServerActionClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { revalidatePath } from 'next/cache'

type ContactRow = {
  first_name?: string
  last_name?: string
  email: string
  phone?: string
}

export async function importContacts(rows: Record<string, string>[]) {
  const supabase = createServerActionClient({ cookies })

  const { data: { session } } = await supabase.auth.getSession()
  if (!session) return { error: 'Not authenticated', imported: 0 }

  // Map CSV rows to contact shape — only include rows with a valid email
  const contacts: ContactRow[] = rows
    .filter(row => row.email && row.email.trim() !== '')
    .map(row => ({
      first_name: row.first_name?.trim() || null,
      last_name: row.last_name?.trim() || null,
      email: row.email.trim().toLowerCase(),
      phone: row.phone?.trim() || null,
    }))

  if (contacts.length === 0) {
    return { error: 'No valid rows to import', imported: 0 }
  }

  // Bulk insert — Supabase .insert() accepts an array
  const { data, error } = await supabase
    .from('contacts')
    .insert(contacts)
    .select()

  if (error) {
    console.error('Import error:', error)
    return { error: error.message, imported: 0 }
  }

  revalidatePath('/contacts')
  return { imported: data?.length ?? 0, error: null }
}
```

How bulk insert works: `.insert(contacts)` where `contacts` is an array sends a single HTTP request to Supabase and inserts all rows in one database transaction. For 100 contacts this is one request, not 100.

`revalidatePath('/contacts')` tells Next.js to re-fetch the data for the contacts page so the new contacts appear immediately.

---

### Step 6 — Add the CSVImport component to the contacts page

Open `app/contacts/page.tsx` and import the component:

```tsx
import CSVImport from '@/components/CSVImport'
```

Add it near the Export button in the action bar:

```tsx
<div className="flex flex-wrap gap-3 items-center">
  {/* Add Contact button */}
  <button ...>Add Contact</button>

  {/* Export CSV */}
  <a href="/contacts/export" ...>Export CSV</a>

  {/* Import CSV */}
  <CSVImport />
</div>
```

---

### Step 7 — Test the import

Download the sample CSV from the deliverable section below. Save it as `test-contacts.csv`. Click "Import CSV" on the contacts page. Select the file. Verify the preview shows the first 3 rows. Click "Import 5 contacts." Verify the summary shows "5 contacts imported, 0 skipped." Navigate away and back — all 5 contacts should appear in the list.

---

### Step 8 — Commit

```bash
# from your project root
git add .
git commit -m "feat: CSV export and import for contacts"
```

Expected output:

```
[main c7d4e21] feat: CSV export and import for contacts
 4 files changed, 203 insertions(+)
 create mode 100644 app/contacts/export/route.ts
 create mode 100644 app/contacts/actions.ts
 create mode 100644 components/CSVImport.tsx
```

---

## Practical workflow

Always test export before import. Export your existing contacts, open the CSV, verify the format matches the expected columns. Then import that same CSV back — it should import with 0 skipped rows because every exported contact has an email.

If the import shows 0 imported: open the browser console. The server action logs errors with `console.error`. Check the Supabase dashboard → Logs → API for the insert error.

If the export file opens as garbled text: add a UTF-8 BOM to the CSV so Excel recognizes the encoding. Change the response to:

```ts
const bom = '﻿'
const csv = bom + [header, ...rows].join('\n')
```

---

## Common mistakes

**The export button downloads an HTML page instead of a CSV.**
This happens when the API route is not found and Next.js falls back to the 404 page. Verify the file is at exactly `app/contacts/export/route.ts` — not `route.tsx`, not inside an `(auth)` route group that changes the URL.

**The import shows "0 imported" with no error.**
The CSV headers do not match what the parse function expects. The function lowercases headers, so `Email` becomes `email`. But if the CSV uses `email_address` instead of `email`, the filter `row.email && row.email.trim() !== ''` drops every row. Open the console, log the parsed rows, and check the field names.

**Supabase insert returns a unique constraint violation.**
Your contacts table probably has a unique constraint on `email`. Importing a CSV with duplicate emails (or emails already in the database) will cause the insert to fail for the entire batch. Handle this by adding `.upsert(contacts, { onConflict: 'email' })` instead of `.insert(contacts)`. Upsert updates existing records rather than failing.

```ts
const { data, error } = await supabase
  .from('contacts')
  .upsert(contacts, { onConflict: 'email' })
  .select()
```

**Rows with trailing whitespace are not recognized.**
A CSV exported from Excel sometimes has Windows line endings (`\r\n`). `split('\n')` leaves a `\r` at the end of each line. The `.trim()` call on each value handles this, but the header trim must also handle it. The function already does `.map(h => h.trim().toLowerCase())` on headers — this is why the trim is there.

**The preview shows undefined for all fields.**
The CSV file uses semicolons (`;`) instead of commas as the delimiter. This is common in European locales where the comma is a decimal separator. Check the file in a text editor. If it uses semicolons, change `line.split(',')` to `line.split(';')` in the `parseCSV` function, or detect the delimiter automatically.

---

## Your turn

1. Add CSV export for companies. Create `app/companies/export/route.ts` with columns: `name, industry, website, phone`.
2. Extend the import to show a red badge on preview rows that are missing an email — so the student sees exactly which rows will be skipped before confirming.
3. Add a "Download template" link next to the Import button. It should download a blank CSV with just the header row, so users know the exact format to use.

---

## Prompt / Template / Checklist pack

### Prompt 1 — Build the CSV export API route

```
I am building ClearCRM with Next.js 14 App Router and Supabase.

Create app/contacts/export/route.ts — a GET route handler that:
1. Uses createRouteHandlerClient from @supabase/auth-helpers-nextjs
2. Checks that the user has an active session — return a 401 if not
3. Fetches all contacts from the contacts table: columns first_name, last_name, email, phone
4. Builds a CSV string with a header row and one row per contact
5. Escapes any fields containing commas by wrapping in double quotes
6. Returns the CSV as a file download with these headers:
   Content-Type: text/csv; charset=utf-8
   Content-Disposition: attachment; filename="contacts.csv"

Add export const dynamic = 'force-dynamic' at the top.
Also add an <a href="/contacts/export"> Export CSV button to app/contacts/page.tsx.
The button should have Tailwind classes for a secondary button style (white background, gray border).
```

---

### Prompt 2 — Build the CSV import component

```
I am building ClearCRM with Next.js 14 App Router and Supabase.

Create two files:

1. components/CSVImport.tsx — a 'use client' component that:
   - Shows an "Import CSV" button (styled as a secondary button, min-h-[44px])
   - On file selection, reads the file with FileReader and parses it with this exact function:
     function parseCSV(text: string): Record<string, string>[] {
       const lines = text.trim().split('\n')
       const headers = lines[0].split(',').map(h => h.trim().toLowerCase())
       return lines.slice(1).map(line => {
         const values = line.split(',').map(v => v.trim())
         return Object.fromEntries(headers.map((h, i) => [h, values[i] ?? '']))
       })
     }
   - Shows a preview table with the first 3 rows (columns: first_name, last_name, email, phone)
   - Highlights rows missing an email in red (bg-red-50)
   - Has an "Import N contacts" confirm button and a Cancel button
   - On confirm, calls the importContacts server action with all rows
   - Shows a summary: "N contacts imported, M skipped (missing email)"

2. app/contacts/actions.ts — a server action file with:
   - 'use server' directive
   - importContacts(rows: Record<string, string>[]) function that:
     - Filters out rows with no email
     - Maps rows to { first_name, last_name, email, phone } shape
     - Bulk inserts with supabase.from('contacts').insert(contacts).select()
     - Calls revalidatePath('/contacts')
     - Returns { imported: number, error: string | null }

Use @supabase/auth-helpers-nextjs. Use Tailwind CSS. Accent color is emerald-600.
```

---

## Deliverable — CSV format template

Save this as `test-contacts.csv` to test your import feature. Copy exactly — including the header row.

```csv
first_name,last_name,email,phone
Amahle,Dlamini,amahle.dlamini@horizontech.co.za,083-441-2290
Pieter,van der Merwe,pieter.vdm@capeconsult.co.za,021-887-5543
Nokwanda,Sithole,nokwanda.s@joburg-digital.co.za,011-663-8821
Riyaad,Isaacs,riyaad.isaacs@blueprint-sa.com,074-219-0067
Chantel,Botha,chantel.botha@saldanha-media.co.za,022-714-3395
```

These are 5 rows of realistic South African names and emails. All 5 have valid emails — the import should show "5 contacts imported, 0 skipped."

To test the skip logic, add a 6th row with no email before importing:

```csv
first_name,last_name,email,phone
Amahle,Dlamini,amahle.dlamini@horizontech.co.za,083-441-2290
Pieter,van der Merwe,pieter.vdm@capeconsult.co.za,021-887-5543
Nokwanda,Sithole,nokwanda.s@joburg-digital.co.za,011-663-8821
Riyaad,Isaacs,riyaad.isaacs@blueprint-sa.com,074-219-0067
Chantel,Botha,chantel.botha@saldanha-media.co.za,022-714-3395
Sipho,Khumalo,,082-554-1199
```

The last row has no email. The import should show "5 contacts imported, 1 skipped (missing email)."
