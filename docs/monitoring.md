# Monitoring & Observability

> Status: **Current section reflects the real, minimal setup on the live site today. Target section is mandated for the product platform.**

## Current State

- **Umami** (self-hosted at `stats.scanventure.de`) is the only observability tool wired into the current site — page-view analytics only, loaded via a `<script defer>` tag with a `data-website-id` in both `index.html` and `vision.html`.
- **No error monitoring, no performance monitoring, no uptime monitoring, no logs** beyond what GitHub Pages itself reports. If the live site breaks, the first signal is a human noticing.
- One known, understood source of "noise" in the current analytics: automated Certificate-Transparency-log scanners and email-security link-scanners visit the site within hours of any new TLS certificate/domain going live, showing up as traffic from cloud-datacenter IP ranges (Ashburn, Boardman) and occasionally as a visited "path" that's actually a local filesystem path from a corporate email-scanning sandbox. This is expected background noise for any newly-provisioned domain, not a sign of real (or concerning) traffic — don't over-interpret early traffic spikes on a freshly deployed domain without checking geography/user-agent patterns first.

This is adequate for a static marketing site with no user-facing logic to break silently, and is explicitly not sufficient for the product platform.

## Target: platform observability

### Instrumentation: OpenTelemetry, from the first service

Every service (NestJS backend, and Next.js server-side where applicable) emits traces, metrics, and logs in OpenTelemetry format from its initial implementation — not retrofitted after an incident makes the gap obvious. Specifically:

- **Traces**: every incoming API request gets a trace spanning the full request lifecycle — API layer → application service → repository/external calls. Cross-module calls within the modular monolith (`architecture.md`) stay in the same trace, so a slow request is diagnosable without guessing which module was the bottleneck.
- **Metrics**: request rate/latency/error-rate per endpoint (the RED method) at minimum, plus domain-specific metrics that matter for the product's actual health signal — see "What to actually watch," below.
- **Logs**: structured (JSON), correlated to the trace ID of the request that produced them (`coding-standards.md`), shipped to Loki.

### Storage and visualization

- **Prometheus** stores metrics, scraped from each service's `/metrics` endpoint.
- **Loki** aggregates logs, queried and cross-referenced with traces/metrics in **Grafana** dashboards.
- **Grafana** is the single pane of glass — no separate ad hoc dashboarding tool per service.

### What to actually watch (beyond generic infra health)

Given the product's actual value mechanism (`vision.md`), infra-level health metrics (CPU, request latency) are necessary but not sufficient. Track, as first-class metrics from day one:

- `Verification` throughput and failure rate by `method` (`domain-model.md`) — a spike in failed verifications is a product-health signal, not just an error count.
- `TrustScore` recomputation lag (time between the triggering event — e.g. a `Relationship` reaching `cooperation_completed` — and the score update landing) — this is the literal latency of the network-effect loop the product is built around; if it's slow, the product's core promise is slow.
- `Relationship` lifecycle funnel (`pending` → `accepted` → `cooperation_*` → `completed`, and where it stalls) — this is a product metric, but it belongs in the same observability stack as infra metrics because a sudden change in this funnel is as urgent a signal as a latency spike.

Define alerting thresholds for these alongside the standard infra alerts (error rate, latency, queue depth) when the first version of each exists — don't treat them as "analytics to look at later."

### Alerting

Alerts route from Grafana (Prometheus/Loki-backed alert rules) to wherever the team actually monitors (decide the channel when the platform's on-call process is defined — see `development-workflow.md`). Alert on symptoms that need a human response, not on every metric that moves — alert fatigue is a real failure mode, avoid it by being deliberate about what pages someone versus what's dashboard-only.

## Product analytics

- **PostHog or Matomo** — decision not yet made (`technology-stack.md`). Whichever is chosen, it replaces Umami for the *product* (authenticated app usage), not necessarily for the *marketing site*, which can reasonably keep a simpler tool. Don't conflate "which tool answers 'is our infra healthy'" (OTel/Grafana stack, above) with "which tool answers 'are users doing the things that make the network valuable'" (product analytics) — they're different questions with different audiences, even though both matter.
- Whatever product-analytics tool is chosen, it must be self-hostable or EU-hosted per `product-principles.md` principle 7 and `security.md` — this is a real constraint on the PostHog-vs-Matomo decision, not a tie-breaker after the fact.
