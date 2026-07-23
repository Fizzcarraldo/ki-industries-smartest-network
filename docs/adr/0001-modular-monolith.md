# ADR-0001: Modular Monolith over Microservices

**Status:** Accepted

## Context

The target platform (`domain-model.md`) has six identifiable domain modules from the outset: identity, organizations, skills, verification, relationships, trust. It would be technically possible to build each as an independently deployed microservice from day one. The team building this is small, the domain is still being learned in production (the exact shape of `TrustScore` computation, for instance, is explicitly not yet fully specified — see `domain-model.md`), and there is no existing operational microservices infrastructure (`technology-stack.md`'s target stack has no service mesh, no distributed tracing set up yet beyond planned OpenTelemetry, no per-service on-call rotation).

## Decision

Build **one deployable NestJS backend**, internally divided into modules that mirror the domain model, each internally layered per Clean Architecture (`architecture.md`). Cross-module communication is in-process (method calls / an in-process event emitter), not network calls.

Enforce module boundaries at the *code* level (no cross-module reach-through into another module's persistence layer — `coding-standards.md`) so that extracting a module into its own service later is a mechanical, low-risk operation if and when a module's independent scaling, deployment cadence, or team ownership actually earns that complexity — not before.

## Consequences

**Positive:**
- One deployment pipeline, one set of infra to operate, one transaction boundary for anything that needs cross-module consistency (e.g. a `Relationship` state change and a `TrustScore` invalidation happening atomically).
- Lower operational overhead while the team and domain model are both still small and still changing.
- The migration path to extracting a service is explicitly designed for (via the enforced module boundaries), not foreclosed.

**Negative / accepted costs:**
- All modules currently scale together — if one module (e.g. `verification`, which might involve slow external register checks) needs different scaling characteristics than the rest, that's not free until/unless it's extracted.
- Requires discipline: the boundary enforcement (`coding-standards.md`'s "no cross-module reach-through," lint rules) has to actually be enforced, or the monolith degrades into an undifferentiated ball of mud with none of the stated benefits.
- A bug in one module can, in principle, affect the availability of the whole deployable unit — mitigated by standard practices (health checks, careful error boundaries) rather than by process isolation.

## Alternatives Considered

- **Microservices from day one:** rejected — the operational cost (service discovery, distributed transactions or sagas for what are currently single-consistency-boundary operations, per-service deployment/monitoring) is not justified by a team and domain model of this size and maturity. Revisit if/when a specific module has a concrete, stated reason (independent scaling need, independent team ownership) to be extracted — at that point, write a superseding ADR.
- **Single undifferentiated monolith (no internal module boundaries):** rejected — this trades away the extraction path and, per `product-principles.md`'s emphasis on structured modeling, would make the domain boundaries in `domain-model.md` purely conceptual rather than enforced in code.
