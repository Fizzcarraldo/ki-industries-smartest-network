# ADR-0006: TypeScript Across the Full Stack

**Status:** Accepted

## Context

The target platform spans a Next.js frontend and a NestJS backend (`technology-stack.md`), communicating through an API whose schema is meant to be the single contract both sides honor (`api-guidelines.md`'s API-First principle). `domain-model.md`'s entities need to be representable identically in the type systems both the frontend and backend reason about, or the "generated typed client" approach in `api-guidelines.md` doesn't actually deliver its main benefit: a change to the domain model breaking the *frontend build*, not a runtime bug discovered by a user.

## Decision

**TypeScript, strict mode, on both frontend and backend** — no polyglot split (e.g. a Python or Go backend paired with a TypeScript frontend).

## Consequences

**Positive:**
- Shared types for API request/response DTOs and domain enums (`Membership.role`, `Verification.method`, etc. — `domain-model.md`) can be generated once and consumed by both sides, eliminating an entire class of "frontend and backend silently disagree about a field's shape" bugs.
- One hiring/skill profile for the team rather than two — meaningful for a small team where full-stack contributors need to move between frontend and backend work.
- NestJS's module/DI system, which `architecture.md`'s Clean Architecture layering leans on directly, is itself TypeScript-native — the architectural pattern and the language choice reinforce each other rather than being independent decisions.
- `coding-standards.md`'s strict-mode, no-`any` rules apply uniformly across the whole codebase with one toolchain (one ESLint/Prettier config, `development-workflow.md`), rather than needing parallel standards documents per language.

**Negative / accepted costs:**
- TypeScript's type system, even in strict mode, doesn't provide the same runtime safety guarantees as a statically-compiled language with a stronger type system (e.g. Rust) — accepted, mitigated by strict linting, runtime validation at API boundaries (`api-guidelines.md`), and the test strategy in `testing.md` rather than relying on the type system alone.
- Node.js runtime performance characteristics are adequate but not best-in-class for CPU-bound workloads — not currently a concern for this product's workload profile (I/O-bound: database queries, external verification checks), revisit only if a specific, measured bottleneck emerges (and if so, that's an isolated-service extraction decision under ADR-0001's stated migration path, not a full-stack language change).

## Alternatives Considered

- **Polyglot (e.g. Go or Python backend, TypeScript frontend):** would likely give backend performance or ecosystem advantages for specific tasks (e.g. Python for future ML-driven matching/recommendation work referenced in `vision.md`'s "Intelligenz" stage), at the cost of duplicated type definitions across a language boundary and a fragmented toolchain. Rejected for the core platform now; **explicitly left open** as a future, narrowly-scoped exception — if the "Intelligenz" stage's recommendation engine needs a Python-ecosystem tool, that's a candidate for a small, isolated service extracted per ADR-0001's migration path, not a reason to abandon this decision for the core platform.
- **A single full-stack framework spanning both concerns (e.g. a Next.js-only app with API routes, no separate NestJS backend):** rejected because it doesn't give the Clean Architecture module/layer separation `architecture.md` requires — Next.js API routes don't provide the same structured DI/module system NestJS does, and mixing frontend and backend concerns in one framework works against the "no business logic in frontend" rule in `coding-standards.md` by making the boundary a convention rather than a structural one.
