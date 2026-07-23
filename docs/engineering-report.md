# Engineering Report

**Date:** 2026-07-23
**Scope:** Full assessment of the current codebase, produced alongside the `docs/` documentation set it accompanies.

## Summary

This repository, as it exists today, is a well-executed static marketing/demo site — nothing more, nothing less. It contains **zero** of the target product platform described elsewhere in `docs/` (no backend, no database, no authentication, no domain model in code). The gap between what's built and what's specified is the single most important fact in this report: everything below should be read with that gap as context, not as a list of problems with an existing platform.

## Assessment of the current architecture

There isn't a platform architecture to assess yet — there's a static-site architecture, and it's assessed on its own terms below.

### Strengths

- **The design system (`_ds/industry-.../`) is genuinely good and genuinely reusable.** Token-based, documented in its own readme, consistently applied across both pages. This is not thrown-away work — it should be ported as-is into the target Next.js frontend (`docs/ui-principles.md`), not rebuilt from scratch.
- **Accessibility is actively maintained, not incidental.** Both pages score 100 on Lighthouse accessibility as of this report, verified with real tooling, with a documented history of real bugs found and fixed (a CSS specificity bug silently darkening button text, a missing `<main>` landmark, contrast failures below WCAG AA). This standard of rigor is worth carrying into the platform build.
- **The commit history is unusually good at capturing *why*, not just *what*.** Several commits document a full debugging journey — approaches tried, why they failed, what was learned — rather than just the final diff. This is a real asset for anyone (human or AI) working in this codebase later, and the workflow rules in `docs/development-workflow.md` and `CLAUDE.md` are written specifically to preserve this habit going forward.
- **Verification discipline.** Every change in this repository's history was checked against the live deployed result, across multiple browser engines, at real viewport sizes — not just assumed correct from local testing. This caught real, non-obvious bugs (see Technical Debt, below) before they reached production.

### Weaknesses

- **No automated tests, no CI, no build system.** Every verification has been manual. This has worked because the surface area is small and visually inspectable — it will not work for a backend with business logic and real data, and there's currently no tooling in place to start that habit incrementally (`docs/testing.md`).
- **No branch protection, direct-push deploys.** Acceptable for a single-maintainer demo site (`docs/security.md`), a real liability if carried forward unchanged to a repository handling real user/business data.
- **The rendering mechanism (`dc-runtime` / `support.js`) is a vendored, generated artifact from an external design tool, not project-owned code.** It works, but it's a dead end to build further product surfaces on — it was never meant to be an application framework, and treating it as one would accumulate exactly the kind of undocumented, ungoverned complexity `docs/constitution.md` is designed to prevent.

### Technical risks

- **The single biggest risk is scope/expectation mismatch**, not a code defect: without the `docs/` set produced alongside this report, there was no written record that "the product" and "the current repository" are different things. That risk is mitigated by this documentation existing — but only if it's kept current (`docs/README.md`'s "keeping this documentation honest" section) rather than left to go stale the moment real platform work starts.
- **DNS/domain/certificate management for the current site is entirely manual and external to the repository** (Strato DNS, GitHub-managed certificates) — undocumented outside `docs/deployment.md` until now. If domain access is ever lost or transferred, there's no infrastructure-as-code record of the current setup, only the documentation.
- **No security-reporting process, no incident-response process exist yet** for either the current site or the future platform (`docs/security.md`, `docs/development-workflow.md` both flag this). Low urgency today (low-stakes static site), high urgency before the platform has real users.

## Technical debt (identified, not yet resolved)

