# Development Workflow

> Status: **Current section describes this repository's actual working pattern. Target section is mandated once the platform repository/monorepo exists.**

## Current State

- **Single branch (`main`), no branch protection, no required review, direct push deploys** (see `deployment.md`). This has been workable because there has been one maintainer and a low-stakes static site — it is explicitly not the model for the product platform.
- **Commit messages** in this repository's history consistently explain *why*, not just *what* — several commits document a debugging journey (e.g. the mobile sticky-diagram saga across commits `91f8b56` → `d54f830` → `4867102` → `f9e88c8`) so that a future reader understands not just what changed but which approaches were already tried and rejected, and why. **Keep doing this.** A commit message that only restates the diff is a wasted opportunity — future-you (or the next contributor, human or AI) will hit the same dead end again without it.
- **No CI.** Verification is manual (`testing.md`) before every push.

## Rules that already apply, static site or not

- **New commits, not amended history**, except when explicitly asked otherwise — this is already standard practice in this repository's history.
- **Verify against the live deployed result, not just local state**, before considering a change finished (`deployment.md`, `testing.md`).
- **Explain non-obvious decisions in the commit message**, especially when an approach was tried and reverted — see the mobile-diagram commit sequence referenced above as the model to follow.

## Target: workflow once the product platform exists

### Branching and review

- Feature branches off `main`, merged via reviewed pull request. No direct pushes to `main` for the product repository — this is a hard change from the current static site's model, driven by the fact that the product handles real user data (`security.md`).
- Branch protection: required review, required CI pass (`testing.md`), before merge.

### Commit and PR conventions

- Continue the existing pattern of explaining *why* in commit messages, not just *what* — this is a cultural strength already present in this repository's history and should carry forward, not get lost in a "generic conventional-commits" template that only captures the *what*.
- PR description states: what changed, why, and — per `constitution.md` — whether it touches an architectural decision (in which case it links an ADR) or the domain model (in which case `domain-model.md` is updated in the same PR).

### Local development

- `docker-compose.yml` at the product repository root brings up Postgres, Valkey, and Keycloak locally, matching the versions used in `staging`/`production` (`deployment.md`) — no "works on my machine because I have a different Postgres version" class of bug.
- One command to run the full local stack (backend + frontend + dependencies); document it in the product repository's own root README once it exists, not just here.

### Definition of done

A change is done when (per `testing.md` and `coding-standards.md`):
1. It follows the architecture boundaries in `architecture.md` (module boundaries, Clean Architecture layering).
2. It has the appropriate test coverage for its layer.
3. It doesn't introduce a new dependency, architectural pattern, or domain-model change without the corresponding ADR/doc update (`constitution.md`).
4. CI passes.
5. It's been reviewed by another person (once the team is more than one person — until then, by `CLAUDE.md`'s "explain, don't just implement" rule when there's a conflict with existing patterns).

### On-call / incident response

Not yet defined — must exist before the platform has real users handling real cooperations (a `Verification` or `Relationship` failure is a business-trust incident, not just a technical one). Track as a concrete pre-launch item alongside the security-reporting process (`security.md`).
