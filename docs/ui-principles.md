# UI Principles

> Status: **The design-system half of this document is current and real** — grounded directly in `_ds/industry-95c656ac-dba7-40f4-955b-adb820946ce8/` and `index.html`/`vision.html`, all present in this repo today. **The React-component half is target** — the current implementation renders this design system through the vendored `dc-runtime`, not through project-owned components (see `architecture.md`); when the Next.js frontend is built, port the tokens and component classes below faithfully rather than reinventing them.

## The existing design system: "Industry"

The real, working design system lives in `_ds/industry-95c656ac-dba7-40f4-955b-adb820946ce8/styles.css`, with its own readme at `_ds/.../readme.md`. Read that file — it is the source of truth for tokens and is kept in sync with itself by convention already. This section summarizes the parts every future contributor must know; it does not replace the readme.

### Visual identity

Industry is a **wireframe aesthetic**: steel-blue on a light technical ground, Barlow Condensed headings over Barlow body text, a modular grid, and cards/figures/buttons framed as blueprint objects — square-cornered, hairline-bordered, with "+" registration marks at the corners (`.blueprint` + four `<i class="corner tl/tr/bl/br">` children). Cards and figures stay transparent line drawings; the primary button is the one deliberately solid object on the page.

**Do:**
- Frame cards, figures, and primary CTAs as blueprint objects (`.blueprint` + corner marks).
- Keep the grid visible — equal cells, strong horizontal/vertical rhythm.
- Condense headings (Barlow Condensed, uppercase), keep body copy in Barlow.
- Take every color, spacing, radius, and shadow value from the CSS custom properties (`var(--color-*)`, `var(--space-*)`, `var(--radius-*)`, `var(--shadow-*)`) — never a hardcoded hex or px value.

**Don't:**
- Round cards, figures, or buttons, or give them a surface fill — they're line drawings (the solid `.btn-primary` is the one deliberate exception).
- Drop the registration marks from a framed element.
- Use thick icon strokes — the set is Lucide at `stroke-width: 1.5`.
- Add decorative color beyond the steel accent.

### Color

Tokens: `--color-bg`, `--color-text`, `--color-accent` (base brand color, `#5980a6`), plus generated tonal ramps `--color-accent-100` through `--color-accent-900` (and equivalent neutral/accent-2 ramps), in OKLCH on a shared lightness scale so the same numeric step means the same visual weight across ramps.

**Contrast rule, learned the hard way and now fixed in the codebase — do not regress it:** the base `--color-accent` (`#5980a6`) only reaches ~3.7:1 against `--color-bg`, below WCAG AA's 4.5:1 for normal text. **Use `--color-accent-700` for any accent-colored text** (links, `.tag-outline`, `.card-kicker`, `.btn-ghost`). Reserve the base `--color-accent` for non-text UI (icons, borders, the `.btn-primary` fill uses `--color-accent-700` too — see below) where the relaxed 3:1 threshold applies. This is already fixed throughout `_ds/.../styles.css` as of the current codebase — when adding a new component class, follow the same rule, don't reintroduce base-`--color-accent` text.

**CSS specificity gotcha, also already fixed, don't reintroduce it:** a bare `.nav a { color: inherit }` rule (class+type selector) has *higher* specificity than a single-class `.btn-primary { color: ... }` rule, and will silently override a nav-embedded button's text color. The fix in the current codebase is `.nav a:not(.btn)` — when writing new component-in-context CSS, check specificity against existing rules, don't assume "class selector" is always low-specificity-enough to be safely overridden.

### Type

`--font-heading` (Barlow Condensed, weight 600), `--font-body` (Barlow), loaded via Google Fonts `@import` at the top of `styles.css`. Density (0.85×) and radius (4px) are baked into the `--space-*`/`--radius-*` scales — use the variables, not raw numbers.

### Components

Use the existing classes — `.btn`/`.btn-primary`/`.btn-ghost`, `.tag`/`.tag-accent`/`.tag-outline`, `.card-kicker`, `.nav`, `.blueprint`, `.table`, `.dialog` — documented with usage examples in `_ds/.../readme.md`. Do not invent parallel classes for something the system already covers. If a genuinely new component is needed, add it to `_ds/.../styles.css` following the existing token-based pattern (see the `vh-*` scroll-reveal and network-diagram classes added for `vision.html` as a worked example of extending the system without breaking its conventions).

