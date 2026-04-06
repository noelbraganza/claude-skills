---
name: us-tech-stack
description: >
  US tech stack reference for Next.js + Supabase + Vercel projects. Covers
  architecture decisions, service nuances, known gotchas, and integration
  patterns. Trigger on: Next.js, Supabase, Vercel, Resend, stack question,
  how do I, architecture, deploy, email, auth, cron, RLS.
---

# US Tech Stack — Next.js + Supabase + Vercel

A reference for **Next.js 14 App Router** applications deployed on **Vercel**, backed by **Supabase** (Postgres + Auth + Realtime), with transactional email via **Resend**.

---

## Stack at a glance

| Layer | Service | Notes |
|---|---|---|
| Framework | Next.js 14 (App Router) | Server Components, API Routes, Middleware |
| Database | Supabase (Postgres) | Pick the region closest to your users |
| Auth | Supabase Auth | Magic link (OTP), PKCE flow |
| Realtime | Supabase Realtime | Live data subscriptions |
| Hosting | Vercel | Auto-deploy from GitHub main |
| Cron | Vercel Cron | Defined in `vercel.json` |
| Email | Resend | Transactional only |
| Language | TypeScript (strict) | |
| Styling | Tailwind CSS | |

---

## Next.js 14 App Router

### Server vs Client Components

- **Default = Server Component.** Only add `'use client'` when you need browser APIs, hooks, or interactivity.
- Server Components can `async/await` directly — no `useEffect` data fetching.
- Pass serialisable props only from Server → Client Components. No functions, no Supabase clients.
- `createClient()` from `@/lib/supabase/server` is for Server Components and API routes. `createClient()` from `@/lib/supabase/client` is for browser components.

### API Routes

- File: `app/api/[path]/route.ts`. Export named functions: `GET`, `POST`, `PATCH`, `DELETE`.
- Use `NextRequest`/`NextResponse` for typed request/response.
- Return `NextResponse.json(data, { status: N })` — not `Response.json()` for consistency.

### Middleware

- `middleware.ts` at project root. Runs on every request matching `config.matcher`.
- Keep it lean — DB queries in middleware add latency to every page load.
- Common pattern: check `is_admin` from profiles for `/admin` routes.

### Dynamic routes

- `app/admin/[resource]/[id]/` — `params.id` is a string, always.
- Bracket paths in shell (zsh) must be quoted: `git add 'app/api/admin/[id]/route.ts'`

---

## Supabase

### Client types — use the right one

| Client | File | When |
|---|---|---|
| Session-based (anon key + cookies) | `lib/supabase/server.ts` | Server Components, API routes with user session |
| Browser (anon key) | `lib/supabase/client.ts` | Client Components only |
| Service role | `lib/supabase/service.ts` | Cron jobs, server-to-server, bypasses RLS |

**Rule:** If the code runs server-side but has no user session (cron, background jobs, lib functions called from cron), use `createServiceClient()`. Never expose the service role key to the browser — it must NEVER use the `NEXT_PUBLIC_` prefix.

```ts
// lib/supabase/service.ts
import { createClient } from '@supabase/supabase-js'

export function createServiceClient() {
  return createClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.SUPABASE_SERVICE_ROLE_KEY!
  )
}
```

### RLS (Row Level Security)

- Enable RLS on every table — the anon key is public, RLS is your only data protection.
- **Admin role pattern:**
  - `profiles.is_admin boolean` column (default false)
  - `is_admin()` Postgres function (SECURITY DEFINER, reads from profiles)
  - RLS policies use `public.is_admin()` for admin-only write access
  - API utility: `requireAdmin()` in `lib/admin.ts` — returns `NextResponse | null`
- **Service role bypasses RLS entirely.** Use it for cron jobs and any server code without a user session.
- Multiple SELECT policies on the same table are OR'd — a row is visible if any policy matches.

**Admin gate pattern:**
```ts
// lib/admin.ts
export async function requireAdmin(): Promise<NextResponse | null> {
  const supabase = createClient()
  const { data: { user } } = await supabase.auth.getUser()
  if (!user) return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
  const { data: profile } = await supabase.from('profiles').select('is_admin').eq('id', user.id).single()
  if (!profile?.is_admin) return NextResponse.json({ error: 'Forbidden' }, { status: 403 })
  return null
}

// Usage in any /api/admin/* route:
const authError = await requireAdmin()
if (authError) return authError
```

### Migrations

- Apply via Supabase MCP: `mcp__claude_ai_Supabase__apply_migration`
- Project ID: found in Supabase Dashboard → Project Settings → General
- Choose the Supabase region closest to your users (e.g. eu-north-1 for Europe, us-east-1 for US East, ap-southeast-1 for Asia)
- Never drop-and-recreate in production — use `ALTER TABLE`, `ADD COLUMN`, `CREATE POLICY`, `DROP POLICY`.
- Always name migrations in snake_case (e.g. `add_published_to_posts`).

