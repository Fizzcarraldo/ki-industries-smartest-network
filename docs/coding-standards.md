# Coding Standards

> Status: **Target — applies to all TypeScript code written for the platform (Next.js frontend, NestJS backend).** The current repository is plain HTML/CSS/JS with no build step (see `technology-stack.md`) and these rules don't retroactively apply to it — but no *new* JavaScript added to the current static site should violate the spirit of them either (no inline business logic, no untyped sprawling state).

## TypeScript

- **`strict: true` in every `tsconfig.json`, no exceptions.** `noImplicitAny`, `strictNullChecks`, `strictFunctionTypes` all on. A PR that weakens `tsconfig.json` strictness to make code compile is rejected — fix the code.
- **No `any`.** Not `any`, not `as any`, not a lint-disable comment to smuggle one through. If a type is genuinely unknown (e.g. a third-party payload before validation), use `unknown` and narrow it — via a runtime validator (e.g. `zod`) at the boundary, not a cast.
- **No non-null assertions (`!`) as a substitute for a real null check**, except where a preceding line makes non-nullness structurally guaranteed and a comment says why.
- **Prefer `type` for data shapes, `interface` for anything meant to be implemented/extended** (e.g. repository interfaces per `architecture.md`'s Clean Architecture layering). Be consistent within a module.

## Architecture boundaries (enforced, not just conventional)

- **No business logic in frontend code.** Matching, scoring, trust calculation, verification rules — all of it lives in the NestJS backend's `domain`/`application` layers (`architecture.md`). The frontend calls the API and renders the result. This is not just a style preference: per `domain-model.md`, trust guarantees only hold if the calculation can't be bypassed or forged client-side.
- **No framework imports in `domain/`.** A file under a module's `domain/` directory imports no `@nestjs/*`, no Drizzle, no HTTP client. If a domain entity needs persistence, that's a repository *interface* in `domain/`, implemented in `infrastructure/`.
- **No cross-module reach-through.** Module A never imports Module B's Drizzle schema, repository, or internal types directly. Module A depends on Module B's public application-service interface only. If that interface doesn't expose what's needed, extend it — don't reach around it.
- **No circular dependencies**, module-to-module or file-to-file within a module. If two modules need each other, one of them owns the shared concept and the other depends on it one-way — or the shared concept is factored out into its own module. CI should fail a build that introduces a cycle (`madge` or equivalent, wired into `development-workflow.md`'s CI once it exists).

## Dependency Injection

- All application and infrastructure services are constructor-injected via NestJS's DI container. No service-locator pattern, no manually-instantiated singletons reaching across module boundaries.
- Repository interfaces live in `domain/`; concrete Drizzle-backed implementations in `infrastructure/`, bound via a NestJS provider token — this is what makes `application/` layer unit tests possible without a database (see `testing.md`).

## Naming

- Names describe what something *is* or *does*, not how it's implemented. `TrustScoreRepository`, not `TrustScoreDrizzleTable`. `VerifyOrganizationUseCase`, not `VerifyOrganizationHelper`.
- Entity and field names match `domain-model.md` exactly. If the model says `SkillAssignment.proficiency`, the code says `proficiency`, not `skillLevel` or `rating`. Drift between the doc and the code is a bug in one of them — fix whichever is wrong, don't let them diverge.
- Boolean fields/variables read as a yes/no question: `isVerified`, `hasActiveMembership`. Not `verified` (ambiguous: is it a state or a past-tense event?) or `status` (too vague — use an enum type with a specific name instead, e.g. `MembershipStatus`).

## No magic strings

- Enum-like values (`Membership.role`, `Verification.method`, `Relationship.type` — see `domain-model.md`) are TypeScript union types or enums, defined once, imported everywhere. Never a bare string literal compared with `===` scattered across the codebase.
- Route paths, queue job names, event names: named constants in one place per module, not string literals repeated at each call site.

## No speculative generality

- Don't add a field, a config option, or an abstraction layer "in case we need it later." Per `product-principles.md` principle 6, extend the model when a concrete feature needs it. A migration is cheap. Unused flexibility is a liability someone has to understand and maintain forever.
- No generic `metadata: jsonb` catch-all columns on core entities (`Person`, `Organization`, `Relationship`, etc.) — this directly contradicts `product-principles.md` principle 3 and `domain-model.md`'s entire premise. If it's a real field, model it as a column and put it in `domain-model.md` via an ADR.

## Error handling

- Domain/application-layer errors are typed exceptions (or a Result-type pattern — pick one convention per backend and apply it consistently; don't mix `throw` and `Result<T, E>` within the same module), never a bare `throw new Error("something went wrong")` with a string a caller has to pattern-match.
- Every error that crosses the API boundary maps to a defined error shape (see `api-guidelines.md`) with a stable machine-readable code, not just an HTTP status and a human sentence.
- Don't swallow errors. A `catch` block that only logs and continues must have a comment explaining why continuing is safe — otherwise let it propagate.

## Logging

- Structured logging only (JSON), never `console.log` of a formatted string, so logs are queryable in Loki (`monitoring.md`).
- Log at the boundary where context is richest (application-service layer), not deep inside domain logic — domain code stays pure and framework-free per the architecture rules above.
- Never log secrets, tokens, or full `Verification.evidenceRef` contents. See `security.md`.
- Every log line that's part of handling a request carries a request/trace ID (OpenTelemetry context propagation — `technology-stack.md`), so a single user-facing error can be traced end-to-end across modules.

## Testability

- If a class needs a running database or HTTP server to unit test, it's in the wrong layer — see `testing.md` for the layer-by-layer testing strategy this maps to.
- Application-layer use cases are testable with in-memory fakes of their repository interfaces. This is the payoff of the DI + Clean Architecture rules above, not a separate effort.

## Formatting and linting

- One formatter (Prettier), one linter (ESLint with `@typescript-eslint`), configured once at the monorepo root, not per-package. No PR debates about formatting — the tool decides, `--fix` resolves disagreements.
- Lint rules enforcing the boundaries above (no cross-module imports, no framework imports in `domain/`) should be automated (e.g. `eslint-plugin-boundaries` or a custom rule) as soon as the module structure exists — don't rely on reviewers catching violations by eye indefinitely.
