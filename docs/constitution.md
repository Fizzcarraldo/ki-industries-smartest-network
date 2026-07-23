# Constitution

> This document holds the rules that do not change with a feature, a deadline, or a preference. Everything else in `docs/` can be refined as the product and team learn more; this document changes only through a deliberate, explicit amendment — never silently, never as a side effect of an unrelated change.

Every future change to this project — human-authored or AI-authored — must be compatible with every rule below. If a change cannot be made compatible with one of these rules, the change is wrong, or the rule needs a deliberate, documented amendment (with its own ADR explaining why) — not a quiet exception.

## I. The vision governs the architecture, not the other way around

`vision.md` and `product-principles.md` describe *why* smartest.network exists. Every architectural and technical decision in `architecture.md`, `technology-stack.md`, and `domain-model.md` exists to serve that vision. If a technical decision and the vision ever conflict, the vision wins and the technical decision is revisited — not the reverse.

## II. Trust is structural, not decorative

Per `domain-model.md`: no entity self-declares its own trust. Every trust-affecting fact traces to a `Verification` record with a source, a method, and a timestamp. This is not a feature that can be simplified away under deadline pressure — a trust signal that can be forged by the party it describes is worse than no trust signal at all, because it's actively misleading.

## III. Structured data over free text, always

Per `product-principles.md` principle 3: if a concept can be modeled, it is modeled — a typed field with a bounded shape, not a text box the system tries to interpret later. This applies to every new feature, without exception, unless the content is genuinely and irreducibly unstructured.

## IV. No business logic outside the backend domain layer

Per `coding-standards.md` and `architecture.md`: matching, scoring, trust calculation, and verification rules live in the backend's `domain`/`application` layers. Never in the frontend, never in a database trigger, never in infrastructure glue code. This is what keeps the trust guarantees in Rule II actually true rather than merely claimed.

## V. Every architectural decision is written down before it's built

A new module boundary, a new external dependency that isn't already in `technology-stack.md`, a change to `domain-model.md`, a change to how modules communicate — each requires an ADR in `docs/adr/` *before* implementation, not as documentation-after-the-fact. See `CLAUDE.md` for how this applies to AI-assisted changes specifically.

## VI. Current state is documented honestly, not aspirationally

Every document in `docs/` that describes both current and target state keeps them clearly, visibly separate (this was true when this documentation set was created — see the Engineering Report — and must stay true as the codebase evolves). A document that lets the target architecture bleed into looking like it's already built is a document that will mislead the next person who reads it. When code changes, the "current state" sections that describe it are updated in the same change — not left to drift.

## VII. Simplicity is a feature, not a shortcut

Per `product-principles.md` principle 6: a smaller, well-modeled system beats a larger, loosely-modeled one. This is not permission to skip the layering, testing, or documentation rules above — it's a constraint against speculative generality, premature abstraction, and unmodeled escape hatches (generic metadata columns, unbounded config flags, "just in case" fields). Simplicity and rigor are the same value here, not opposites.

## VIII. European by default

Per `product-principles.md` principle 7 and `security.md`: data residency, hosting, and vendor choices default to EU-based or self-hostable options for anything touching participant data. A US-hosted SaaS dependency for core product data is an exception requiring an explicit ADR, not a default.

## IX. Nothing ships that regresses accessibility or breaks on mobile without being caught

Per `ui-principles.md`: Lighthouse accessibility 100 is the maintained bar on any page it currently passes on. Responsive/mobile behavior is verified with real tooling against real viewports before being called done — this codebase has a documented history of CSS that looked right and wasn't; that history is the reason this rule exists, not a hypothetical.

## X. This constitution is amendable, but never silently

A rule here can change. It changes by: a PR that edits this file, states which rule is changing and why, and is reviewed with the same weight as an ADR (see `constitution.md`'s own governance is itself `adr/README.md`-equivalent — treat an amendment to this file as requiring the same rigor as the highest-stakes ADR in the project). It does not change by a rule being quietly ignored often enough that it stops being true.
