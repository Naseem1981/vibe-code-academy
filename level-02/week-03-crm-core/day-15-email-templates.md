# Day 15: Email Templates

## Outcome artifact

ClearCRM has an `/email-templates` page where you can create, preview, edit, and delete email templates. Each template has a name, subject line, and body. A live preview pane shows the template with placeholder values substituted in. A "Copy to Clipboard" button copies the rendered subject and body, ready to paste directly into Gmail or Outlook. Three starter templates are seeded into the database.

---

## The core idea

Email templates save repetitive typing. A salesperson sends the same introduction email, the same follow-up, the same proposal cover email hundreds of times. Store the template once. Personalise it with placeholders like `{{first_name}}` and `{{company}}`. Copy and paste when needed.

The placeholder replacement function scans the template body for `{{variable_name}}` patterns and swaps each one with a real value. The Clipboard API (`navigator.clipboard.writeText`) writes text to the user's clipboard — no library, one line of code.

The preview pane updates in real time as the user types. This is done with controlled textarea state — the same pattern you have used in previous forms.

---

## Step-by-step walkthrough

### Step 1 — Create the email_templates table in Supabase

Open the Supabase SQL Editor. Run this exact SQL:

```sql
-- Create email_templates table
CREATE TABLE email_templates (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  subject TEXT NOT NULL,
  body TEXT NOT NULL,
  created_at TIMESTAMPTZ NOT NULL DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE email_templates ENABLE ROW LEVEL SECURITY;

-- Policy: authenticated users can read all templates
CREATE POLICY "Authenticated users can read templates"
  ON email_templates FOR SELECT
  TO authenticated
  USING (true);

-- Policy: authenticated users can insert templates
CREATE POLICY "Authenticated users can insert templates"
  ON email_templates FOR INSERT
  TO authenticated
  WITH CHECK (true);

-- Policy: authenticated users can update templates
CREATE POLICY "Authenticated users can update templates"
  ON email_templates FOR UPDATE
  TO authenticated
  USING (true);

-- Policy: authenticated users can delete templates
CREATE POLICY "Authenticated users can delete templates"
  ON email_templates FOR DELETE
  TO authenticated
  USING (true);
```

Expected output: a green success banner reading "Success. No rows returned."

---

### Step 2 — Seed the three starter templates

Still in the Supabase SQL Editor, run this INSERT to add the three starter templates:

```sql
INSERT INTO email_templates (name, subject, body) VALUES
(
  'Introduction Email',
  'Introduction: {{your_name}} from {{your_company}}',
  'Hi {{first_name}},

I hope this finds you well. My name is {{your_name}} and I work at {{your_company}}.

We help businesses like {{company}} with [your value proposition in one sentence].

I''d love to schedule a quick 15-minute call to see if there''s a fit. Would you have time this week or next?

Looking forward to connecting.

Best regards,
{{your_name}}
{{your_company}}'
),
(
  'Follow-Up Email',
  'Following up — {{your_company}}',
  'Hi {{first_name}},

I sent you an email a few days ago and wanted to follow up in case it got buried.

We''ve been helping companies similar to {{company}} to [specific outcome]. I think there could be a good opportunity for us to work together.

Would you be open to a brief conversation this week?

Best,
{{your_name}}'
),
(
  'Proposal Email',
  'Proposal for {{company}} — {{deal_title}}',
  'Hi {{first_name}},

Thank you for our conversation earlier. As discussed, please find our proposal for {{deal_title}} attached.

Proposal summary:
- Scope: [brief scope description]
- Investment: {{deal_value}}
- Timeline: [estimated timeline]
- Next step: [what you need from them to proceed]

Please review and let me know if you have any questions. I''m available for a call to walk you through the details.

Looking forward to working with you.

Best regards,
{{your_name}}
{{your_company}}'
);
```

Expected output: "Success. 3 rows affected."

---

### Step 3 — Create the email templates page

```
Directory: ~/clearcrm
Command:   mkdir -p app/email-templates
```

Create `app/email-templates/page.tsx`:

