# ADR-0005: Hetzner Cloud Infrastructure

**Status:** Accepted

## Context

`vision.md` positions smartest.network explicitly as European infrastructure ("Europas intelligente Infrastruktur für Unternehmenskooperation"), and `product-principles.md` principle 7 makes EU data residency and self-hostability a default engineering constraint, not just marketing framing. The target platform (`technology-stack.md`) needs compute for the NestJS backend, Next.js frontend, PostgreSQL, Keycloak, and Valkey, plus object storage for `Verification.evidenceRef` and profile media (`domain-model.md`).

## Decision

**Hetzner Cloud** for compute, provisioned via OpenTofu (`deployment.md`), with **Hetzner Object Storage** for binary assets.

## Consequences

**Positive:**
- EU-headquartered and EU-region hosting available, directly satisfying `product-principles.md` principle 7 without a compliance workaround — data residency is structural, not a policy statement layered on top of US infrastructure.
- Cost-competitive compute relative to the hyperscalers (AWS/GCP/Azure), which matters for a small team funding its own infrastructure ahead of significant revenue.
- Straightforward API and Terraform/OpenTofu provider support, fitting the target provisioning approach (`deployment.md`) without exotic tooling.
- S3-compatible Object Storage API means the storage-client code in the backend (`architecture.md`'s infrastructure layer) uses a standard, replaceable interface rather than a proprietary SDK, keeping a future storage-provider change (if ever needed) a swap, not a rewrite.

**Negative / accepted costs:**
- Smaller managed-service ecosystem than a hyperscaler — no managed Postgres, managed Kubernetes-with-everything, or managed Keycloak equivalent. This is why `deployment.md` specifies self-managed Postgres/Keycloak via Ansible/Docker rather than reaching for managed offerings — an accepted operational responsibility in exchange for cost and data-residency control.
- Smaller global footprint (fewer regions) than a hyperscaler — acceptable given the product's stated European focus; this would need revisiting only if the product expands beyond a market where EU-region latency is acceptable.

## Alternatives Considered

- **AWS / GCP / Azure (EU regions):** technically satisfies data residency if configured carefully, but at meaningfully higher cost for equivalent compute, with a much larger surface of managed services that invite scope creep away from the deliberately lean target stack (`technology-stack.md`). Also carries US-parent-company legal exposure considerations (e.g. CLOUD Act) that a Hetzner-hosted, EU-headquartered setup avoids more cleanly — relevant given the product's trust-and-European-infrastructure positioning.
- **A smaller EU-native cloud provider (e.g. Scaleway, OVHcloud):** viable alternatives with similar residency properties; Hetzner was chosen for its cost/performance ratio and provider maturity. Not a strong rejection of the alternatives — could be revisited with a comparative ADR if Hetzner proves insufficient for a specific, encountered need (e.g. a required managed service Hetzner doesn't offer).
