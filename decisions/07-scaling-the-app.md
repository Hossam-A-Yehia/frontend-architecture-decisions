# 07. Scaling the Application

## Problem

Small frontends tolerate messy boundaries. As domains, teams, and traffic grow, the same choices slow delivery, raise defect rates, and make changes risky. Scaling is about deliberately tightening boundaries and feedback loops while keeping the product fast.

## Refactor signals (it’s time to evolve)

- A typical feature PR touches many unrelated folders.
- Cross‑feature imports or implicit coupling (deep imports, circular deps).
- Global state holds unrelated concerns; adding state feels risky.
- Performance regressions appear after “harmless” UI changes.
- Flaky tests and fragile mocks; hard to reproduce issues.
- Onboarding takes weeks to make a safe change.

## What to change first

1. Boundaries and ownership
   - Organize by feature/module with a small, curated `shared/`.
   - Define a public API per feature (barrel) and forbid deep imports.
2. Data access and contracts
   - Centralize data access behind small adapters/services per domain.
   - Keep types/schemas close to adapters; map API → domain early.
3. State strategy
   - Prefer local/derived state; keep cross‑feature state rare and explicit.
   - Model URL state for shareable view concerns (filters, pagination).
4. Guardrails
   - Add lint rules/import guards for dependency direction and cycles.
   - Add basic CI checks: typecheck, lint, unit tests, size budgets.

## Architecture moves that help at scale

- Stable interfaces
  - Feature barrels export a small, intentional surface.
  - Adapters hide API details; screens consume interfaces, not endpoints.
- Dependency direction
  - UI → hooks/services → adapters → transport (never the reverse).
  - Avoid feature ↔ feature cycles; extract shared pieces when necessary.
- Composition over inheritance
  - Compose small components into screens; avoid monoliths with option flags.
- Async boundaries
  - Timeouts, retries, and cancellation at adapter level.
  - Idempotent actions and optimistic flows have clear rollback paths.

## Data and state at scale (vendor‑neutral)

- Local/derived state first; avoid mirroring props in state.
- URL state for routing, filters, pagination, tabs, and sorting.
- Server‑facing state (remote/cache) behind adapters; treat as a separate concern.
- Global state only for truly cross‑cutting concerns (auth/session, feature flags), with a narrow API.

## Performance and observability checklist (vendor‑neutral)

- Budgets and SLIs
  - Establish budgets for bundle size, route‑level JS, image weight.
  - Track Core Web Vitals (LCP, INP, CLS) and p95 latency for critical actions.
- Instrumentation
  - Add lightweight timing markers for user‑visible steps (e.g., “search:results:rendered”).
  - Emit structured logs (JSON) with a request/session correlation id.
- Errors
  - Capture uncaught errors and promise rejections; group by fingerprint.
  - Record user action context (screen/feature, params) with privacy in mind.
- Tracing
  - Propagate trace/context ids from entry to client actions.
  - Measure end‑to‑end latency for key flows (navigation → data → paint).
- Caching and data freshness
  - Define cache lifetimes and invalidation triggers per domain.
  - Prefer background refresh over blocking spinners where safe.
- UI performance
  - Code‑split along feature boundaries; lazy‑load non‑critical UI.
  - Avoid re‑render cascades (memoize, stable props, virtualization for large lists).
  - Defer non‑blocking work to idle periods.
- Testing and protection
  - Add perf smoke checks in CI for hot paths (budget violations fail builds).
  - Synthetic checks for critical journeys (login, search, checkout).

## Examples

### 1) Feature module with stable API and adapter injection
```ts
// features/users/index.ts (public API)
export { UsersTable } from './components/users-table';
export type { User, UsersQuery } from './types';
export { makeUsersService, type UsersService } from './services/users-service';
```

```ts
// features/users/services/users-service.ts
export type UsersQuery = { q?: string; page?: number };
export type User = { id: string; name: string };

export type UsersPort = {
  list(query: UsersQuery): Promise<User[]>;
};

export type UsersService = ReturnType<typeof makeUsersService>;

export function makeUsersService(port: UsersPort) {
  return {
    async list(query: UsersQuery) {
      // business‑level concerns (validation, mapping, caching policy)
      const raw = await port.list(query);
      return raw.map((u) => ({ id: String(u.id), name: u.name?.trim() ?? '' }));
    },
  };
}
```

```ts
// adapters/http/users-port.ts (transport hidden behind interface)
import type { UsersPort, UsersQuery, User } from '../../features/users/services/users-service';

export function makeHttpUsersPort(baseUrl: string): UsersPort {
  return {
    async list(query: UsersQuery): Promise<User[]> {
      const url = new URL(baseUrl + '/users');
      if (query.q) url.searchParams.set('q', query.q);
      if (query.page) url.searchParams.set('page', String(query.page));
      const res = await fetch(url.toString());
      if (!res.ok) throw new Error('Users fetch failed');
      return res.json();
    },
  };
}
```

```ts
// screen composition (UI depends on service interface, not transport)
import { makeUsersService } from '@/features/users';
import { makeHttpUsersPort } from '@/adapters/http/users-port';

const usersService = makeUsersService(makeHttpUsersPort('/api'));

async function loadUsers() {
  return usersService.list({ q: 'alex' });
}
```

### 2) Dependency direction guard (concept)
- Allow: UI → feature API → services/adapters.
- Forbid: feature A importing deep files from feature B.
- Periodically scan imports and visualize cycles; break by extracting shared parts.

### 3) State placement example
```ts
// Local/derived
const fullName = `${first} ${last}`; // no state needed

// URL state for shareable view
const params = new URLSearchParams(location.search);
const page = Number(params.get('page') ?? 1);

// Global state only when truly cross‑cutting (e.g., session)
```

## Red flags and mitigations

- Red flag: multiple features write into one “god” global store.
  - Mitigation: split stores by domain; expose narrow selectors and commands.
- Red flag: a single change requires edits across many features.
  - Mitigation: extract shared utilities or invert dependency direction.
- Red flag: size/perf regressions after each sprint.
  - Mitigation: add budgets, CI checks, and lazy‑load non‑critical paths.
- Red flag: brittle tests coupled to implementation details.
  - Mitigation: test via public feature APIs; prefer interaction over internals.

## How to measure success

- Lead time from idea → prod is trending down.
- Fewer production incidents tied to “small refactors.”
- Faster cold/warm interactions (p95 latency down, vitals stable or better).
- Onboarding: first useful PR lands in days, not weeks.