```tsx
// app/email-templates/page.tsx
import { createServerComponentClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import TemplateList from './TemplateList'
import type { Database } from '@/types/database'

export type EmailTemplate = {
  id: string
  name: string
  subject: string
  body: string
  created_at: string
}

export default async function EmailTemplatesPage() {
  const supabase = createServerComponentClient<Database>({ cookies })

  const { data: templates, error } = await supabase
    .from('email_templates')
    .select('*')
    .order('created_at', { ascending: true })

  if (error) {
    console.error('Error fetching templates:', error)
  }

  return (
    <div className="p-6 max-w-5xl mx-auto">
      <div className="flex items-center justify-between mb-6">
        <div>
          <h1 className="text-2xl font-bold text-gray-900">Email Templates</h1>
          <p className="text-sm text-gray-500 mt-1">
            {templates?.length ?? 0} template{templates?.length !== 1 ? 's' : ''}
          </p>
        </div>
      </div>

      <TemplateList initialTemplates={(templates ?? []) as EmailTemplate[]} />
    </div>
  )
}
```

---

### Step 4 — Create the TemplateList component

Create `app/email-templates/TemplateList.tsx`:

```tsx
// app/email-templates/TemplateList.tsx
'use client'

import { useState } from 'react'
import TemplateCard from './TemplateCard'
import TemplateForm from './TemplateForm'
import type { EmailTemplate } from './page'

type Props = {
  initialTemplates: EmailTemplate[]
}

export default function TemplateList({ initialTemplates }: Props) {
  const [showForm, setShowForm] = useState(false)
  const [editingTemplate, setEditingTemplate] = useState<EmailTemplate | null>(null)

  function handleEdit(template: EmailTemplate) {
    setEditingTemplate(template)
    setShowForm(true)
  }

  function handleClose() {
    setShowForm(false)
    setEditingTemplate(null)
  }

  return (
    <>
      {/* Add Template button */}
      <div className="mb-6 flex justify-end">
        <button
          onClick={() => setShowForm(true)}
          className="bg-emerald-600 text-white text-sm font-medium px-4 py-2 rounded-lg hover:bg-emerald-700 transition-colors"
        >
          + New Template
        </button>
      </div>

      {/* Template grid */}
      {initialTemplates.length === 0 ? (
        <div className="text-center py-16 text-gray-400">
          <p className="text-4xl mb-3">📧</p>
          <p className="font-medium text-gray-600">No templates yet</p>
          <p className="text-sm mt-1">Create your first email template above.</p>
        </div>
      ) : (
        <div className="grid gap-4">
          {initialTemplates.map((template) => (
            <TemplateCard
              key={template.id}
              template={template}
              onEdit={() => handleEdit(template)}
            />
          ))}
        </div>
      )}

      {/* Create / Edit modal */}
      {showForm && (
        <div className="fixed inset-0 bg-black/40 flex items-center justify-center z-50 p-4">
          <div className="bg-white rounded-2xl shadow-xl w-full max-w-2xl max-h-[90vh] overflow-y-auto">
            <div className="p-6">
              <div className="flex items-center justify-between mb-4">
                <h2 className="text-lg font-bold text-gray-900">
                  {editingTemplate ? 'Edit Template' : 'New Template'}
                </h2>
                <button
                  onClick={handleClose}
                  className="text-gray-400 hover:text-gray-600 text-xl leading-none"
                >
                  ×
                </button>
              </div>
              <TemplateForm
                template={editingTemplate}
                onSuccess={handleClose}
              />
            </div>
          </div>
        </div>
      )}
    </>
  )
}
```

---

### Step 5 — Create the TemplateCard component

Create `app/email-templates/TemplateCard.tsx`:

```tsx
// app/email-templates/TemplateCard.tsx
'use client'

import { useState } from 'react'
import { deleteTemplate } from './actions'
import { useRouter } from 'next/navigation'

// Placeholder replacement function
function applyPlaceholders(text: string, data: Record<string, string>): string {
  return text.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] ?? `{{${key}}}`)
}

// Preview data — shown when the user clicks "Preview"
const PREVIEW_DATA: Record<string, string> = {
  first_name: 'Sarah',
  company: 'Acme Corp',
  deal_title: 'CRM Implementation',
  deal_value: 'R 45,000',
  your_name: 'John Doe',
  your_company: 'ClearCRM Ltd',
}

type Props = {
  template: {
    id: string
    name: string
    subject: string
    body: string
  }
  onEdit: () => void
}

export default function TemplateCard({ template, onEdit }: Props) {
  const [showPreview, setShowPreview] = useState(false)
  const [copied, setCopied] = useState(false)
  const [isDeleting, setIsDeleting] = useState(false)
  const router = useRouter()

  // Apply placeholder replacement to both subject and body
  const renderedSubject = applyPlaceholders(template.subject, PREVIEW_DATA)
  const renderedBody = applyPlaceholders(template.body, PREVIEW_DATA)

  async function handleCopy() {
    const textToCopy = `Subject: ${renderedSubject}\n\n${renderedBody}`
    await navigator.clipboard.writeText(textToCopy)
    setCopied(true)
    setTimeout(() => setCopied(false), 2000)
  }

  async function handleDelete() {
    if (!confirm(`Delete "${template.name}"? This cannot be undone.`)) return
    setIsDeleting(true)
    await deleteTemplate(template.id)
    router.refresh()
  }

  return (
    <div className="bg-white rounded-2xl border border-gray-200 shadow-sm overflow-hidden">
      {/* Card header */}
      <div className="px-5 py-4 flex items-start justify-between gap-4">
        <div className="min-w-0 flex-1">
          <h3 className="font-semibold text-gray-900 truncate">{template.name}</h3>
          <p className="text-sm text-gray-500 mt-0.5 truncate">{template.subject}</p>
          <p className="text-sm text-gray-400 mt-1 line-clamp-2 leading-relaxed">
            {template.body.substring(0, 120)}
            {template.body.length > 120 ? '…' : ''}
          </p>
        </div>

        {/* Action buttons */}
        <div className="flex items-center gap-2 flex-shrink-0">
          <button
            onClick={() => setShowPreview(!showPreview)}
            className="text-xs font-medium px-3 py-1.5 rounded-lg border border-gray-300 text-gray-600 hover:bg-gray-50 transition-colors"
          >
            {showPreview ? 'Hide' : 'Preview'}
          </button>
          <button
            onClick={handleCopy}
            className={`text-xs font-medium px-3 py-1.5 rounded-lg transition-colors ${
              copied
                ? 'bg-emerald-100 text-emerald-700 border border-emerald-200'
                : 'bg-emerald-600 text-white hover:bg-emerald-700'
            }`}
          >
            {copied ? '✓ Copied' : 'Copy'}
          </button>
          <button
            onClick={onEdit}
            className="text-xs font-medium px-3 py-1.5 rounded-lg border border-gray-300 text-gray-600 hover:bg-gray-50 transition-colors"
          >
            Edit
          </button>
          <button
            onClick={handleDelete}
            disabled={isDeleting}
            className="text-xs font-medium px-3 py-1.5 rounded-lg border border-red-200 text-red-600 hover:bg-red-50 transition-colors disabled:opacity-50"
          >
            {isDeleting ? '…' : 'Delete'}
          </button>
        </div>
      </div>

      {/* Preview panel */}
      {showPreview && (
        <div className="border-t border-gray-100 bg-gray-50 px-5 py-4">
          <p className="text-xs font-semibold text-gray-400 uppercase tracking-wider mb-3">
            Preview (with sample data)
          </p>
          <div className="bg-white rounded-xl border border-gray-200 p-4">
            <p className="text-xs text-gray-500 mb-1">Subject:</p>
            <p className="text-sm font-semibold text-gray-900 mb-4">
              {renderedSubject}
            </p>
            <p className="text-xs text-gray-500 mb-1">Body:</p>
            <pre className="text-sm text-gray-700 whitespace-pre-wrap font-sans leading-relaxed">
              {renderedBody}
            </pre>
          </div>
        </div>
      )}
    </div>
  )
}
```

---

