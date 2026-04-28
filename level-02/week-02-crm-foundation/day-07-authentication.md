# Day 7: Authentication — Login, Signup, and Protected Routes

## Outcome artifact
By the end of this day ClearCRM has a working login page at `/login`, a signup page at `/signup`, a middleware file that redirects unauthenticated users to `/login`, and a sign-out button in the sidebar — all powered by Supabase Auth using the `@supabase/ssr` package.

## The core idea

Next.js 14 App Router runs code in two places: the server (Node.js, no browser) and the client (the browser). Supabase Auth needs to read and write session cookies in both places. The `@supabase/ssr` package solves this by providing two separate Supabase client creators: one for server components and server actions, one for browser components. You will create a thin wrapper file for each.

The middleware file (`middleware.ts`) runs on the Edge — before any page renders. It intercepts every request, checks whether the user has a valid session cookie, and either allows the request through or redirects to `/login`. This is what makes routes "protected" without you having to add auth checks to every single page. The middleware also silently refreshes the session token when it is near expiry, which keeps users logged in without forcing them to re-enter credentials.

Supabase Auth stores sessions as cookies when you use `@supabase/ssr`. The old package (`@supabase/auth-helpers-nextjs`) stored sessions in localStorage, which does not work on the server. If you search old tutorials and see `@supabase/auth-helpers-nextjs`, those instructions are outdated. Always use `@supabase/ssr` for Next.js 14 App Router projects.

**Note on Context7:** If you are using Claude Code with the Context7 plugin installed, it will automatically inject the current Supabase `@supabase/ssr` documentation into your Claude prompts. This means Claude will have access to the latest API — you do not need to paste doc links. If you are not using Context7, the complete code below is self-contained.

## Step-by-step walkthrough

### Step 1 — Install @supabase/ssr

Open the VS Code integrated terminal (`` Ctrl+` ``). Make sure you are in the `clearcrm/` directory. Run:

```
npm install @supabase/ssr
```

Directory: `clearcrm/`
Expected output:
```
added 2 packages, and audited 315 packages in 3s
found 0 vulnerabilities
```

---

### Step 2 — Create the server Supabase client

In VS Code Explorer panel, create a new folder: `utils/supabase/`. Inside it, create a file named `server.ts`.

File path: `clearcrm/utils/supabase/server.ts`

```typescript
import { createServerClient } from '@supabase/ssr'
import { cookies } from 'next/headers'

export function createClient() {
  const cookieStore = cookies()

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return cookieStore.getAll()
        },
        setAll(cookiesToSet) {
          try {
            cookiesToSet.forEach(({ name, value, options }) =>
              cookieStore.set(name, value, options)
            )
          } catch {
            // setAll called from a Server Component — safe to ignore
          }
        },
      },
    }
  )
}
```

---

### Step 3 — Create the browser Supabase client

In `utils/supabase/`, create a file named `client.ts`.

File path: `clearcrm/utils/supabase/client.ts`

```typescript
import { createBrowserClient } from '@supabase/ssr'

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  )
}
```

---

### Step 4 — Create the middleware

In VS Code Explorer, create a file named `middleware.ts` in the project root — the same level as `package.json`, NOT inside the `app/` folder.

File path: `clearcrm/middleware.ts`

```typescript
import { createServerClient } from '@supabase/ssr'
import { NextResponse, type NextRequest } from 'next/server'

export async function middleware(request: NextRequest) {
  let supabaseResponse = NextResponse.next({
    request,
  })

  const supabase = createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        getAll() {
          return request.cookies.getAll()
        },
        setAll(cookiesToSet) {
          cookiesToSet.forEach(({ name, value }) =>
            request.cookies.set(name, value)
          )
          supabaseResponse = NextResponse.next({ request })
          cookiesToSet.forEach(({ name, value, options }) =>
            supabaseResponse.cookies.set(name, value, options)
          )
        },
      },
    }
  )

  // Refresh session — must call getUser() to keep session alive
  const {
    data: { user },
  } = await supabase.auth.getUser()

  // If no user and not already on an auth page, redirect to /login
  if (
    !user &&
    !request.nextUrl.pathname.startsWith('/login') &&
    !request.nextUrl.pathname.startsWith('/signup')
  ) {
    const url = request.nextUrl.clone()
    url.pathname = '/login'
    return NextResponse.redirect(url)
  }

  return supabaseResponse
}

