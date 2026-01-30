# Mistakes to Avoid

## Over‑abstraction too early
- Problem: building for hypothetical reuse creates indirection and slows delivery.
- Avoid: wait until a second real use before extracting.
- Do: keep code simple; extract when duplication is obvious and stable.

## Premature optimization
- Problem: micro‑optimizing before measuring leads to complexity with no impact.
- Avoid: speculative memoization and custom caches.
- Do: set budgets, measure, then optimize the few hot paths.

## Global state everywhere
- Problem: cross‑feature coupling, hard to reason about changes, incidental complexity.
- Avoid: dumping view concerns into global stores.
- Do: prefer local/derived and URL state; restrict global to truly cross‑cutting concerns.

## Dumping ground `shared/`
- Problem: unclear ownership, hidden coupling via “helpers”.
- Avoid: placing feature‑specific utilities in `shared/`.
- Do: promote only widely reused, stable utilities with good names and docs.

## Deep cross‑feature imports
- Problem: tight coupling and brittle refactors.
- Avoid: importing internal files of other features.
- Do: consume only the public API (feature barrel) or extract to shared.

## God components
- Problem: long props lists, conditional branches, and poor testability.
- Avoid: bolting on flags and modes.
- Do: split responsibilities; compose small components with clear roles.

## Effects for everything
- Problem: race conditions, double work, inconsistent UI.
- Avoid: mirroring state and fetching on mount by default.
- Do: derive where possible; keep effects for imperative needs.

## Copy‑pasted data access
- Problem: duplicated error handling, caching policies, and mapping logic.
- Avoid: ad‑hoc `fetch` calls scattered across UI.
- Do: centralize access behind small adapters/services per domain.

## Missing boundaries in tests
- Problem: brittle tests tied to implementation details.
- Avoid: deep mocks of internals.
- Do: test via public feature APIs and observable behavior.

## No guardrails
- Problem: regressions sneak in, cycles go unnoticed.
- Avoid: relying only on code reviews.
- Do: add lint rules, type checks, unit tests, and perf budgets in CI.
