# 03. State Management Decisions (Vendor‑neutral)

## Problem

Not all state is equal. Treating every piece of data as “global” creates coupling, re-renders, and complexity. The goal is to place state where it’s easiest to reason about and least costly to maintain.

## Types of state

- Local UI state — ephemeral, component/screen scope (inputs, toggles, open/close).
- Derived state — computed from existing state/props; doesn’t need its own store.
- URL state — shareable view state (filters, sorting, pagination, tabs).
- Server (remote) state — fetched from a backend; has freshness, caching, and error states.
- Global app state — truly cross-cutting (session, feature flags, theme), narrow surface.

## Decision tree (quick rules)

1) Can it be derived from existing data? If yes → derive, don’t store.
2) Is it purely view-local and transient? If yes → local component/screen state.
3) Must it survive reload/share via link? If yes → URL state.
4) Does the backend own the source of truth? If yes → treat as server/remote state.
5) Is it needed by many unrelated areas all the time? If yes → global, but keep API narrow.

## Conventions

- Prefer local/derived first; lift only when a child cannot own it and multiple siblings depend on it.
- Keep remote/server state behind adapters or a data layer; cache and invalidate intentionally.
- Use URL for shareable view concerns; keep serialization stable and backward compatible.
- Keep global state minimal; expose selectors/commands instead of raw objects.

## Examples

### Local UI state
```tsx
function ModalExample() {
  const [open, setOpen] = useState(false);
  return (
    <>
      <Button onClick={() => setOpen(true)}>Open</Button>
      {open && <Modal onClose={() => setOpen(false)} />}
    </>
  );
}
```

### Derived state
```tsx
const fullName = `${user.first} ${user.last}`; // no state; compute when needed
```

### URL state
```ts
// read
const params = new URLSearchParams(location.search);
const page = Number(params.get('page') ?? 1);
const q = params.get('q') ?? '';

// write
params.set('page', String(page + 1));
history.pushState(null, '', `?${params.toString()}`);
```

### Server (remote) state
```ts
// adapter hides transport details, centralizes caching and error policy
export async function listUsers(query: { q?: string }) {
  const url = new URL('/api/users', location.origin);
  if (query.q) url.searchParams.set('q', query.q);
  const res = await fetch(url.toString(), { headers: { Accept: 'application/json' } });
  if (!res.ok) throw new Error('Failed');
  return res.json();
}
```

### Global state (narrow and stable)
```ts
// session-store.ts
export type Session = { userId: string | null };
let session: Session = { userId: null };

export function getSession() { return session; }
export function login(userId: string) { session = { userId }; }
export function logout() { session = { userId: null }; }
```

## Anti-patterns

- Mirroring props into state without need.
- Storing server data globally by default.
- Using global state to coordinate sibling components that could lift state to a common parent.
- Encoding UI-only concerns (like open/closed) in global stores.
- Entangled state where a write in one feature triggers surprising changes elsewhere.

## When this breaks

- Real-time heavy UIs that require client-local caches and subscriptions.
- Offline-first scenarios that need durable storage and sync.

## Checklist

- Can this be derived instead of stored?
- Who owns the source of truth (UI, URL, server, global)?
- Will a URL capture help share/reproduce the view?
- What’s the invalidation policy for remote data?
- Is the global API surface as small as possible?
