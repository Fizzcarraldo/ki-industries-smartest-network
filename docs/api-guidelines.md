# API Guidelines

> Status: **Target — no API exists yet.** This document defines the conventions the NestJS backend's HTTP API must follow from its first endpoint, per `architecture.md`'s API-First principle.

## API-First means the schema is the contract

The OpenAPI schema is **generated from the NestJS backend** (via `@nestjs/swagger` decorators on DTOs, or an equivalent schema-first tool if that proves insufficient — decide and record as an ADR when the first module is built). It is never hand-written separately from the code and never allowed to drift from it. The Next.js frontend consumes the API exclusively through a client generated from that schema (e.g. `openapi-typescript` + a thin fetch wrapper). No hand-written `fetch('/api/...')` calls with inline, re-typed response shapes anywhere in frontend code — see `coding-standards.md`.

## Resource design

- Endpoints are resource-oriented and map onto `domain-model.md` entities: `/organizations`, `/organizations/{id}/memberships`, `/skill-assignments`, etc. Don't design RPC-style action endpoints (`/verifyOrganization`) where a resource-oriented shape works (`POST /organizations/{id}/verifications`).
- Nesting reflects real ownership from `domain-model.md` (`Membership` belongs under both `Person` and `Organization` conceptually — expose both `/organizations/{id}/memberships` and `/people/{id}/memberships` as views over the same resource, not two divergent models).
- Polymorphic entities (`Verification.subjectType`/`subjectId`, `SkillAssignment`, `Interests`, `Relationship` — see `domain-model.md`) are exposed as their own top-level resource (`/verifications`, `/skill-assignments`) with `subjectType`/`subjectId` as filterable fields, not duplicated under every possible parent path.

## Versioning

- Version in the URL path (`/v1/...`) from the first release. Breaking changes ship as a new version; the previous version stays live for a defined deprecation window (specify the window per breaking change, document it in the changelog — don't leave it implicit).
- Additive changes (new optional field, new endpoint) don't require a version bump.

## Request/response conventions

- All request/response bodies are validated against the DTO schema at the boundary (NestJS `ValidationPipe` + `class-validator`, or `zod` — pick one and apply consistently). Invalid input never reaches application-layer code.
- Timestamps: ISO 8601, UTC, always. No epoch integers, no local-time strings.
- IDs: UUIDs as strings, matching `domain-model.md`.
- Pagination: cursor-based for anything that can grow unbounded (`/relationships`, `/verifications`), offset-based is acceptable only for genuinely small, bounded lists (e.g. `/skills` categories). Response envelope includes the cursor for the next page, never requires the client to compute an offset.

## Error format

Every error response has a stable shape:

```json
{
  "error": {
    "code": "VERIFICATION_ALREADY_EXISTS",
    "message": "A verification of this type already exists for this subject.",
    "details": {}
  }
}
```

- `code` is a stable, machine-readable, SCREAMING_SNAKE_CASE identifier — frontend code branches on `code`, never on `message` (which is for humans and may be localized later).
- HTTP status communicates the error *category* (400 validation, 401/403 auth, 404 not found, 409 conflict, 422 domain-rule violation, 5xx server fault); `code` communicates the specific cause. Don't invent new HTTP statuses for domain-specific conditions — use 422 with a specific `code` instead.
- See `coding-standards.md`'s Error Handling section for how this maps back to backend exception types.

## Authentication and authorization

- Every non-public endpoint requires a valid Keycloak-issued token (see `security.md`, [ADR-0004](./adr/0004-keycloak-authentication.md)). Token validation happens once, at the API gateway/guard layer — never re-implemented per-endpoint.
- Authorization (can this `Account`, acting via which `Membership`, do this to this `Organization`) is checked in the application layer, using `Membership.role` from `domain-model.md` — not left to the frontend to enforce by hiding buttons. The frontend hiding a button is a UX nicety; the backend check is the actual security boundary.

## Idempotency

- All `POST` endpoints that create a resource with real-world side effects (initiating a `Verification`, creating a `Relationship` request) accept an `Idempotency-Key` header and deduplicate on it. This matters specifically for `Relationship`/`Verification` creation, where a retried request must not create a duplicate connection or a duplicate trust claim.

## Events vs. API

Domain events (per `architecture.md`'s Event-Ready principle) are an internal concern, not exposed as a public API surface until there's a concrete need (webhooks for integration partners, say) — and when that need arrives, it gets its own ADR and its own section here, not an ad hoc addition.
