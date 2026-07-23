# Architecture Decision Records

An ADR captures a significant architectural decision: the context that forced it, what was decided, what it costs, and what else was considered. It is a historical record — once accepted, an ADR is not edited to reflect new information; a changed decision gets a *new* ADR that supersedes the old one (and says so explicitly).

Per [`constitution.md`](../constitution.md) Rule V, an ADR is written **before** the decision is implemented, not as after-the-fact documentation.

## When to write one

- Choosing or replacing a core technology (database, ORM, auth provider, hosting).
- Establishing or changing a module/service boundary.
- Any decision `CLAUDE.md` flags as requiring one when a conflict comes up during implementation.

Not every decision needs an ADR — a naming choice or a local refactor doesn't. If you're unsure whether something is ADR-worthy, ask: *"would a new engineer joining in a year need to understand why we didn't do the obvious alternative?"* If yes, write it down.

## Format

Each ADR states: **Status** (Proposed / Accepted / Superseded), **Context**, **Decision**, **Consequences**, **Alternatives Considered**.

## Index

| ADR | Decision |
|---|---|
| [0001](./0001-modular-monolith.md) | Modular monolith over microservices |
| [0002](./0002-postgresql-source-of-truth.md) | PostgreSQL as the single source of truth |
| [0003](./0003-drizzle-orm.md) | Drizzle ORM |
| [0004](./0004-keycloak-authentication.md) | Keycloak for authentication and identity |
| [0005](./0005-hetzner-cloud-infrastructure.md) | Hetzner Cloud infrastructure |
| [0006](./0006-typescript-fullstack.md) | TypeScript across the full stack |
