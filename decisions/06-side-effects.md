# 06. Side Effects & Effects Management

## Problem

`useEffect` is often used as a catch‑all for anything that needs to "happen". Many of those cases are not effects; they’re data derivations or event responses. Overusing effects leads to race conditions, double work, and inconsistent state.

## Decision

- Treat effects as a last resort for imperative work: subscriptions, timers, DOM APIs, non-React I/O.
- Derive state where possible (props → UI) instead of mirroring it in state.
- Prefer event-driven logic (handlers) to effect-driven logic.
- Avoid putting fetches in effects for initial render when server components can fetch.

## Patterns

### Derive instead of mirror
```tsx
// Bad: mirroring props in state
const [fullName, setFullName] = useState('');
useEffect(() => setFullName(`${first} ${last}`), [first, last]);

// Good: derive inline
const fullName = `${first} ${last}`;
```

### Effects for subscriptions with cleanup
```tsx
useEffect(() => {
  const sub = bus.subscribe('user:updated', onUser);
  return () => sub.unsubscribe();
}, [bus, onUser]);
```

### Abortable fetch in effects (when client fetch is justified)
```tsx
useEffect(() => {
  const ctrl = new AbortController();
  fetch('/api/data', { signal: ctrl.signal }).then(/* ... */);
  return () => ctrl.abort();
}, []);
```

## Smells

- Effects that set state derived from props or other state.
- Effects that depend on unstable functions/objects without memoization.
- Fetching on mount for data that could be server-fetched.
- Missing cleanup; listeners leak across navigations.

## Tooling & guardrails

- ESLint rules: `react-hooks/exhaustive-deps` and `no-restricted-imports` patterns to avoid deep cross-feature imports that cause cycles.
- Encapsulate effectful logic in hooks (`use-*.ts`), keep UI components pure.

## When this breaks

- Real-time heavy apps that rely on client subscriptions — embrace effects with care and test for race conditions.
- Third-party SDKs that require browser-only lifecycle.

## Key idea

> Most effects are not actually effects.