### Step 6 — Create the TemplateForm component

Create `app/email-templates/TemplateForm.tsx`:

```tsx
// app/email-templates/TemplateForm.tsx
'use client'

import { useState, useTransition } from 'react'
import { addTemplate, updateTemplate } from './actions'
import { useRouter } from 'next/navigation'

function applyPlaceholders(text: string, data: Record<string, string>): string {
  return text.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] ?? `{{${key}}}`)
}

const PREVIEW_DATA: Record<string, string> = {
  first_name: 'Sarah',
  company: 'Acme Corp',
  deal_title: 'CRM Implementation',
  deal_value: 'R 45,000',
  your_name: 'John Doe',
  your_company: 'ClearCRM Ltd',
}

type Template = {
  id: string
  name: string
  subject: string
  body: string
}

type Props = {
  template?: Template | null
  onSuccess: () => void
}

export default function TemplateForm({ template, onSuccess }: Props) {
  const [name, setName] = useState(template?.name ?? '')
  const [subject, setSubject] = useState(template?.subject ?? '')
  const [body, setBody] = useState(template?.body ?? '')
  const [isPending, startTransition] = useTransition()
  const [error, setError] = useState<string | null>(null)
  const router = useRouter()

  const isEditing = !!template

  // Live preview with placeholder substitution
  const previewSubject = applyPlaceholders(subject, PREVIEW_DATA)
  const previewBody = applyPlaceholders(body, PREVIEW_DATA)

  async function handleSubmit(e: React.FormEvent) {
    e.preventDefault()
    setError(null)

    const formData = new FormData()
    formData.set('name', name)
    formData.set('subject', subject)
    formData.set('body', body)

    startTransition(async () => {
      let result
      if (isEditing) {
        result = await updateTemplate(template.id, formData)
      } else {
        result = await addTemplate(formData)
      }

      if (result?.error) {
        setError(result.error)
      } else {
        router.refresh()
        onSuccess()
      }
    })
  }

  return (
    <form onSubmit={handleSubmit}>
      <div className="grid grid-cols-1 md:grid-cols-2 gap-6">
        {/* Left: form fields */}
        <div className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              Template name <span className="text-red-500">*</span>
            </label>
            <input
              type="text"
              value={name}
              onChange={(e) => setName(e.target.value)}
              required
              placeholder="e.g. Introduction Email"
              className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              Subject line <span className="text-red-500">*</span>
            </label>
            <input
              type="text"
              value={subject}
              onChange={(e) => setSubject(e.target.value)}
              required
              placeholder="e.g. Introduction: {{your_name}} from {{your_company}}"
              className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500"
            />
          </div>

          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              Body <span className="text-red-500">*</span>
            </label>
            <textarea
              value={body}
              onChange={(e) => setBody(e.target.value)}
              required
              rows={12}
              placeholder="Write your email body. Use {{first_name}}, {{company}}, {{deal_title}} as placeholders."
              className="w-full border border-gray-300 rounded-lg px-3 py-2 text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 resize-none font-mono"
            />
          </div>

          <div className="bg-gray-50 rounded-lg p-3 text-xs text-gray-500">
            <p className="font-semibold text-gray-600 mb-1">Available placeholders:</p>
            <code className="text-emerald-700">{'{{first_name}}'}</code>,{' '}
            <code className="text-emerald-700">{'{{company}}'}</code>,{' '}
            <code className="text-emerald-700">{'{{deal_title}}'}</code>,{' '}
            <code className="text-emerald-700">{'{{deal_value}}'}</code>,{' '}
            <code className="text-emerald-700">{'{{your_name}}'}</code>,{' '}
            <code className="text-emerald-700">{'{{your_company}}'}</code>
          </div>
        </div>

        {/* Right: live preview */}
        <div>
          <p className="text-sm font-medium text-gray-700 mb-1">Live preview</p>
          <div className="bg-gray-50 rounded-xl border border-gray-200 p-4 h-full">
            <p className="text-xs text-gray-400 mb-1">Subject:</p>
            <p className="text-sm font-semibold text-gray-900 mb-4 min-h-[20px]">
              {previewSubject || <span className="text-gray-300">Subject will appear here…</span>}
            </p>
            <p className="text-xs text-gray-400 mb-1">Body:</p>
            <pre className="text-sm text-gray-700 whitespace-pre-wrap font-sans leading-relaxed min-h-[100px]">
              {previewBody || <span className="text-gray-300">Body will appear here…</span>}
            </pre>
          </div>
        </div>
      </div>

      {error && (
        <p className="text-sm text-red-600 bg-red-50 border border-red-200 rounded-lg px-3 py-2 mt-4">
          {error}
        </p>
      )}

      <div className="flex gap-3 pt-4 mt-4 border-t border-gray-100">
        <button
          type="submit"
          disabled={isPending}
          className="bg-emerald-600 text-white text-sm font-medium px-5 py-2 rounded-lg hover:bg-emerald-700 disabled:opacity-60 transition-colors"
        >
          {isPending ? 'Saving…' : isEditing ? 'Save Changes' : 'Create Template'}
        </button>
      </div>
    </form>
  )
}
```

