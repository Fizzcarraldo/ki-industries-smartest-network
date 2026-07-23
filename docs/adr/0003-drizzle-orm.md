# ADR-0003: Drizzle ORM

**Status:** Accepted

## Context

`architecture.md`'s Clean Architecture layering requires that `domain/` code stay free of framework/ORM imports, with persistence pushed to `infrastructure/` behind repository interfaces (`coding-standards.md`). Given that constraint, the ORM's job is narrow and well-defined: type-safe schema definition and querying in the infrastructure layer, nothing more — it never needs to "own" the domain model, because the domain model is defined independently in TypeScript per `domain-model.md` and mapped, not generated top-down from decorated entity classes.

## Decision

**Drizzle ORM** for schema definition and querying against PostgreSQL (ADR-0002).

## Consequences

**Positive:**
- Drizzle's schema-as-TypeScript-code approach produces fully-typed query results with no code generation step and no runtime reflection — consistent with `coding-standards.md`'s strict-TypeScript, no-`any` mandate, and simpler to reason about than decorator-heavy ORMs (e.g. TypeORM) whose entity classes tend to accumulate framework concerns that leak toward the domain layer.
- SQL-like query builder keeps generated queries predictable and inspectable — important for the integration-testing strategy in `testing.md`, which tests the infrastructure layer against a real Postgres instance specifically because ORM-generated SQL can otherwise be a silent source of bugs.
- Drizzle's migration tooling (`drizzle-kit`) gives an explicit, reviewable migration file per schema change — fits `development-workflow.md`'s requirement that a `domain-model.md` change and its migration land together, deliberately, not as an implicit side effect of application boot (`deployment.md`).
- Lightweight — no heavyweight ORM runtime, which matters for a modular monolith (ADR-0001) where every module's persistence code shares one process.

**Negative / accepted costs:**
- Smaller ecosystem and fewer "batteries included" features (no built-in soft-delete, no automatic timestamp management) than a more opinionated ORM — acceptable, because `domain-model.md`'s cross-cutting rules (soft-delete, `createdBy`/`updatedBy`, timestamps on every entity) are explicit, deliberate, and enforced consistently by convention/lint rather than framework magic anyway.
- Team needs to write slightly more explicit query code in places a heavier ORM would auto-generate — treated as a feature, not a cost, given `coding-standards.md`'s general preference for explicit over implicit.

## Alternatives Considered

- **Prisma:** strong type safety and DX, but its own DSL for schema definition (rather than TypeScript) and generated client add a build step and a layer of indirection that sits awkwardly against the Clean Architecture goal of keeping the domain layer's shape as the one source of truth for entity structure, with infrastructure mapping onto it — not generating it. Reconsider only if Drizzle proves insufficient for a concrete, encountered need.
- **TypeORM:** decorator-based entity classes actively encourage exactly the framework-leaking-into-domain pattern `coding-standards.md` prohibits. Rejected on architecture-fit grounds, not raw capability.
- **Raw SQL / a query builder without an ORM layer (e.g. Kysely alone):** viable alternative with similar architecture fit to Drizzle; Drizzle was chosen for its combination of migration tooling and query builder in one, reducing the number of tools to operate. Not a strong rejection — could be revisited if Drizzle's migration tooling proves inadequate in practice.
