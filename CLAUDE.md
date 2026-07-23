# CLAUDE.md

Working agreement for Claude Code (and any AI assistant) in this repository. This file is itself governed by [`docs/constitution.md`](./docs/constitution.md) — if something here ever conflicts with the constitution, the constitution wins and this file is wrong and should be fixed.

## Read this first: what's actually in this repository

This repo currently contains a **static marketing/demo site** (`index.html`, `vision.html`, the `_ds/` design system, `support.js`, `image-slot.js`) deployed to GitHub Pages. It does **not** contain the smartest.network product platform (no backend, no database, no auth). The full target architecture — modular monolith, NestJS, PostgreSQL, Keycloak, etc. — is documented in `docs/` as the direction that work follows once it begins, not as something already built. See [`docs/README.md`](./docs/README.md) for the full current-vs-target framing before you assume anything is already wired up.

## Before every change

In this order, every time, not just the first time:

1. **Understand the architecture.** Read [`docs/architecture.md`](./docs/architecture.md) for the layer/module the change touches. If the change is to the current static site, read the "Current State" section of the relevant docs (`docs/deployment.md`, `docs/ui-principles.md`) — don't assume target-architecture rules (NestJS layering, etc.) apply to code that doesn't have a backend yet.
2. **Reuse existing components and patterns before writing new ones.** For the current site: the `_ds/industry-.../styles.css` design system already has the component you need more often than not — check `docs/ui-principles.md` and the design system's own readme before adding a parallel class or a new visual pattern. For the target platform: check `docs/domain-model.md` before inventing a new entity or field, and check whether an existing module owns the concept before creating a new one.
3. **Read [`docs/constitution.md`](./docs/constitution.md).** Its rules apply regardless of how urgent or small the change feels.
4. **Check `docs/adr/` for a relevant decision.** If an ADR already covers the technology or pattern in question, follow it. If the change you're about to make would contradict an existing ADR, that's a conflict — see below, don't just proceed.
5. **Check `docs/product-principles.md`** if the change is product-facing (a new feature, a UI change) — does it strengthen the network effect, does it keep structured data over free text, does it have a stated benefit?

## When a request conflicts with what's documented

**Do not just implement it anyway.** This applies whether the conflict is with the constitution, an ADR, the domain model, or an established pattern in the code.

Instead:
1. **Explain** what the conflict is, specifically — cite the document and rule.
2. **Justify** why the existing rule/pattern exists (the docs generally explain their own reasoning — use it, don't just cite the rule number).
3. **Propose an alternative** that satisfies the actual underlying need without the conflict, if one exists.
4. Only proceed with the original request if the user explicitly confirms after hearing the above — and if it represents a real, deliberate architectural change, it needs an ADR (see below) as part of the same piece of work, not a follow-up "someday."

This mirrors how work in this repository has actually been done historically: when a mobile-layout approach turned out to conflict with how CSS sticky positioning actually behaves, the response was to explain what was actually happening, test the alternative, and document the reasoning in the commit message — not to force the original approach through. Keep doing that.

## No architecture decisions without documentation

A new module boundary, a new external dependency not already in `docs/technology-stack.md`, a change to `docs/domain-model.md`, a new cross-cutting pattern — each gets an ADR in `docs/adr/` (see [`docs/adr/README.md`](./docs/adr/README.md) for the format), written **before** the implementation, in the same piece of work. "I'll document it later" is not an acceptable substitute — write the ADR first, then implement against it.

## No breaking changes without justification

A breaking API change, a domain-model change that invalidates existing data's meaning, a removal of a documented pattern — each needs its reasoning stated explicitly (in the PR description at minimum, in an ADR if it's architecturally significant per the rule above). "It's cleaner this way" is a reason but needs a sentence, not just an assertion — what breaks, who's affected, what's the migration path.

## Verify, don't assume

This repository's real development history (see `git log`) has repeatedly found that something which looked correct locally or in isolation broke on the live site, on mobile, or across browsers. Before calling a change done:
- For the current static site: verify against the **live deployed URL**, not just a local server — see `docs/deployment.md` and `docs/testing.md`. Use a real headless-browser tool across Chromium/Firefox/WebKit and real mobile viewport sizes for anything layout- or interaction-related.
- For the target platform, once it exists: follow the test-layer requirements in `docs/testing.md` — don't consider a change done because it worked in manual testing once.

## Sensitive content check

Any new generated or geometric visual content (diagrams, icons, illustrations) gets a deliberate check for unintended symbolic resemblance before shipping — this project has hit this exact issue once already (a network diagram whose connection pattern read as an unintended symbol) and caught it. Don't skip this check because a rendering "looks cool" or "looks technical" — look at it deliberately for what it resembles, not just whether it's visually appealing.

## Keeping the Architecture Review current

[`docs/architecture-review.md`](./docs/architecture-review.md) is a living document, not a one-time snapshot. Update it whenever a change:

- introduces a new module
- changes module boundaries
- extends the technology stack
- requires new infrastructure
- changes authentication or authorization
- materially extends the domain model
- introduces a new external dependency
- affects security or privacy risk

After a larger implementation, check whether the review's scores, risks, or recommended measures need to change as a result. Don't leave a finding marked as an open risk once you've actually fixed it, and don't leave a score stale once the thing it was scoring has materially changed — either update it in the same change, or explicitly note in the PR/commit why it still applies unchanged.

## Keep the documentation honest

If you change code that a `docs/` file describes, update that file in the same change. If you find a doc that's already wrong, fix it as part of your change rather than working around the discrepancy silently. See `docs/README.md`'s "Keeping this documentation honest" section.
