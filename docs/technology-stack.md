# Technology Stack

> Status: **Mixed — current stack verified against source; target stack is the mandated direction for all platform work.**

## Current Stack (what's actually in this repo)

| Layer | Technology | Notes |
|---|---|---|
| Markup/runtime | Plain HTML + a vendored `dc-runtime` (`support.js`) | Renders React (loaded from `unpkg.com` at runtime) against an `<x-dc>` template format. Not a project-owned framework — see `architecture.md` |
| Styling | Plain CSS, custom "Industry" design system (`_ds/.../styles.css`) | No CSS framework, no preprocessor, no CSS-in-JS. See `ui-principles.md` |
| Fonts | Google Fonts (`@import` in `styles.css`) | Barlow / Barlow Condensed |
| Icons | Lucide, loaded from `unpkg.com` at runtime | |
| Interactivity | Vanilla JS (`requestAnimationFrame`-throttled scroll handlers, `IntersectionObserver` avoided in favor of geometry checks — see git history on `vision.html` for why) | No frontend framework state management exists |
| Analytics | Umami (self-hosted at `stats.scanventure.de`), loaded via `<script>` tag | **Not** PostHog or Matomo — see "Gap" below |
| Hosting | GitHub Pages | Custom domain via `CNAME` + `.nojekyll` |
| Build | None | No `package.json`, no bundler, no transpilation step |
| Tests | None | Verification during development is manual, via Playwright launched ad hoc (not committed to the repo) — see `testing.md` |

This is a complete, coherent stack **for a static marketing site** — it is simple, has zero infrastructure to maintain, and does its job. It is not, and should not become, the stack the product (accounts, matching, trust, cooperation) is built on. See `architecture.md`.

## Target Stack (product platform)

### Frontend

| Technology | Role |
|---|---|
| **Next.js** | Application framework, App Router. Server Components by default |
| **React** | UI layer |
| **TypeScript** | Strict mode, no exceptions — see `coding-standards.md` |

### Backend

| Technology | Role |
|---|---|
| **NestJS** | Application framework — module system maps directly onto `architecture.md`'s modular-monolith boundaries |
| **TypeScript** | Strict mode |

### Data

| Technology | Role |
|---|---|
| **PostgreSQL** | Source of truth for all domain data (see `domain-model.md`). No secondary "convenience" datastore duplicating it without a stated reason |
| **Drizzle ORM** | Schema definition + query layer. Chosen over alternatives — see [ADR-0003](./adr/0003-drizzle-orm.md) |

### Identity

| Technology | Role |
|---|---|
| **Keycloak** | Authentication and identity provider (OIDC). Backs the `Account` entity in `domain-model.md`. See [ADR-0004](./adr/0004-keycloak-authentication.md) |

### Storage

| Technology | Role |
|---|---|
| **Hetzner Object Storage** | Binary assets — profile photos, logos, verification evidence documents (`Verification.evidenceRef` in `domain-model.md`) |

### Async / Queue

| Technology | Role |
|---|---|
| **Valkey** | In-memory store backing the queue (Redis-compatible fork) |
| **BullMQ** | Job queue on top of Valkey — handles the async event handlers described in `architecture.md`'s "Event-Ready" section (e.g. `TrustScore` recomputation, verification-expiry checks, notification delivery) |

### Infrastructure

| Technology | Role |
|---|---|
| **Hetzner Cloud** | Compute — see [ADR-0005](./adr/0005-hetzner-cloud-infrastructure.md) |
| **OpenTofu** | Infrastructure provisioning (open-source Terraform fork) |
| **Ansible** | Configuration management on provisioned hosts |
| **Docker** | Containerization for backend, frontend, and their dependencies |

### Observability

| Technology | Role |
|---|---|
| **OpenTelemetry** | Instrumentation standard — traces/metrics/logs emitted in OTel format from day one, not retrofitted |
| **Grafana** | Dashboards |
| **Loki** | Log aggregation |
| **Prometheus** | Metrics storage |

See `monitoring.md` for what "day one" instrumentation actually means in practice.

### Analytics

| Technology | Role |
|---|---|
| **PostHog or Matomo** | Product analytics — final choice not yet made, see `monitoring.md` |

## Gap: current vs. target

| Area | Current | Target | Migration implication |
|---|---|---|---|
| Analytics | Umami | PostHog or Matomo | The current site's Umami integration is fine for a marketing-only demo site and doesn't need to change on its own timeline — but it should **not** be assumed to be the product's analytics tool. Decide PostHog vs. Matomo before any product-side event tracking is built, don't default to "just add Umami events too" |
| Everything else in this table | Does not exist | Full stack above | See `architecture.md` and the Engineering Report for sequencing — this is not a migration of existing code, it's new construction |

## Rules for introducing a new dependency

1. It must serve the target stack above, or be justified in an ADR if it's a genuine addition to the stack (not a swap).
2. Prefer the already-chosen tool over adding a second tool that does something similar (e.g. don't add a second queue technology alongside BullMQ "for one specific job type" — extend the existing one, or write an ADR explaining why it's insufficient).
3. Self-hostable / EU-hosted alternatives are preferred over US SaaS dependencies for anything touching participant data, per `product-principles.md` principle 7.
