# Testing

> Status: **Current section describes the real (manual, ad hoc) verification process used on this repository. Target section is mandated for the product platform, where "verify manually with Playwright before pushing" is not sufficient once there's a database and real users.**

## Current State

**There is no automated test suite in this repository.** No `package.json`, no test runner, no CI. Verification of every change in this repository's history has been manual: a local static server, Playwright (Chromium/Firefox/WebKit) launched ad hoc from a scratch directory, checking console errors, layout via screenshots, and Lighthouse accessibility scores against both a local copy and the live deployed site.

This has been effective for a static site with a small, visually-inspectable surface — but it does not scale to a backend with business logic, and it leaves no regression protection: nothing stops a future change from silently breaking something a past manual check caught. This is flagged as technical debt (see the Engineering Report) specifically because the *pattern* of "verify with a real browser against real behavior, not just assumption" is worth keeping — the *absence of an automated, repeatable version of it* is the debt.

## Target: testing strategy for the platform

Layered, matching `architecture.md`'s Clean Architecture layering — each layer is tested at the cheapest level that actually exercises it.

### Unit tests — `domain/` and `application/` layers

- Domain entities and pure business rules (e.g. `TrustScore` calculation logic, `Verification` validity rules): plain unit tests, no framework, no database, no mocking beyond simple test doubles.
- Application-layer use cases: unit tested against **in-memory fakes of repository interfaces** (not mocks that just assert call counts — real, if simplified, in-memory implementations that behave like the real repository). This is the direct payoff of `coding-standards.md`'s DI + Clean Architecture rules — if a use case can't be tested this way, the layering has been violated somewhere and that's the bug to fix, not a reason to reach for a heavier test setup.
- Target: every application-layer use case has unit tests covering its success path and its documented failure modes (the `api-guidelines.md` error codes it can produce).

### Integration tests — `infrastructure/` layer

- Drizzle repository implementations tested against a real (containerized, ephemeral) PostgreSQL instance — not mocked. An ORM query that's syntactically valid but semantically wrong is exactly the kind of bug that only a real database catches.
- Run against the same Postgres version as production (pinned in `docker-compose.yml` per `deployment.md`).

### API/contract tests

- Each NestJS controller endpoint has a test asserting its request/response shape matches the OpenAPI schema it generates (`api-guidelines.md`) — this is what keeps "API-first" honest over time instead of drifting.
- Auth/authorization tests specifically: verify that a `Membership` with insufficient `role` is actually rejected server-side for every mutating endpoint — this is a security control (`security.md`), not just a feature test, and deserves explicit, deliberate coverage rather than incidental coverage from happy-path tests.

### End-to-end tests

- Critical user journeys (sign up → verify organization → connect → cooperate) tested end-to-end against a real running stack (frontend + backend + Postgres + Keycloak, via Docker Compose), using Playwright — continuing the tool choice already proven useful in this repository's manual verification history, now automated and committed.
- E2E tests are expensive to run and maintain — reserve them for genuinely critical, cross-system journeys, not for coverage of individual UI states (that's a frontend component test's job).

### Frontend component tests

- React components tested in isolation (e.g. with React Testing Library) for behavior and accessibility (ARIA roles, keyboard interaction) — not for pixel-perfect visual output.
- Any component that reimplements the scroll-driven animation pattern proven out in `vision.html` (see `ui-principles.md`) gets a specific test verifying its scroll-state logic doesn't regress the bugs already found and fixed once in that page's history (phase-detection skipping under fast scroll, sticky/overlap issues) — those bugs are subtle enough that "it looked right when I checked" is not sufficient assurance twice.

### CI gate

Once the platform repository exists: unit + integration + contract tests run on every PR, blocking merge on failure. E2E tests run on every PR against a fresh ephemeral environment, or at minimum on every merge to `main` before promotion to staging (`deployment.md`) — decide the exact gate when CI is set up, but "tests exist but don't block anything" is not an acceptable end state.

## What "done" means for a feature, going forward

A feature is not done when it works in manual testing. It's done when: unit tests cover its application-layer logic, an integration test covers any new repository query, and — if it's a user-facing journey — either an existing E2E test covers it or a new one was added. This bar starts applying the moment the platform's first module is built, not retroactively to the current static site.
