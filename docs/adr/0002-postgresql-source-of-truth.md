# ADR-0002: PostgreSQL as the Single Source of Truth

**Status:** Accepted

## Context

The domain model (`domain-model.md`) is deliberately relational: entities reference each other by foreign key (`Membership` joins `Person` and `Organization`; `Verification` and `SkillAssignment` are polymorphic but still structurally typed; `Relationship` connects two typed parties). `product-principles.md` principle 3 mandates structured data over free text and explicitly rules out generic document/JSON dumps as a substitute for modeling. The system also needs strong consistency guarantees in specific places — a `TrustScore` recomputation should never observe a `Relationship` in a half-committed state.

## Decision

**PostgreSQL is the single source of truth** for all domain data described in `domain-model.md`. No secondary datastore (a document store, a separate search index, a cache used as a read model) holds data that isn't ultimately derived from and reconcilable against Postgres. Any such secondary store, if introduced later for a specific performance need, is explicitly a derived/rebuildable cache, never authoritative.

## Consequences

**Positive:**
- Relational integrity (foreign keys, constraints) enforces the domain model's shape at the database level, not just in application code — a second line of defense against `product-principles.md` principle 3 being silently violated.
- Transactions give the consistency guarantees needed for trust-affecting operations (`security.md`'s audit-trail requirements, `domain-model.md`'s cross-entity invariants).
- Mature, well-understood operational characteristics; strong ecosystem support for the rest of the target stack (Drizzle — see ADR-0003).
- Fits the EU-hosting requirement (`product-principles.md` principle 7) — self-hostable on Hetzner (ADR-0005) without a proprietary managed-database lock-in.

**Negative / accepted costs:**
- Horizontal write scaling is harder than with some NoSQL alternatives — accepted, because the product's current and foreseeable scale doesn't need it, and the modular monolith (ADR-0001) keeps the write surface manageable.
- Schema changes require migrations (via Drizzle — ADR-0003), which is deliberate friction, not a downside: it's the mechanism that keeps `domain-model.md` and the actual schema from drifting apart.

## Alternatives Considered

- **A document database (e.g. MongoDB) as primary store:** rejected outright — directly conflicts with `product-principles.md` principle 3's rejection of unmodeled JSON dumps, and the domain model's relational structure (polymorphic references aside) doesn't benefit from schemaless storage.
- **Managed database-as-a-service (e.g. a US-hosted managed Postgres):** rejected in favor of self-hosted-on-Hetzner Postgres specifically for the EU-data-residency reasoning in ADR-0005 and `security.md` — a managed EU-region offering could be reconsidered as an operational convenience later, via its own ADR, without changing this decision's core (Postgres as the engine).