| Item | Where documented | Severity |
|---|---|---|
| Entire product platform (backend, DB, auth, domain model) does not exist in code | `docs/architecture.md`, this report | N/A — this is the starting line, not debt in the traditional sense, but it's the item every other priority below is downstream of |
| No automated tests anywhere in the repository | `docs/testing.md` | High, once platform work starts; low today |
| No CI/CD pipeline, no branch protection | `docs/development-workflow.md`, `docs/security.md` | High, once real user data exists; low today |
| Umami used for analytics, not the target-specified PostHog/Matomo | `docs/technology-stack.md`, `docs/monitoring.md` | Low — acceptable for the marketing site indefinitely; decide the product-analytics tool separately, don't conflate the two |
| No error/performance/uptime monitoring on the live site | `docs/monitoring.md` | Low today, required before platform launch |
| `Relationship`'s `cooperation_*` states likely need a richer `Cooperation` entity | `docs/domain-model.md` | Deferred by design — flag, don't build speculatively (`product-principles.md` principle 6) |
| `TrustScore`'s actual computation algorithm is unspecified | `docs/domain-model.md` | Needs its own design work and ADR before the `trust` module is implemented |
| GDPR erasure vs. soft-delete/audit-trail tension unresolved | `docs/security.md` | Needs an ADR before the first real erasure request, not after |
| No `SECURITY.md` / vulnerability reporting process | `docs/security.md` | Needed before launch |

## Recommended priorities

### Short-term (weeks)

1. **Keep the current site's existing quality bar intact** while platform work begins — don't let accessibility/cross-browser rigor lapse because attention shifts to the backend.
2. **Stand up the product repository/monorepo skeleton**: NestJS + Next.js + TypeScript strict mode from the start (`docs/coding-standards.md`), Docker Compose for local Postgres/Valkey/Keycloak (`docs/deployment.md`), CI running lint + unit tests on every PR from the very first commit — don't defer CI until "there's enough code to test."
3. **Implement the `identity` and `organizations` modules first** — they're the foundation every other module (`skills`, `verification`, `relationships`, `trust`) depends on per `docs/domain-model.md`'s relationships, and they're the minimum needed to validate the Clean Architecture layering (`docs/architecture.md`) end-to-end on real code before more modules copy the pattern.
4. **Write the `TrustScore` computation ADR** before implementing the `trust` module — this is flagged as unspecified and is too central to the product's value proposition (`vision.md`) to implement ad hoc.

### Mid-term (months)

1. **Build out `skills`, `verification`, and `relationships` modules**, each following the pattern validated by `identity`/`organizations`.
2. **Stand up the target observability stack (OpenTelemetry → Grafana/Loki/Prometheus)** alongside the first modules, not after — per `docs/monitoring.md`, instrumentation from day one is the stated standard, not an aspiration.
3. **Provision staging + production on Hetzner via OpenTofu/Ansible** (`docs/deployment.md`), and cut over from "no environment" to a real staging→production promotion flow before the platform has its first real (non-test) user.
4. **Resolve the GDPR erasure/audit-trail tension** (`docs/security.md`) with a concrete design and ADR before it's needed in anger.
5. **Decide PostHog vs. Matomo** for product analytics (`docs/monitoring.md`) once there's a product to have analytics about.

### Long-term architecture vision

1. **The `Intelligenz` stage of the product vision** (`vision.md`) — active recommendation/matching — is the payoff of everything else being built correctly first. Per `vision.md`'s own reasoning, this should not be attempted until `Verification` and `TrustScore` have real production data behind them; an ML/recommendation layer on top of unverified, unstructured data would just be confidently wrong. When it's time, revisit ADR-0006's note about a possible narrowly-scoped Python-ecosystem service extraction for this specific module, under ADR-0001's established extraction path — don't let it justify a wholesale architecture change.
2. **Module extraction from the monolith** (ADR-0001) should remain the exception, done when a specific module has a concrete, stated reason (independent scaling, independent team ownership) — not a default trajectory. Revisit this ADR explicitly if that condition arises, rather than drifting into microservices piecemeal.
3. **This documentation set itself is a long-term asset only if maintained.** The single highest-leverage long-term practice this report can recommend is procedural, not technical: every PR that changes architecture, the domain model, or an established pattern updates the relevant `docs/` file in the same PR, enforced by review (`docs/development-workflow.md`) and by `CLAUDE.md`'s standing instruction to AI assistants working in this repository. A documentation set that drifts from the code it describes is worse than no documentation, because it's actively misleading — the discipline to keep it honest is the actual deliverable of this report, more than any individual document in it.