export const config = {
  matcher: [
    '/((?!_next/static|_next/image|favicon.ico|.*\\.(?:svg|png|jpg|jpeg|gif|webp)$).*)',
  ],
}
```

---

### Step 5 — Create the login page

In VS Code Explorer, create the folder `app/login/` and inside it create `page.tsx`.

File path: `clearcrm/app/login/page.tsx`

```typescript
'use client'

import { useState } from 'react'
import { useRouter } from 'next/navigation'
import Link from 'next/link'
import { createClient } from '@/utils/supabase/client'

export default function LoginPage() {
  const router = useRouter()
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [error, setError] = useState<string | null>(null)
  const [loading, setLoading] = useState(false)

  async function handleLogin(e: React.FormEvent) {
    e.preventDefault()
    setLoading(true)
    setError(null)

    const supabase = createClient()
    const { error } = await supabase.auth.signInWithPassword({ email, password })

    if (error) {
      setError(error.message)
      setLoading(false)
      return
    }

    router.push('/')
    router.refresh()
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="w-full max-w-md">
        <div className="bg-white rounded-2xl shadow-sm border border-gray-200 p-8">
          <h1 className="text-2xl font-bold text-gray-900 mb-1">Sign in to ClearCRM</h1>
          <p className="text-sm text-gray-500 mb-6">Enter your email and password to continue.</p>

          <form onSubmit={handleLogin} className="space-y-4">
            <div>
              <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-1">
                Email
              </label>
              <input
                id="email"
                type="email"
                required
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
                placeholder="you@company.com"
              />
            </div>

            <div>
              <label htmlFor="password" className="block text-sm font-medium text-gray-700 mb-1">
                Password
              </label>
              <input
                id="password"
                type="password"
                required
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
                placeholder="••••••••"
              />
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
              {loading ? 'Signing in…' : 'Sign in'}
            </button>
          </form>

          <p className="mt-4 text-center text-sm text-gray-500">
            No account?{' '}
            <Link href="/signup" className="text-emerald-600 font-medium hover:underline">
              Sign up
            </Link>
          </p>
        </div>
      </div>
    </div>
  )
}
```

---

### Step 6 — Create the signup page

Create the folder `app/signup/` and inside it create `page.tsx`.

File path: `clearcrm/app/signup/page.tsx`

```typescript
'use client'

import { useState } from 'react'
import { useRouter } from 'next/navigation'
import Link from 'next/link'
import { createClient } from '@/utils/supabase/client'

export default function SignupPage() {
  const router = useRouter()
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [error, setError] = useState<string | null>(null)
  const [loading, setLoading] = useState(false)

  async function handleSignup(e: React.FormEvent) {
    e.preventDefault()
    setLoading(true)
    setError(null)

    const supabase = createClient()
    const { error } = await supabase.auth.signUp({ email, password })

    if (error) {
      setError(error.message)
      setLoading(false)
      return
    }

    // After sign-up, redirect to login with success message
    router.push('/login?message=Check your email to confirm your account')
  }

  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="w-full max-w-md">
        <div className="bg-white rounded-2xl shadow-sm border border-gray-200 p-8">
          <h1 className="text-2xl font-bold text-gray-900 mb-1">Create your account</h1>
          <p className="text-sm text-gray-500 mb-6">Start managing your contacts and deals.</p>

          <form onSubmit={handleSignup} className="space-y-4">
            <div>
              <label htmlFor="email" className="block text-sm font-medium text-gray-700 mb-1">
                Email
              </label>
              <input
                id="email"
                type="email"
                required
                value={email}
                onChange={(e) => setEmail(e.target.value)}
                className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
                placeholder="you@company.com"
              />
            </div>

            <div>
              <label htmlFor="password" className="block text-sm font-medium text-gray-700 mb-1">
                Password
              </label>
              <input
                id="password"
                type="password"
                required
                minLength={6}
                value={password}
                onChange={(e) => setPassword(e.target.value)}
                className="w-full px-3 py-2 border border-gray-300 rounded-lg text-sm focus:outline-none focus:ring-2 focus:ring-emerald-500 focus:border-transparent"
                placeholder="At least 6 characters"
              />
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
              {loading ? 'Creating account…' : 'Create account'}
            </button>
          </form>

          <p className="mt-4 text-center text-sm text-gray-500">
            Already have an account?{' '}
            <Link href="/login" className="text-emerald-600 font-medium hover:underline">
              Sign in
            </Link>
          </p>
        </div>
      </div>
    </div>
  )
}
```

---

### Step 7 — Update the root layout to reflect auth state

Open `app/layout.tsx`. The root layout wraps every page. Update it so the sidebar only shows for authenticated users. This is the server component version — it reads the session on the server before rendering.

File path: `clearcrm/app/layout.tsx`

Replace the contents of your existing `layout.tsx` with:

```typescript
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'
import { createClient } from '@/utils/supabase/server'
import Sidebar from '@/components/Sidebar'

