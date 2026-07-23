# Product Principles

> Status: **Engineering rules — authoritative.** These are the operational translation of [`vision.md`](./vision.md). Where `vision.md` says *why*, this document says *what that means when you're deciding whether to build something.*

Each principle below states the rule, then what it forbids in practice — the forbidding half is the part that actually changes behavior.

## 1. Companies are the unit, not individuals

The network exists to connect *organizations* (and the people who represent them acting in that capacity), not to be a general-purpose professional social graph.

**In practice:** `Person` is never a root entity on its own — it only exists attached to an `Account` and, through `Membership`, to an `Organization` (see [`domain-model.md`](./domain-model.md)). Do not build features that treat a `Person` as a free-floating profile independent of the organization they represent (e.g. no "personal brand" feed, no follower counts on individuals).

## 2. Trust is earned, not declared

A `TrustScore` or a `Verification` is never something a user sets about themselves. It is always derived from a verifiable external fact (a validated business registration, a completed cooperation with an outcome, a credential check) or issued by another verified party.

**In practice:** No profile field where a user just types "Trusted Partner" or self-assigns a badge. Every trust-affecting field must trace back to a `Verification` record with a source and a timestamp. If you can't answer "verified by whom, based on what, when," it's not a trust signal — it's a claim, and claims get labeled as claims in the UI.

## 3. Structured data before free text

If a concept can be modeled (a skill, a location, an industry, a cooperation status), it must be a modeled field with a defined type and a bounded vocabulary — not a text field the user fills in freely and the system tries to parse later.

**In practice:** `Skill` is a reference-table entity linked via `SkillAssignment`, not a comma-separated string on a profile. `Location` is structured (see `domain-model.md`), not a free-text "based in..." field. Free text is allowed for genuinely unstructured content (a cooperation's project description, a message) but never as a *substitute* for a field that should be structured — if you're tempted to add a text field for something with a natural fixed shape, model it instead.

## 4. Every feature must strengthen the network effect

Before a feature is built, its proposal states — concretely — whose future match, recommendation, or trust signal it improves, and how. "It increases engagement" is not an answer to this question.

**In practice:** Features that only serve a single user's convenience in isolation (a personal dashboard widget, a cosmetic profile customization) are legitimate but low priority by default, and should never be built ahead of anything that touches `Relationship`, `Verification`, or `TrustScore` quality. When prioritizing a backlog, connection-and-trust-quality features outrank engagement features unless there's a specific, stated reason otherwise.

## 5. No features without a stated, falsifiable benefit

A feature proposal must state who benefits and how you'd know if it didn't work. "Users have asked for X" is a valid *trigger* to investigate, not by itself a valid justification to build.

**In practice:** Feature proposals in `docs/adr/` or PR descriptions should be able to fill in: *"We expect this to [specific, measurable outcome]. We'll know it failed if [specific signal]."* If that can't be filled in, the feature isn't ready to build yet — narrow the scope until it can.

## 6. Simplicity beats complexity, even at the cost of short-term completeness

Given a choice between a small, well-modeled surface and a large, loosely-modeled one, ship the small one. This applies to the domain model, the API surface, and the UI equally.

**In practice:** Don't add optional fields "in case we need them later" — extend the model when a real feature needs the field, following [`coding-standards.md`](./coding-standards.md)'s stance against speculative generality. Don't ship a generic "custom fields" / metadata-bag escape hatch on core entities — see Principle 3. A migration to add a column is cheap; a `jsonb` catch-all field that silently absorbs a decade of unmodeled data is expensive to ever get out of.

## 7. European by default

The product's positioning ("Europas intelligente Infrastruktur") is not just marketing copy — it constrains infrastructure and data-handling decisions. Hosting, data residency, and authentication defaults are chosen to be defensible under EU/GDPR expectations without a later migration (see [`security.md`](./security.md) and [ADR-0005](./adr/0005-hetzner-cloud-infrastructure.md)).

**In practice:** Don't default to US-based SaaS dependencies for anything that touches participant or cooperation data. Where a EU-hosted or self-hostable alternative exists at comparable engineering cost, prefer it.

## 8. Demo and product are different things, and the code must say so

The current public site (`index.html`, `vision.html`) is explicitly a demo with fictional example content, clearly labeled as such in the UI itself. This distinction is deliberate and must be preserved: marketing/demo surfaces are allowed to move fast and look ahead of the real product; anything that claims to be the actual product (real accounts, real data, real trust scores) must be backed by the domain model and verification chain described in `domain-model.md` — never faked to look further along than it is.
