# 01. Folder Structure

## Problem

As a codebase grows, file placement becomes the first point of friction. A structure that felt "clean" at 10 components can become a maze at 200. The decision here is less about fashion and more about **how quickly a new developer can answer: _where does this live?_**

## Options

### Option A — Layer-based

```
components/
hooks/
services/
pages/
```

**Pros**
- Simple to start
- Easy to teach to new teams
- Works for small or short‑lived apps

**Cons**
- Related files get scattered
- Changes often touch multiple folders
- Refactors become slow as features grow

### Option B — Feature-based

```
auth/
orders/
users/
```

**Pros**
- Keeps related code together
- Easier ownership boundaries
- Scales better with teams

**Cons**
- Needs shared/common conventions
- Risk of repeated patterns across features

### Option C — Hybrid (feature + shared)

```
features/
shared/
```

**Pros**
- Clear boundaries for features
- Shared utilities stay visible

**Cons**
- Requires discipline to avoid dumping into `shared/`

## Decision

Default to a **feature-based structure** with a small, intentional `shared/` area. Start simple, but bias toward keeping related files together as soon as features appear.

If the app is a prototype or will stay tiny, a layer-based approach is fine. The moment features multiply or ownership grows, switch to feature-based to reduce cross‑folder churn.

## Trade-offs

- Feature-based structures can hide cross-cutting concerns if `shared/` is poorly maintained.
- Layer-based structures stay tidy until the first real scaling moment — then they get noisy fast.

## When This Breaks

- When `shared/` becomes a dumping ground
- When features start importing each other in a web of dependencies
- When a single feature spans too many folders to reason about quickly

## Recommended Conventions

- Folder names: `kebab-case`
- Component files: `PascalCase.tsx` (e.g., `UserCard.tsx`)
- Non-component files: `kebab-case.ts`
- Hooks: `use-*.ts`
- Types: `types.ts` or `*.types.ts`
- Tests: colocated `*.test.ts(x)`
- Stories: colocated `*.stories.tsx`
- One barrel per feature: `features/<name>/index.ts`

### Feature module shape

```
features/
  auth/
    components/
    hooks/
    services/
    lib/
    types.ts
    index.ts
```

Create subfolders when a feature passes ~7–10 files or has clear sub-flows (e.g., `orders/cart`, `orders/checkout`).

### Public API of a feature

- Export only what other features need from `index.ts`.
- Avoid deep imports like `features/auth/components/Form`; import from `features/auth`.
- If something becomes widely reused, promote it to `shared/` with a clear name.

Example barrel:

```ts
// features/auth/index.ts
export { LoginForm } from './components/LoginForm';
export { useAuth } from './hooks/use-auth';
export * as authService from './services/auth-service';
```

### Cross-feature imports

- Prefer: `features/<x>` imports `features/<y>` via `<y>/index.ts`
- Avoid circular deps; extract shared pieces to `shared/` when needed.
- Consider lint guards (`import/no-cycle`, `no-restricted-imports`) to prevent deep imports.

### Tests and stories placement

- Co-locate with the unit under test.
- Feature integration tests can live at `features/<name>/<name>.test.ts`.

### Barrel files: when and how

- Keep barrels at feature roots; avoid barrels for every subfolder.
- Export explicitly; avoid `export *` from deep trees to keep the API surface intentional.

### Path aliases (optional)

`tsconfig.json`

```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": {
      "@features/*": ["features/*"],
      "@shared/*": ["shared/*"]
    }
  }
}
```

## Migration: layer-based → feature-based

1. Pick one feature (e.g., auth). Create `features/auth`.
2. Move related `components/`, `hooks/`, `services/` into `features/auth/`.
3. Add `features/auth/index.ts` exposing the public API.
4. Update imports to use the barrel; keep temporary re-exports if needed.
5. Repeat per feature; extract truly common pieces to `shared/` last.

## Smells to watch for

- `shared/` becomes a dumping ground
- Deep imports across features
- “God” feature folder with unrelated concerns
- Barrels that re-export entire trees
- Utilities that become a dumping ground

## Key Takeaway

> Folder structure is about how fast a developer understands the codebase — not how neat it looks in a tree.
