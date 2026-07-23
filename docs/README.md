# smartest.network Engineering Documentation

This directory is the durable knowledge base for smartest.network — for human engineers and for AI coding assistants working in this repository. It replaces ad hoc context passed in prompts: if something here and something said in a chat conflict, **this documentation wins**, and the conflict should be resolved by updating the docs, not by quietly going with whatever the chat said.

Start with [`CLAUDE.md`](../CLAUDE.md) at the repository root if you're an AI assistant about to make a change — it tells you the order to read things in and what to do when a request conflicts with what's documented here.

## Read this first: current vs. target

**This is critical and easy to miss:** as of this documentation set's creation, this repository contains a static marketing/demo site (`index.html`, `vision.html`, the `_ds/` design system) and *nothing else*. There is no backend, no database, no authentication, no build system. Every document below that describes the product platform (accounts, organizations, trust, matching) is describing a **target architecture that does not exist in code yet.**

Every document that has both a current and a target reality says so explicitly, near the top, in a status line. Read that line before trusting anything a document says about "how this works" — it may mean "how this will work."

For the full picture of that gap and what to do about it, read the [Engineering Report](./engineering-report.md).

## Reading order

If you're new to this project, read in this order:

1. [`vision.md`](./vision.md) — what we're building and why. Everything else follows from this.
2. [`product-principles.md`](./product-principles.md) — the vision translated into engineering rules.
3. [`domain-model.md`](./domain-model.md) — the entities the product is built from.
4. [`architecture.md`](./architecture.md) — how the system is structured (current: static site; target: modular monolith).
5. [`constitution.md`](./constitution.md) — the rules that don't change.
6. Everything else, as needed for the task at hand.

## Document index

| Document | Covers |
|---|---|
| [`vision.md`](./vision.md) | Product vision, the network-effect mechanism, non-negotiable priorities |
| [`product-principles.md`](./product-principles.md) | Engineering rules derived from the vision |
| [`domain-model.md`](./domain-model.md) | Entities, relationships, responsibilities (target — not yet implemented) |
| [`architecture.md`](./architecture.md) | Current state (static site) and target architecture (modular monolith, DDD, Clean Architecture) |
| [`technology-stack.md`](./technology-stack.md) | Current and target technology choices, and the gap between them |
| [`coding-standards.md`](./coding-standards.md) | TypeScript, module boundaries, DI, naming, error handling, logging |
| [`api-guidelines.md`](./api-guidelines.md) | REST API conventions for the target backend |
| [`ui-principles.md`](./ui-principles.md) | The real "Industry" design system, accessibility and responsive rules |
| [`security.md`](./security.md) | Auth, authorization, data protection, secrets |
| [`deployment.md`](./deployment.md) | Current GitHub Pages workflow; target Hetzner/OpenTofu/Ansible/Docker |
| [`testing.md`](./testing.md) | Current manual verification process; target layered testing strategy |
| [`monitoring.md`](./monitoring.md) | Current minimal analytics; target OpenTelemetry/Grafana/Loki/Prometheus stack |
| [`development-workflow.md`](./development-workflow.md) | Branching, review, commit conventions, local dev |
| [`constitution.md`](./constitution.md) | The rules that don't change without deliberate amendment |
| [`adr/`](./adr/README.md) | Architecture Decision Records — the *why* behind major choices |
| [`engineering-report.md`](./engineering-report.md) | Assessment of the current codebase, technical debt, prioritized roadmap |
| [`architecture-review.md`](./architecture-review.md) | Formal, scored review across 21 areas (security, data privacy, dependencies, accessibility, testability, etc.) — the detailed evidence base the Engineering Report's conclusions rest on |

## Keeping this documentation honest

- A document describing existing code (`ui-principles.md`'s design-system section, `deployment.md`'s current-state section) is updated in the **same change** that changes the code it describes. Documentation drift is a bug.
- A change to the domain model, an architectural boundary, or a technology choice gets an ADR (`adr/`) before implementation, not after. See [`constitution.md`](./constitution.md) Rule V.
- If you find a document here that's wrong, out of date, or contradicts the code — fix it as part of whatever change you're making, don't work around it silently.