---

### Step 7 — Create the server actions

Create `app/email-templates/actions.ts`:

```ts
// app/email-templates/actions.ts
'use server'

import { createServerActionClient } from '@supabase/auth-helpers-nextjs'
import { cookies } from 'next/headers'
import { revalidatePath } from 'next/cache'

export async function addTemplate(formData: FormData) {
  const supabase = createServerActionClient({ cookies })

  const name = (formData.get('name') as string)?.trim()
  const subject = (formData.get('subject') as string)?.trim()
  const body = (formData.get('body') as string)?.trim()

  if (!name || !subject || !body) {
    return { error: 'Name, subject, and body are all required.' }
  }

  const { error } = await supabase
    .from('email_templates')
    .insert({ name, subject, body })

  if (error) {
    console.error('addTemplate error:', error)
    return { error: error.message }
  }

  revalidatePath('/email-templates')
}

export async function updateTemplate(id: string, formData: FormData) {
  const supabase = createServerActionClient({ cookies })

  const name = (formData.get('name') as string)?.trim()
  const subject = (formData.get('subject') as string)?.trim()
  const body = (formData.get('body') as string)?.trim()

  if (!name || !subject || !body) {
    return { error: 'Name, subject, and body are all required.' }
  }

  const { error } = await supabase
    .from('email_templates')
    .update({ name, subject, body })
    .eq('id', id)

  if (error) {
    console.error('updateTemplate error:', error)
    return { error: error.message }
  }

  revalidatePath('/email-templates')
}

export async function deleteTemplate(id: string) {
  const supabase = createServerActionClient({ cookies })

  const { error } = await supabase
    .from('email_templates')
    .delete()
    .eq('id', id)

  if (error) {
    console.error('deleteTemplate error:', error)
    return { error: error.message }
  }

  revalidatePath('/email-templates')
}
```

---

### Step 8 — The placeholder replacement function

The complete `applyPlaceholders` function — use this exact implementation anywhere you need placeholder substitution:

```ts
function applyPlaceholders(text: string, data: Record<string, string>): string {
  return text.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] ?? `{{${key}}}`)
}
```

How it works:

- `text.replace(regex, replacer)` — replaces every match of the regex in the string.
- `/\{\{(\w+)\}\}/g` — the regex. `\{\{` matches a literal `{{`. `(\w+)` captures the variable name (letters, digits, underscores). `\}\}` matches `}}`. The `g` flag matches all occurrences, not just the first.
- `(_, key)` — the replacer function receives the full match as the first argument (discarded with `_`) and the captured group as the second argument (`key`).
- `data[key] ?? \`{{${key}}}\`` — if the key exists in the data object, use its value. If not, leave the placeholder as-is. This way, unrecognised placeholders show visibly rather than disappearing.

Example:

```ts
applyPlaceholders('Hi {{first_name}}, welcome to {{company}}.', {
  first_name: 'Sarah',
  company: 'Acme Corp',
})
// → 'Hi Sarah, welcome to Acme Corp.'

applyPlaceholders('Hi {{first_name}}, your {{unknown}} is ready.', {
  first_name: 'Sarah',
})
// → 'Hi Sarah, your {{unknown}} is ready.'
```