const inter = Inter({ subsets: ['latin'] })

export const metadata: Metadata = {
  title: 'ClearCRM',
  description: 'Simple, fast CRM for growing teams',
}

export default async function RootLayout({
  children,
}: {
  children: React.ReactNode
}) {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()

  return (
    <html lang="en">
      <body className={inter.className}>
        {user ? (
          <div className="flex h-screen overflow-hidden">
            <Sidebar />
            <main className="flex-1 overflow-y-auto bg-white">
              {children}
            </main>
          </div>
        ) : (
          <>{children}</>
        )}
      </body>
    </html>
  )
}
```

---

### Step 8 — Add the sign-out button to the sidebar

Open your sidebar component. If Bolt generated it as `components/Sidebar.tsx`, open that file. Add a sign-out button at the bottom of the sidebar using a server action.

Create a new file: `app/actions/auth.ts`

File path: `clearcrm/app/actions/auth.ts`

```typescript
'use server'

import { createClient } from '@/utils/supabase/server'
import { redirect } from 'next/navigation'

export async function signOut() {
  const supabase = createClient()
  await supabase.auth.signOut()
  redirect('/login')
}
```

Now update your `components/Sidebar.tsx` to include the sign-out button. Add this at the bottom of your sidebar JSX, inside the sidebar container:

```typescript
import { signOut } from '@/app/actions/auth'

// Inside your sidebar component JSX, at the bottom:
<form action={signOut}>
  <button
    type="submit"
    className="w-full flex items-center gap-3 px-4 py-2 text-sm text-gray-400 hover:text-white hover:bg-white/10 rounded-lg transition-colors mt-auto"
  >
    Sign out
  </button>
</form>
```

The `action={signOut}` pattern calls the server action directly when the form is submitted. No JavaScript event handler needed — the browser handles form submission natively.

---

### Step 9 — Disable email confirmation for development

By default, Supabase sends a confirmation email before activating a new account. During development, disable this so you can test without checking email every time.

In Supabase dashboard: go to **Authentication → Providers → Email**. Toggle off **Confirm email**. Save.

You can re-enable this before you launch to real users.

---

### Step 10 — Test the auth flow

1. Run `npm run dev` in the terminal
2. Open `http://localhost:3000` — the middleware should immediately redirect you to `http://localhost:3000/login`
3. Click **Sign up** link — you should reach `/signup`
4. Enter an email and password (at least 6 characters) — click **Create account**
5. You should be redirected to `/login` (or directly to the dashboard if you wired it to auto-login)
6. Sign in with the same credentials
7. You should see the ClearCRM dashboard with the sidebar
8. Click **Sign out** — you should be redirected to `/login`

---

### Step 11 — Commit

```
git add . && git commit -m "feat: add Supabase auth — login, signup, protected routes"
```

Directory: `clearcrm/`

---

## Practical workflow

1. Run `npm install @supabase/ssr` in `clearcrm/`
2. Create `utils/supabase/server.ts` with the server client
3. Create `utils/supabase/client.ts` with the browser client
4. Create `middleware.ts` in project root with session refresh and redirect logic
5. Create `app/login/page.tsx` with email/password form and signInWithPassword handler
6. Create `app/signup/page.tsx` with email/password form and signUp handler
7. Update `app/layout.tsx` to check user session and conditionally show sidebar
8. Create `app/actions/auth.ts` with signOut server action
9. Add sign-out button to sidebar using `<form action={signOut}>`
10. Disable email confirmation in Supabase Auth settings (dev only)
11. Test: localhost redirects to login → sign up → sign in → see sidebar → sign out → back to login
12. Commit with `git add . && git commit -m "feat: add Supabase auth — login, signup, protected routes"`

---

## Common mistakes

**1. Auth works locally but not on Vercel — redirects loop or the session is lost after login**

The middleware file must be at `clearcrm/middleware.ts` — the project root. If it is inside the `app/` folder or inside `src/`, Next.js will not pick it up and no session checking happens. Also verify Vercel has both `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` set. After adding env vars to Vercel, trigger a new deployment (push a small change) — Vercel does not automatically rebuild when you add variables.

