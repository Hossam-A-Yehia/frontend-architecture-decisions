# 02. Feature-Based vs Layer-Based

## Problem

As teams and features grow, folder layout starts to influence velocity. The question isn’t which pattern is trendy, but which one reduces friction for your product’s stage and team shape.

## Options

### Option A — Layer-based

```
components/
hooks/
services/
utils/
```

**When it works**
- Prototypes, small apps, short-lived projects
- Single-team ownership, minimal domain complexity

**Pros**
- Simple mental model
- Easy to start and teach

**Cons**
- Related files scatter across folders
- PRs touch many layers for a single feature
- Ownership and boundaries blur as the app grows

### Option B — Feature-based

```
features/
  auth/
  orders/
  users/
```

**When it works**
- Product apps with multiple domains and teams
- Clear ownership per feature/area

**Pros**
- High cohesion: related files live together
- Easier ownership and autonomy
- Smaller, more focused PRs

**Cons**
- Requires shared conventions to avoid drift
- Risk of duplicate patterns across features

### Option C — Hybrid (feature-first + small shared)

```
features/
shared/
```

**When it works**
- Most real-world apps once features start multiplying

**Pros**
- Keeps feature code close
- Provides a clear home for truly shared utilities

**Cons**
- Needs discipline: `shared/` shouldn’t become a dumping ground

## Decision

Default to a **feature-based structure** with a small, intentional `shared/` area. Start simple; move to feature-based as soon as features and contributors grow. Keep the feature’s public API small via a single barrel (`features/<name>/index.ts`).

Exceptions: very small prototypes or libraries can remain layer-based until the structure starts to slow you down.

## Trade-offs

- Discoverability vs encapsulation
- Reuse vs coupling (promote only widely reused pieces to `shared/`)
- Consistency vs autonomy (codify conventions; don’t over-police)
- Refactor cost now vs chronic friction later

## Signals to switch (or that structure is wrong)

- A typical PR touches 3–5 unrelated folders
- Deep imports across features or circular deps
- A bloated `components/` at the root
- Developers ask “where do I put this?” too often
- Tests break when moving files across layers

## Migration (layer → feature)

1. Choose an anchor feature (e.g., `auth`); create `features/auth`.
2. Move related `components/`, `hooks/`, `services/` into `features/auth/...`.
3. Create `features/auth/index.ts` and export only what other features need.
4. Update imports to use the barrel; add temporary re-exports if necessary.
5. Repeat per feature; extract truly common pieces to `shared/` last.

## Examples

Layer-based:
```
components/
  Button.tsx
  UserCard.tsx
hooks/
  use-auth.ts
services/
  auth-service.ts
```

Feature-based:
```
features/
  auth/
    components/
      LoginForm.tsx
    hooks/
      use-auth.ts
    services/
      auth-service.ts
    index.ts
```

Import examples:
```ts
// layer-based
import { LoginForm } from '../components/LoginForm';
import { useAuth } from '../hooks/use-auth';

// feature-based (preferred)
import { LoginForm, useAuth } from '@/features/auth';
```

## Note on Next.js

Keep feature boundaries independent of routing. Route segments can live inside a feature or reference feature modules; avoid tying everything to `app/` structure. Default to Server Components where possible; place client-specific pieces inside the feature alongside their hooks.