---

### Step 9 — The clipboard copy button

The complete `onClick` handler for copying to clipboard:

```ts
async function handleCopy() {
  // Render placeholders before copying
  const renderedSubject = applyPlaceholders(template.subject, PREVIEW_DATA)
  const renderedBody = applyPlaceholders(template.body, PREVIEW_DATA)

  // Format as a complete email: subject line then body
  const textToCopy = `Subject: ${renderedSubject}\n\n${renderedBody}`

  // Write to clipboard — returns a Promise
  await navigator.clipboard.writeText(textToCopy)

  // Give the user visual feedback
  setCopied(true)
  setTimeout(() => setCopied(false), 2000)
}
```

`navigator.clipboard.writeText` requires a secure context (HTTPS or localhost). It will not work on a plain HTTP URL. On Vercel (HTTPS), it works without any configuration. Locally on `http://localhost:3000`, it also works.

---

### Step 10 — Wire the sidebar

Add the Email Templates link to your sidebar:

```tsx
// In your sidebar navLinks array
const navLinks = [
  { href: '/dashboard', label: 'Dashboard', icon: '🏠' },
  { href: '/contacts', label: 'Contacts', icon: '👥' },
  { href: '/companies', label: 'Companies', icon: '🏢' },
  { href: '/deals', label: 'Deals', icon: '💼' },
  { href: '/email-templates', label: 'Templates', icon: '📧' },
]
```

---

### Step 11 — Test the full module

1. Navigate to `/email-templates`. You should see the three seeded templates.
2. Click **Preview** on the Introduction Email. The rendered preview should show "Sarah" for `{{first_name}}`, "Acme Corp" for `{{company}}`, etc.
3. Click **Copy** on any template. Open a new browser tab, go to Gmail (or any text field), paste. The subject and body should appear correctly formatted.
4. Click **+ New Template**. Create a template with placeholder text. Verify the live preview updates as you type.
5. Click **Edit** on a template. Change the subject. Save. The list should update.
6. Click **Delete** on a template. Confirm the deletion. The template should disappear from the list.

---

### Step 12 — Commit

```
Directory: ~/clearcrm
Command:   git add . && git commit -m "feat: email templates — create, preview, copy to clipboard"
Expected:  [main xxxxxxx] feat: email templates — create, preview, copy to clipboard
```

---

## Practical workflow

**Why controlled state for the form fields.** The live preview needs to react to every keystroke. Uncontrolled inputs (with `ref` and `formRef.current.value`) cannot drive live previews because you cannot observe their changes without event listeners. Controlled inputs (`value={body}` and `onChange={(e) => setBody(e.target.value)}`) update state on every keystroke, which triggers a re-render, which updates the preview automatically.

**Why `<pre>` for the email body preview.** Email bodies contain line breaks. A `<p>` tag collapses line breaks into spaces. A `<pre>` tag with `whitespace-pre-wrap` preserves line breaks exactly as stored, and wraps long lines within the container. `font-sans` resets the monospace font that `<pre>` applies by default.

**The 2-second copy confirmation.** `setCopied(true)` changes the button to show "✓ Copied". `setTimeout(() => setCopied(false), 2000)` resets it after 2 seconds. This pattern gives the user clear confirmation without any permanent UI change. No toast library needed.

**Why the preview pane uses sample data.** A real preview would require a contact selection. That is scope creep. Sample data (`PREVIEW_DATA`) shows the user exactly how the template will look without adding form complexity. The user can imagine "Sarah = my contact's name". The goal is to show that placeholders are working, not to render a 100% accurate email.

---

## Common mistakes

**Mistake: The copy button throws a `DOMException: Not allowed` error.**
Cause: `navigator.clipboard.writeText` requires user interaction. If called outside of a direct event handler (e.g., inside a `useEffect` or a delayed callback), browsers block it.
Fix: Ensure `handleCopy` is called directly from an `onClick` prop — not via `setTimeout` or inside an async chain that the browser does not recognise as user-initiated.