## Accessibility by default

Not aspirational — this is enforced and verified in this codebase:

- **Lighthouse accessibility score 100 is the bar**, verified for both existing pages as of the current codebase. Any UI change that regresses it is a bug, not a style nitpick.
- Every animated/revealing element has a `prefers-reduced-motion: reduce` fallback that shows the final state instantly, no exceptions (see the `.reveal`, `.vh-node-dot`, `.vh-hub-pulse` rules in `_ds/.../styles.css`).
- Every page has exactly one `<main>` landmark. (This was a real bug, found and fixed in `index.html` — don't reintroduce a page without one.)
- Decorative SVG/icon content is `aria-hidden="true"`; where a graphic conveys real information (the network diagram in `vision.html`), it ships with an `sr-only` text description of what it shows, not just a hidden empty graphic.
- Nested-scroll or otherwise non-obvious interactive regions get `tabindex`, a `role`, and an `aria-label` so keyboard users can reach and operate them (see the `vision.html` history for a worked, tested example — since removed in favor of a simpler static layout, but the pattern is the reference if nested scroll is ever reintroduced).
- Color contrast is checked with a real tool (Lighthouse or `axe`), not eyeballed — this codebase has twice had contrast regressions that looked fine visually but measured below 4.5:1.

## Responsive by default

- Mobile breakpoint in this codebase's convention: `@media (max-width: 880px)`. Follow it for consistency rather than introducing a second breakpoint value.
- **Verify actual behavior on mobile, not just narrow-desktop-window behavior.** This codebase has a documented history (see `git log` on `vision.html`) of CSS that looked right in isolation but broke under real mobile constraints — `position: sticky` interacting with CSS Grid row sizing differently than the spec suggests, `position: sticky` not pushing sibling content out of its way (a load-bearing lesson — read the commit messages on `vision.html`'s mobile-diagram history before attempting anything sticky/fixed on mobile again), nested-scroll containers needing measured (not guessed) header heights. Test with a real headless-browser tool (Playwright, as used throughout this project's development) across Chromium/Firefox/WebKit and at actual mobile viewport sizes before considering a responsive change done.
- Full-bleed/full-width treatments on mobile are preferred over shrinking content into a small floating badge — this was tried, explicitly rejected, and reverted in this codebase's history. Don't reintroduce a small floating diagram/widget pattern on mobile without checking that history first.

## Animation

- Animations support content, they don't decorate it. The scroll-driven network diagram in `vision.html` is the reference example: each animation step corresponds to a specific narrative beat (a new connection, a trust ring, a central hub appearing), not motion for its own sake.
- Prefer CSS transitions/animations over JavaScript-driven animation wherever possible. Where JS is required (scroll-position-driven state), use a single `requestAnimationFrame`-throttled handler, not multiple uncoordinated scroll listeners — see the `_initScrollDrivenState` pattern in `vision.html`.
- `transform-box: fill-box` is required on any SVG element animated with `transform` + `transform-origin` — browsers disagree on the default (Chromium/WebKit resolve `transform-origin: center` against the shape, Firefox against the whole SVG viewport by spec default). This is already applied throughout `_ds/.../styles.css`'s `.vh-*` rules; don't add a new animated SVG element without it.
- **Content-sensitivity check for generated/geometric graphics:** a diagram built from straight lines radiating from a center point with a consistent rotational bend is structurally a swastika, regardless of intent — this happened in this codebase's history and was caught and fixed (see `git log`). Any new geometric/generative graphic gets a visual sanity check for unintended symbolic resemblance before shipping, not just a "does it look cool" check.

## What to build when the Next.js frontend starts

Port `_ds/.../styles.css`'s tokens as-is (CSS custom properties work identically in Next.js — no conversion to a CSS-in-JS or Tailwind config needed, and doing so risks introducing subtle token drift). Rebuild the component classes as React components with the same class names and DOM shape where practical, so the design system readme continues to describe the real UI. Re-implement `vision.html`'s scroll-driven diagram as a proper component (a custom hook wrapping the same `requestAnimationFrame` scroll-tracking approach) rather than the current inline dc-runtime script — the *behavior* proved out in `vision.html` is right, the *mechanism* (inline script in a generated HTML file) is not what the target architecture uses long-term.
