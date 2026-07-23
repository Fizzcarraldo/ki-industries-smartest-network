# Deployment

> Status: **Current section is the real, working process for this repository today. Target section is mandated infrastructure for the product platform — entirely separate from, and not a replacement path for, the current static site's deployment.**

## Current: static site on GitHub Pages

This is the actual, verified deployment process for `index.html`/`vision.html` as of this writing.

### How it works

- GitHub Pages serves this repository's `main` branch, root directory, directly — no build step, no GitHub Actions workflow file needed for the deploy itself (Pages' own "pages build and deployment" system handles it automatically on every push to `main`).
- **`.nojekyll`** (empty file at repo root) is required and present — without it, GitHub Pages runs the default Jekyll processor, which silently excludes any directory starting with `_` (this broke `_ds/` — the design system — once already; see `git log`, commit `61d3dda`). Do not remove this file.
- **`CNAME`** (repo root, contains `smartest-network-demo.florian-kratzer.com`) points the custom domain at GitHub Pages. DNS is managed externally (currently at Strato, an A-record pointing at GitHub Pages' IP) — DNS changes are made outside this repository and are not automatable from here.
- HTTPS is provisioned automatically by GitHub once DNS resolves correctly (Let's Encrypt via GitHub's Pages infrastructure) — this can take from minutes up to ~24h after a DNS change and cannot be sped up from this side.

### How to deploy a change

```
git add <files>
git commit -m "..."
git push
```

That's the entire deploy. Verify it landed with:

```
gh api repos/Fizzcarraldo/ki-industries-smartest-network/actions/runs --jq '.workflow_runs[0] | {head_sha, status, conclusion}'
```

(Not `gh api .../pages/builds/latest` — that endpoint has been observed to return stale data for Actions-based Pages deployments; the Actions run status above is the reliable signal.)

### Verification, not assumption

Every change to `index.html`, `vision.html`, or `_ds/.../styles.css` in this repository's history has been verified against the **live** deployed site after push, not just locally — using Playwright (headless Chromium/Firefox/WebKit) to load the actual production URL and check for console errors, layout, and (where relevant) Lighthouse accessibility scores. Do not consider a deploy "done" on the basis of a local `python3 -m http.server` test alone; the GitHub Pages CDN, HTTPS termination, and real mobile viewports have each independently surfaced bugs that a local static server didn't.

### What this process does not have

No staging environment, no CI checks gating the merge, no automated tests, no rollback mechanism beyond `git revert`. This is an accepted tradeoff for a single-maintainer static demo site (see `security.md`) and must **not** be the model copied for the product platform below.

## Target: product platform on Hetzner Cloud

See [ADR-0005](./adr/0005-hetzner-cloud-infrastructure.md) for why Hetzner specifically.

### Provisioning: OpenTofu

Infrastructure (compute instances, networking, load balancer, managed Postgres if used, Object Storage buckets) is defined as OpenTofu configuration, version-controlled in the product repository (not this one, or in a clearly separated `infra/` root if the platform ends up in this repository — decide and record when the platform repository is created). No infrastructure is provisioned by hand through the Hetzner console — the console is for read-only inspection and emergency intervention only, with any manual change reconciled back into OpenTofu state immediately after.

### Configuration: Ansible

Once OpenTofu provisions hosts, Ansible playbooks bring them to a defined configuration state (runtime dependencies, Docker installation, log shipping agents for Loki, etc.). Playbooks are idempotent and re-runnable — a playbook run against an already-configured host should be a no-op, not an error or a duplicate change.

### Runtime: Docker

Backend (NestJS) and frontend (Next.js) each ship as a Docker image, built in CI, tagged with the commit SHA, and deployed by reference to that tag — never `latest` in production. Local development also runs against Docker (Postgres, Valkey) via a `docker-compose.yml` at the product repository root, so "works on my machine" means the same Postgres/Valkey versions as production.

### Environments

At minimum: `production` and `staging`, both on Hetzner, provisioned identically via the same OpenTofu configuration with environment-specific variables — not hand-maintained divergent setups. A change is deployed to `staging` and verified before `production`, unlike the current static site's direct-to-production model.

### Deployment pipeline (target shape, refine when the platform repository exists)

1. PR merged to `main` → CI builds and tests (see `testing.md`) → Docker images built and pushed to a registry.
2. Automatic deploy to `staging`.
3. Manual promotion (not automatic) from `staging` to `production`, after verification.
4. Database migrations (Drizzle) run as an explicit, reviewed step before the new application version receives traffic — never an implicit side effect of application boot.

### What must not happen

- No direct `git push`-to-deploy model for the product platform, even though that's exactly what works fine for the current static site. The moment there's a database and real user data, an unreviewed direct-to-production deploy is a different risk category entirely.
- No infrastructure change made by hand that isn't reflected in OpenTofu state within the same working session.
