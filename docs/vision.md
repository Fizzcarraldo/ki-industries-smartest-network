# Vision

> Status: **Product direction — authoritative.** This document describes what smartest.network is *for*. It does not describe what's built yet; see [`architecture.md`](./architecture.md) and the [Engineering Report](./engineering-report.md) for the gap between this vision and the current codebase.

## What we are building

smartest.network is not a business networking platform. It is **Europe's intelligent infrastructure for enterprise cooperation.**

The distinction matters because it constrains every subsequent decision in this repository:

- A "platform" optimizes for engagement, reach, and content volume.
- **Infrastructure** optimizes for reliability, trust, and the quality of a small number of high-value connections.

We connect companies, experts, research institutions and organizations — not to maximize how much they talk to each other, but to maximize how often that contact turns into a real, structured, verifiable cooperation.

## The core mechanism: the network effect

The product's entire value proposition rests on one compounding loop:

```
Identity → Connections → Cooperation → Trust → Intelligence
```

Every participant who joins with a verified identity makes the network's matching more accurate for everyone already in it. Every successful cooperation produces a data point (who worked with whom, on what, with what outcome) that makes the *next* match better. This is why the domain model treats `Verification`, `Relationship`, and `TrustScore` as first-class entities rather than incidental metadata (see [`domain-model.md`](./domain-model.md)) — they are the mechanism, not decoration.

This is also why growth-stage metrics that are normal for social platforms (DAU, session length, content posted) are explicitly **not** north-star metrics here. The metric that matters is: *does a connection made through the network turn into a verifiable cooperation, and does that cooperation improve the accuracy of the next recommendation.*

## Non-negotiable priorities

These are listed in priority order. When two of them conflict, the higher one wins — this is not a menu to pick from per feature.

1. **Trust over reach.** A smaller network of verified, accountable participants is worth more than a larger network of anonymous ones. Features that trade verification friction for signup volume are rejected by default.
2. **Structured data over free text.** A skill, a competency, a cooperation outcome — these are modeled entities with defined shapes (see `domain-model.md`), not blobs of text a human has to re-read to extract meaning from. Free text is permitted only where the content is genuinely unstructured (a message body, a project description) and even then it lives alongside structured fields, not instead of them.
3. **The network effect, always.** Every feature proposal should be able to answer: *whose future match does this improve, and how?* If the honest answer is "none, it's just a nice-to-have," the feature needs a different justification or doesn't ship.
4. **Simplicity over completeness.** We are building infrastructure other people build on top of and trust with their business relationships. A smaller, well-modeled surface area beats a large, half-modeled one.

## Who this is for

- **Mittelstand companies** who need innovation partners, digital transformation expertise, or pilot customers but don't have the internal reach to find them reliably.
- **Research institutions and universities** who need industry partners to turn research into applied outcomes.
- **Technology and service providers** who need to reach decision-makers with a real mandate, not cold-outreach volume.

The `Landingpage` in this repository (`index.html`) targets this audience directly today, even though the matching/verification/cooperation product behind it does not exist yet — see the Engineering Report for what that gap means in practice.

## What "intelligence" means here

The final stage of the network-effect loop — the platform actively surfacing opportunities, recommending partners, and improving its own recommendations — is a **consequence** of the trust and structured-data foundation, not a feature to bolt on early. An AI-powered recommendation system trained on a network with unverified identities and unstructured cooperation records would just be confidently wrong. `Verification` and `TrustScore` are prerequisites for `Intelligence`, in that literal order — this is reflected in the domain model's dependency direction and should be reflected in build sequencing too.

## Source of narrative truth

The public-facing articulation of this vision lives in [`vision.html`](../vision.html) (the "Vision & Entwicklung" page) as a five-stage scroll story: Identität → Verbindungen → Kooperation → Vertrauen → Intelligenz. That page and this document must stay in sync — if the narrative on the page changes, update this file in the same change, and vice versa.
