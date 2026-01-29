# 05. Component Boundaries

## Problem

Components tend to grow until they do "everything": data fetching, business rules, formatting, and UI. That slows changes and makes reuse brittle.

## Principles

- One component, one reason to change.
- Prefer composition over props bloat.
- Keep business logic in hooks/services, not in the view.
- Push state down (local) unless there’s a clear reason to lift.

## Decision

- Keep UI components **pure** and focused on rendering.
- Move side effects and data access to hooks/services.
- Expose **event-first APIs** (callbacks) rather than controlling everything via props.
- Compose small components into screens instead of building “god components.”

## Conventions

- UI components: `PascalCase.tsx`, minimal props, no I/O.
- Hooks: `use-*.ts`, own effectful logic and orchestration.
- Screen modules assemble hooks + UI; they may fetch data (prefer server) and pass results down.
- Use children/composition for extension points; avoid boolean-prop matrices.

## Examples

### Composition over props
```tsx
// Good: Card composed from small parts
<Card>
  <Card.Header title="Users" />
  <Card.Body>
    <UsersTable users={users} />
  </Card.Body>
  <Card.Footer>
    <Button onClick={onAdd}>Add</Button>
  </Card.Footer>
</Card>
```

```tsx
// Risky: monolithic props surface
<UsersCard
  showHeader
  showFooter
  footerPrimaryCtaLabel="Add"
  onFooterPrimaryCta={onAdd}
  tableVariant="compact"
  withSearch
  withExport
/>
```

### Move logic to hooks
```tsx
// use-users.ts
import { useQuery } from '@tanstack/react-query';
export function useUsers(q: string) {
  return useQuery({ queryKey: ['users', q], queryFn: () => fetch(`/api/users?q=${q}`).then(r => r.json()) });
}
```

```tsx
// UsersTable.tsx
export function UsersTable({ users }: { users: any[] }) {
  return (
    <table>
      <tbody>
        {users.map(u => (
          <tr key={u.id}>
            <td>{u.name}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

## Trade-offs

- More files, more seams — but clearer responsibilities.
- Composition can hide complexity; naming and structure matter.
- Hooks centralize logic but can become mini-services; keep APIs tight.

## Smells

- Components with long prop lists and conditional rendering branches.
- Repeated data access scattered across components.
- UI that is hard to test because it owns effects and data fetching.

## When this breaks

- Extremely simple screens where composition adds overhead.
- Highly dynamic UIs where abstraction hides too much context.