**Mistake: Placeholders in the preview show as `{{unknown}}` even when they should be replaced.**
Cause: The key in `PREVIEW_DATA` does not match the placeholder exactly. Regex match is case-sensitive and requires exact word-character matching.
Fix: Check that `PREVIEW_DATA` keys exactly match what is written in the template. `{{first_name}}` → key must be `first_name`. `{{firstName}}` would not match.

**Mistake: The live preview does not update when typing.**
Cause: The form fields are uncontrolled (using `defaultValue` instead of `value` and `onChange`).
Fix: Switch to controlled inputs. Each field needs `value={stateVar}` and `onChange={(e) => setStateVar(e.target.value)}`.

**Mistake: Deleting a template does not remove it from the list.**
Cause: `router.refresh()` is not being called after `deleteTemplate` resolves, or the server component data is cached.
Fix: Call `router.refresh()` in `handleDelete` after `deleteTemplate` resolves. Also confirm `revalidatePath('/email-templates')` is called inside the server action.

**Mistake: The INSERT SQL for seed data fails with a syntax error.**
Cause: The multi-row INSERT uses single-quoted strings containing apostrophes. In SQL, a single quote inside a single-quoted string must be escaped as two single quotes (`''`).
Fix: Everywhere the template body contains an apostrophe (e.g., `I've`, `We've`), it must appear as `I''ve` in the SQL. This is already correct in the SQL provided in Step 2 — do not modify it.

---

## Your turn

1. Add a "Use in Deal" button on each template card. Clicking it should pre-fill the email body in a modal, let the user select a deal, and then log an "email" activity on that deal using the `addActivity` server action from Day 13.
2. Add a character count below the body textarea (e.g., "312 characters").
3. Add a search/filter input above the template list that filters templates by name in real time (client-side, no new Supabase query needed).

---

## Prompt / Template / Checklist pack

### Claude Code prompt — Build the email templates module

```
I'm building ClearCRM with Next.js 14 App Router, Supabase, and Tailwind CSS.

Build the email templates module at /email-templates. Requirements below.

DATABASE: The email_templates table exists with: id (UUID), name (TEXT), subject (TEXT), body (TEXT), created_at (TIMESTAMPTZ). RLS is enabled with policies for authenticated users to SELECT/INSERT/UPDATE/DELETE.

FILES TO CREATE:

1. app/email-templates/page.tsx — SERVER COMPONENT
- Fetch all templates: supabase.from('email_templates').select('*').order('created_at', { ascending: true })
- Show heading "Email Templates" and a template count
- Render a TemplateList client component, passing templates as initialTemplates prop

2. app/email-templates/TemplateList.tsx — CLIENT COMPONENT ('use client')
- Manages showForm (boolean) and editingTemplate (Template | null) state
- Renders "+ New Template" button (top right, emerald filled)
- Renders a TemplateCard for each template, passing onEdit callback
- When showForm is true: renders a modal overlay with a TemplateForm inside
- Modal: fixed inset-0 bg-black/40, centered white card (max-w-2xl, max-h-[90vh] overflow-y-auto)

3. app/email-templates/TemplateCard.tsx — CLIENT COMPONENT ('use client')
- Props: template (id, name, subject, body), onEdit callback
- Displays: template name (font-semibold), subject (text-sm text-gray-500), first 120 chars of body (text-sm text-gray-400 line-clamp-2)
- 4 action buttons: "Preview" (toggle), "Copy" (copy to clipboard), "Edit" (calls onEdit), "Delete" (calls deleteTemplate server action then router.refresh())
- Preview panel: shown below the card header when showPreview=true. Shows rendered subject and body with placeholders replaced using sample data: { first_name: 'Sarah', company: 'Acme Corp', deal_title: 'CRM Implementation', deal_value: 'R 45,000', your_name: 'John Doe', your_company: 'ClearCRM Ltd' }
- Copy button: calls navigator.clipboard.writeText('Subject: ' + renderedSubject + '\n\n' + renderedBody), then shows '✓ Copied' for 2 seconds
- Delete button: confirm() dialog first, then call deleteTemplate, then router.refresh()

4. app/email-templates/TemplateForm.tsx — CLIENT COMPONENT ('use client')
- Props: template? (null for create, Template for edit), onSuccess callback
- Controlled state for name, subject, body fields
- Layout: 2 columns on md+. Left column: name input, subject input, body textarea (rows=12, font-mono), placeholder reference box showing available placeholders. Right column: live preview panel that updates as user types.
- applyPlaceholders function: text.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] ?? '{{' + key + '}}')
- Live preview: apply applyPlaceholders to subject and body with the sample data above. Show subject as bold, body in <pre> with whitespace-pre-wrap and font-sans.
- On submit: if editing call updateTemplate(template.id, formData), else call addTemplate(formData). On success: router.refresh() then onSuccess().

5. app/email-templates/actions.ts — 'use server'
- addTemplate(formData): insert into email_templates, revalidatePath('/email-templates'), return { error } on failure
- updateTemplate(id, formData): update email_templates where id=id, revalidatePath, return { error } on failure
- deleteTemplate(id): delete from email_templates where id=id, revalidatePath, return { error } on failure
- All three functions validate that name, subject, and body are non-empty strings

Tailwind only. No UI library. No rich text editor. Show complete file contents for all 5 files.
```

