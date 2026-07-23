# Security

> Status: **Mixed.** The current static site has a narrow, low-stakes security surface (documented below, as it actually is). The target platform handles real business and personal data and needs the controls described in the target section before it handles a single real `Account`.

## Current State

The current site (`index.html`, `vision.html`) is a static marketing/demo page with:

- No authentication, no user data collection beyond third-party analytics (Umami, self-hosted — see `technology-stack.md`).
- No secrets in the repository (verified — no `.env`, no API keys committed; the Umami site ID in the tracking script is a public identifier, not a secret).
- Its only write surface is `mailto:` links — no forms submit data anywhere.
- Deployment is via `git push` + GitHub Pages, gated by whoever has push access to the repository (see `deployment.md`). There is currently no branch protection or required review configured — this is acceptable risk for a demo site with one maintainer, but must not carry over to the product repository once real user data is involved (see `development-workflow.md`).

This is a genuinely low-risk surface today. The rules below are about what changes the moment `Account`/`Person`/`Organization` data becomes real.

## Target: Authentication & Identity

- **Keycloak is the single source of truth for authentication** (OIDC). Neither the NestJS backend nor the Next.js frontend implements its own password storage, session management, or credential verification. See [ADR-0004](./adr/0004-keycloak-authentication.md).
- The `Account` entity (`domain-model.md`) mirrors the Keycloak subject; it never duplicates or caches a password hash, MFA secret, or other Keycloak-owned credential.
- Tokens are validated on every request at the API boundary (`api-guidelines.md`). Frontend route guards are a UX convenience, not a security control — the backend is the actual enforcement point, always.

## Target: Authorization

- Authorization decisions use `Membership.role` (`domain-model.md`) evaluated in the application layer, never inferred from the frontend's current view state.
- Every action that mutates `Organization`-owned data (accepting a `Relationship`, triggering a `Verification`) checks that the acting `Account`, through an active (`status: active`, `endedAt: null`) `Membership`, has sufficient `role` for that organization — checked server-side, on every request, not cached client-side as a standing permission.

## Target: Data protection & GDPR

The product's stated positioning is European infrastructure (`vision.md`, `product-principles.md` principle 7); the following are not optional nice-to-haves given that positioning:

- **Data residency:** production data stays on EU-based infrastructure (Hetzner — [ADR-0005](./adr/0005-hetzner-cloud-infrastructure.md)). Don't introduce a US-hosted SaaS dependency that stores participant PII (`Person`, `Organization`, `Verification.evidenceRef`) without an explicit ADR addressing the data-residency implication.
- **Right to erasure vs. audit trail tension:** `domain-model.md` mandates soft-delete for `Person`/`Organization`/`Relationship`/`Verification` to preserve trust provenance. This is in tension with GDPR erasure requests and must be resolved deliberately, not by accident: when a real erasure request needs to be handled, the design is to anonymize (strip PII fields) while preserving the *structural* trust record (a `Relationship` existed, a `Verification` of some kind occurred) — not to hard-delete rows that other entities' `TrustScore` computations depend on. This needs its own ADR before the first real erasure request arrives, not after.
- **`Verification.evidenceRef`** (business register extracts, credential documents) is the most sensitive data in the model — stored in Hetzner Object Storage with access scoped to the verification process itself, never served through a public URL, never logged in full (see `coding-standards.md`'s logging rules).
- **Data minimization:** don't collect a field "because it might be useful" — this is `product-principles.md` principle 6 applied to privacy specifically. Every field in `domain-model.md` should be traceable to a concrete product need.

## Target: Secrets management

- No secrets in source control, ever — enforced by a pre-commit/CI secret-scan (e.g. `gitleaks`) once the product repository exists, not just by discipline.
- Secrets (database credentials, Keycloak client secrets, Object Storage keys) are injected via environment variables at deploy time, managed through Ansible vault or an equivalent secret store — never baked into a Docker image layer.
- Rotate on suspected exposure, not on a fixed calendar alone — but do have a fixed rotation cadence for anything that doesn't have a specific reason not to.

## Target: Audit trail

- Every entity mutation carries `createdBy`/`updatedBy` (`domain-model.md`'s cross-cutting rule) — this is a security control as much as a data-modeling one: it's what makes a disputed `Verification` or `Relationship` change investigable after the fact.
- Authentication events (login, token refresh failures, permission denials) are logged with enough context to reconstruct an incident, without logging the token/credential itself (`coding-standards.md`).

## Reporting a vulnerability

Not yet formalized — add a `SECURITY.md` with a real contact/process before the platform has real users. Track this as a concrete pre-launch item, not a someday task (see the Engineering Report's short-term recommendations).
