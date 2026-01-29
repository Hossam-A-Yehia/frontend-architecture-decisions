# 04. Data Fetching Strategy (Next.js)

## Problem

In Next.js (App Router + React Server Components), you can fetch data in multiple places: server components, client components, route handlers, or client-side libraries. The wrong choice leads to waterfall requests, fragile caching, or unnecessary client complexity.

## Options

### A) Server Components (default)
- Fetch on the server where data is needed.
- Benefits: no client bundle cost, instant access to server secrets, easier caching, less JS.
- Tools: `fetch` with Next.js caching, `revalidate`, `cache: 'no-store'`, segment-level streaming.

### B) Client Components
- Fetch in the browser for highly interactive or user-local state.
- Benefits: incremental hydration, optimistic UI, background refetching.
- Tools: TanStack Query (React Query), SWR.

### C) Route Handlers / Server Actions
- Consolidate server logic in `app/api/...` or server actions.
- Benefits: reuse across multiple pages/components; centralized authorization and validation.

## Decision

- Default to **Server Components fetching**. Fetch close to where the data is rendered. Let Next.js handle caching and streaming by default.
- Use **client fetching** when the state is truly client-driven (e.g., live filtering, user-local drafts, optimistic mutations).
- Extract to **route handlers** or **server actions** when multiple consumers need the same server logic or when you need a stable API boundary.

## Conventions

- Co-locate fetch with the component that renders the data (prefer server components).
- Control caching explicitly:
  - Stable public data: `revalidate: <seconds>`
  - Per-user or sensitive data: `cache: 'no-store'`
  - Invalidate via `revalidateTag`/`revalidatePath` when appropriate
- Prefer streaming for slow data sources using Suspense boundaries.
- Avoid fetching in `useEffect` for initial render data that could be fetched on the server.

## Examples

### Server component with cache and revalidation
```tsx
// app/users/page.tsx (Server Component)
export const revalidate = 60; // ISR per route or segment

async function getUsers() {
  const res = await fetch('https://api.example.com/users', { next: { revalidate: 60 } });
  if (!res.ok) throw new Error('Failed to fetch users');
  return res.json();
}

export default async function UsersPage() {
  const users = await getUsers();
  return (
    <ul>
      {users.map((u: any) => (
        <li key={u.id}>{u.name}</li>
      ))}
    </ul>
  );
}
```

### Client fetching for interactive views
```tsx
// features/users/components/UserSearch.tsx (Client Component)
'use client';
import { useState } from 'react';
import { useQuery } from '@tanstack/react-query';

export function UserSearch() {
  const [q, setQ] = useState('');
  const { data, isLoading } = useQuery({
    queryKey: ['users', q],
    queryFn: async () => {
      const res = await fetch(`/api/users?q=${encodeURIComponent(q)}`);
      if (!res.ok) throw new Error('Failed');
      return res.json();
    },
    enabled: q.length > 0,
  });

  return (
    <div>
      <input value={q} onChange={(e) => setQ(e.target.value)} placeholder="Search users" />
      {isLoading ? 'Loading…' : <pre>{JSON.stringify(data, null, 2)}</pre>}
    </div>
  );
}
```

### Route handler for reuse and auth
```ts
// app/api/users/route.ts
import { NextResponse } from 'next/server';

export async function GET(req: Request) {
  const { searchParams } = new URL(req.url);
  const q = searchParams.get('q') ?? '';
  // auth, validation, and data access live here
  const data = await fetch(`https://api.example.com/users?q=${encodeURIComponent(q)}`).then(r => r.json());
  return NextResponse.json(data);
}
```

## Trade-offs
- Server fetching reduces JS but can increase server load if caching is off.
- Client fetching supports rich interactivity but increases bundle size and complexity.
- Route handlers provide reuse and control, but introduce extra hops if overused.

## Smells
- `useEffect` used to fetch initial page data that could be server-fetched.
- Duplicated fetching logic across pages/components.
- `no-store` used everywhere without reason; cache disabled by default.
- Data needed above a component is fetched far below it.

## When to break the rule
- Real-time feeds (websockets) and rapidly changing data.
- Strong personalization dependent on client-only state.
- Third-party SDKs that only run in the browser.