---

### Starter email template pack — ready to copy

---

**Template 1: Introduction Email**

Subject:
```
Introduction: {{your_name}} from {{your_company}}
```

Body:
```
Hi {{first_name}},

I hope this finds you well. My name is {{your_name}} and I work at {{your_company}}.

We help businesses like {{company}} with [your value proposition in one sentence].

I'd love to schedule a quick 15-minute call to see if there's a fit. Would you have time this week or next?

Looking forward to connecting.

Best regards,
{{your_name}}
{{your_company}}
```

When to use: First contact with a new lead. They have not spoken to anyone at your company before.

---

**Template 2: Follow-Up Email**

Subject:
```
Following up — {{your_company}}
```

Body:
```
Hi {{first_name}},

I sent you an email a few days ago and wanted to follow up in case it got buried.

We've been helping companies similar to {{company}} to [specific outcome]. I think there could be a good opportunity for us to work together.

Would you be open to a brief conversation this week?

Best,
{{your_name}}
```

When to use: 3–5 business days after the introduction email with no reply. Send once. If no reply after this, move the deal to Lost and set a reminder to try again in 6 weeks.

---

**Template 3: Proposal Email**

Subject:
```
Proposal for {{company}} — {{deal_title}}
```

Body:
```
Hi {{first_name}},

Thank you for our conversation earlier. As discussed, please find our proposal for {{deal_title}} attached.

Proposal summary:
- Scope: [brief scope description]
- Investment: {{deal_value}}
- Timeline: [estimated timeline]
- Next step: [what you need from them to proceed]

Please review and let me know if you have any questions. I'm available for a call to walk you through the details.

Looking forward to working with you.

Best regards,
{{your_name}}
{{your_company}}
```

When to use: After a qualification call when the prospect has asked for a formal quote or proposal. Attach your actual proposal document. Fill in the scope, timeline, and next step fields before sending.

---

### Placeholder syntax reference

| Placeholder | Replace with | Example |
|-------------|-------------|---------|
| `{{first_name}}` | Contact's first name | Sarah |
| `{{company}}` | Contact's company name | Acme Corp |
| `{{deal_title}}` | Name of the deal | Website Redesign |
| `{{deal_value}}` | Formatted deal value | R 45,000 |
| `{{your_name}}` | Your own name | John Doe |
| `{{your_company}}` | Your company name | ClearCRM Ltd |

**Rules for placeholders:**
- Always use double curly braces: `{{placeholder}}`
- Use underscores, not spaces: `{{first_name}}` not `{{first name}}`
- Case-sensitive: `{{First_Name}}` will not match `{{first_name}}`
- Unrecognised placeholders are left as-is in the output — they will be visible in the copy, reminding you to fill them in manually