**2. `createClient()` in `utils/supabase/server.ts` throws "cookies() should be awaited" or similar error**

This happens when Next.js 15 is installed instead of Next.js 14. The `cookies()` function is synchronous in Next.js 14 but became async in Next.js 15. Check your `package.json` — if `"next"` version is `"15.x.x"`, you either need to downgrade to 14 or update the server client to `await cookies()`. The scaffold from Bolt should be on 14 — if it is not, run `npm install next@14`.

**3. Signing up creates the user in Supabase Auth but the session is not set — user is sent back to login instead of the dashboard**

When email confirmation is enabled in Supabase, `signUp()` does not create a session — it sends an email. The user has to click the link before they can sign in. Disable email confirmation in Supabase dashboard (Authentication → Providers → Email → toggle off "Confirm email") during development. Re-enable it before going to production with real users.

---

## Your turn

Start the dev server. Open `http://localhost:3000`. The browser should immediately redirect to `/login`. Sign up with a test email and password. Sign in. Verify the sidebar is visible. Click sign out.

Expected output: Navigating to `http://localhost:3000` immediately shows `http://localhost:3000/login` in the URL bar. After signing in, the URL shows `http://localhost:3000` and the dark sidebar is visible on the left.

**Failure state:** You sign in but you are redirected back to `/login` in a loop.
Fix: Open the browser DevTools (F12) → Application tab → Cookies. Look for a cookie from `supabase.co`. If no cookies are set after signing in, the `setAll` function in your server client is not being called correctly. Double-check that your `utils/supabase/server.ts` exactly matches the code in Step 2. Pay special attention to the `setAll` method — a typo there prevents session cookies from being written.

**Failure state:** The middleware file exists but `/` does not redirect to `/login`.
Fix: Next.js requires `middleware.ts` to be in the project root — the same directory as `next.config.js`. Open VS Code Explorer and confirm `middleware.ts` is at the top level of the file tree, not inside any folder. If it is inside `app/`, drag it to the project root.

---

## Prompt / Template / Checklist pack

### Claude Code prompt — add Supabase SSR auth to Next.js 14

Use this prompt in Claude Code to generate the full auth setup in one pass. Paste it into the Claude Code chat:

```
Add Supabase Auth to this Next.js 14 App Router project using the @supabase/ssr package.

Create these files:
1. utils/supabase/server.ts — server client using createServerClient from @supabase/ssr, reading/writing cookies via next/headers cookies()
2. utils/supabase/client.ts — browser client using createBrowserClient from @supabase/ssr
3. middleware.ts in the project root — refresh session with supabase.auth.getUser(), redirect unauthenticated users to /login, exclude /login and /signup from the redirect, use the matcher config to exclude static assets
4. app/login/page.tsx — client component with a clean email/password form, calls supabase.auth.signInWithPassword(), shows inline error state, redirects to / on success, links to /signup
5. app/signup/page.tsx — client component with email/password form, calls supabase.auth.signUp(), redirects to /login after signup
6. app/actions/auth.ts — server action for signOut that calls supabase.auth.signOut() and redirects to /login
7. Update app/layout.tsx — async server component that calls supabase.auth.getUser(), conditionally wraps children in the sidebar layout only when a user session exists

Use Tailwind CSS for all styles. Use emerald-500 (#10B981) as the primary accent color. The sidebar background is #0F172A.

Do not use @supabase/auth-helpers-nextjs — use @supabase/ssr only.
```

---

### Auth flow diagram

```
User visits any protected route (e.g. /)
        |
        v
middleware.ts intercepts the request
        |
        v
supabase.auth.getUser() — checks session cookie
        |
        +--- Session found ---------> Request passes through
        |                                      |
        |                                      v
        |                             Page renders with sidebar
        |
        +--- No session -----------> NextResponse.redirect('/login')
                                              |
                                              v
                                     /login page renders
                                              |
                                              v
                               User enters email + password
                                              |
                                              v
                              supabase.auth.signInWithPassword()
                                              |
                              +-- Error ----> Show error message
                              |
                              +-- Success --> Session cookie written
                                              |
                                              v
                                     router.push('/') + router.refresh()
                                              |
                                              v
                              middleware runs again — session found
                                              |
                                              v
                                    Dashboard renders with sidebar
                                              |
                         User clicks Sign out button (form action)
                                              |
                                              v
                                    signOut() server action runs
                                              |
                                              v
                              supabase.auth.signOut() — clears cookie
                                              |
                                              v
                                     redirect('/login')
```
