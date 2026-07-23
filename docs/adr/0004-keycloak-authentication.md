# ADR-0004: Keycloak for Authentication and Identity

**Status:** Accepted

## Context

`domain-model.md`'s `Account` entity is deliberately separate from `Person` precisely so that authentication concerns (credentials, sessions, MFA) never mix with domain data. That separation only pays off if there's a dedicated identity provider doing the authentication job, rather than the NestJS backend rolling its own password storage and session management. The product's EU-hosting stance (`product-principles.md` principle 7, `security.md`) also rules out defaulting to a US-hosted identity SaaS for something as sensitive as participant credentials.

## Decision

**Keycloak**, self-hosted on Hetzner infrastructure (ADR-0005), as the sole identity provider, using OIDC. The NestJS backend validates Keycloak-issued tokens at the API boundary (`api-guidelines.md`) and never stores or handles raw credentials itself.

## Consequences

**Positive:**
- Mature, open-source, self-hostable — fits the EU-data-residency requirement without a recurring per-user SaaS cost or a foreign-hosted dependency for the single most sensitive data category (login credentials).
- Standard OIDC integration means the Next.js frontend and NestJS backend both use well-supported, unexceptional libraries rather than a bespoke auth flow — reduces the custom security-critical code this team has to get right.
- Supports the organizational-membership model (`domain-model.md`'s `Membership`) via Keycloak's own realm/group/role primitives if useful for coarse-grained access control, while the fine-grained `Membership.role` authorization logic still lives in the application layer (`security.md`) — Keycloak is not asked to encode the full domain authorization model, only to authenticate identity.
- Decouples credential lifecycle (password reset, MFA enrollment, SSO) from application deployments — the backend can ship independently of identity-provider changes.

**Negative / accepted costs:**
- Another stateful service to operate (Keycloak needs its own database, backups, upgrade cadence) — accepted as a bounded, well-understood operational cost, and one Ansible/Docker (`deployment.md`) are already set up to handle as a pattern.
- Steeper initial configuration than a "just use a hosted auth SaaS" approach — accepted for the data-residency and cost-at-scale reasoning above.

## Alternatives Considered

- **A hosted third-party auth SaaS (e.g. Auth0, Clerk):** faster initial setup, but most mainstream options are US-hosted or have unclear EU-data-residency guarantees for the specific plan tiers this project could afford — conflicts with `product-principles.md` principle 7. Would need its own ADR with an explicit data-residency justification to reconsider.
- **Rolling custom authentication in the NestJS backend:** rejected outright — implementing credential storage, session/token issuance, and MFA correctly and securely is a substantial, high-stakes undertaking that a dedicated, audited identity provider already solves. This is exactly the kind of custom security-critical code `security.md` argues against building in-house without strong justification.