### Auth

- Magic link (OTP) via `supabase.auth.signInWithOtp()`. No passwords.
- PKCE flow — browser client uses default (no explicit `flowType` needed since `@supabase/ssr`).
- After magic link click: `auth/callback` route exchanges code for session → cookie set.
- Profile auto-created by trigger on `auth.users` insert.
- `auth.uid()` in RLS = current user's UUID. Returns null for unauthenticated/cron requests.

### Realtime

- Subscribe via browser client: `supabase.channel('...').on('postgres_changes', ...)`.
- Requires RLS SELECT policy — Realtime respects RLS.
- Use for live dashboards, notifications, collaborative features.

---

## Vercel

### Deployment

- Auto-deploys on push to `main` from GitHub.
- To redeploy with updated env vars (without a new commit): `vercel --prod --yes` from project root.

### Environment variables

```
NEXT_PUBLIC_SUPABASE_URL          — public, browser-safe
NEXT_PUBLIC_SUPABASE_ANON_KEY     — public, browser-safe (RLS protects data)
NEXT_PUBLIC_BASE_URL              — https://your-domain.com
SUPABASE_SERVICE_ROLE_KEY         — server-only, never NEXT_PUBLIC_
RESEND_API_KEY                    — server-only
CRON_SECRET                       — server-only, must be a real random value
```

**CRON_SECRET gotcha:** `.env` files do not evaluate shell commands. `CRON_SECRET=openssl rand -hex 32` stores the literal string, not the output. Always run `openssl rand -hex 32` in terminal and paste the result.

### Cron jobs

Defined in `vercel.json`:
```json
{ "crons": [{ "path": "/api/cron/your-job", "schedule": "0 * * * *" }] }
```
- Vercel calls the route with `Authorization: Bearer <CRON_SECRET>`.
- Routes check: `const secret = process.env.CRON_SECRET; if (!secret || request.headers.get('Authorization') !== \`Bearer ${secret}\`) ...`
- The `!secret` guard prevents an empty/missing secret from accepting all requests.
- **Pro plan timeout**: 60s. Hobby: 10s. Keep batch operations within the limit.

### Security headers

Set in `next.config.mjs` via `async headers()`:
- `X-Frame-Options: DENY`
- `X-Content-Type-Options: nosniff`
- `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Content-Security-Policy` (script-src, connect-src includes `*.supabase.co`)
- CORS on `/api/*`: restricted to `NEXT_PUBLIC_BASE_URL`

---

## Resend

- Rate limit: **5 emails/second**. Batch sends use `BATCH_SIZE = 5` with 1s delay between batches.
- From address: `noreply@your-domain.com` — must be a verified domain in Resend.
- Deduplication: track a `email_sent boolean` column per record — check before sending, mark true after.
- Test emails: send to a single address before broadcast.

**Basic send:**
```ts
import { Resend } from 'resend'

const resend = new Resend(process.env.RESEND_API_KEY)

await resend.emails.send({
  from: 'noreply@your-domain.com',
  to: 'user@example.com',
  subject: 'Hello',
  html: '<p>Hello world</p>',
})
```

---

## Admin role system

```
profiles.is_admin (boolean, default false)
    ↓
is_admin() Postgres function (SECURITY DEFINER)
    ↓ used in
RLS policies on admin-only tables
    ↓
requireAdmin() in lib/admin.ts
    ↓ used in
All /api/admin/* routes
    ↓
middleware.ts checks is_admin for /admin/* pages
```

To grant admin to a user:
```sql
UPDATE profiles SET is_admin = true
WHERE id = (SELECT id FROM auth.users WHERE email = 'user@example.com');
```

---

## Known gotchas

| Gotcha | Fix |
|---|---|
| zsh expands `[id]` as a glob in shell commands | Quote paths: `git add 'app/api/admin/[id]/route.ts'` |
| `createClient()` in cron = no session = RLS blocks everything | Use `createServiceClient()` for all code called from cron |
| Upsert requires both INSERT + UPDATE policies | Add both policies or use service role |
| Supabase anon key is public — protect data with RLS, not key secrecy | Every table must have RLS enabled with correct policies |
| `NEXT_PUBLIC_*` vars are bundled into client JS | Never put secrets in `NEXT_PUBLIC_*` vars |
| `dangerouslySetInnerHTML` with user/admin HTML | Always sanitize with `sanitize-html` before rendering |
| Multiple SELECT RLS policies = OR logic | A row matching ANY select policy is visible |
| Empty `CRON_SECRET` env var bypasses auth | Add `!secret` guard before comparing |
