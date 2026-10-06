---
date: 2026-10-06T10:05:53.347154+00:00
git_commit: 8b4a3933fec922632ebc5fa1f21c617e714ce9e7
branch: main
topic: "Safari side bar follows the app row tint on iPhone Duo"
tags: [plan, css, safari, iphone-duo, Base.astro, AppRow.astro, index.astro]
status: done
---

# PLAN: Safari side bar follows the app row tint on iPhone Duo

On a closed iPhone Duo, Safari puts its controls into a vertical bar to the right of
the page. The tinted app rows stop hard at that bar, which stays white. The goal is
that the bar takes the tint of the app row currently in view, so the row background
appears to continue behind the bar — without adding client-side JavaScript.

## Acceptance Criteria

- iPhone Duo (closed), Safari: the side bar shows the tint of the app row that covers
  the vertical centre of the viewport.
- When no app row covers the centre (hero, talks, contact, footer), the bar shows `--bg`.
- Works in light and dark mode with the matching tints.
- Hero, talks, contact and footer look unchanged (no tint shining through).
- Imprint, privacy and 404 pages look unchanged and get no `edge-` CSS.
- The site still ships no client-side JavaScript.
- A new entry in `src/data/apps.json` gets the behaviour without component changes.
- Browsers without scroll-driven animations render the site as before.

## Technical Key Decisions and Tradeoffs

1. **Mechanism:** The `body` background colour drives the bar; a new `.page` wrapper
   with `background: var(--bg)` covers the body.
   - Why: The bar lies outside the viewport (viewport 382 px of 466 px screen width,
     all `safe-area-inset-*` are 0, `viewport-fit=cover` has no effect). Safari paints
     it in one solid colour taken from the `body` background (not `html`). A fixed
     edge element only overrides it when it is visibly wide (10 px worked, 1 px,
     off-screen and `opacity: 0` did not).
   - Impact: `Base.astro` wraps `<slot />` and `<Footer />` in `<div class="page">`.
2. **CSS instead of JS:** Each app row exposes a named view timeline with
   `view-timeline-inset: 50%`; the body gets one animation per app that is only
   active while that row covers the viewport centre line.
   - Why: Keeps the README promise "no client-side JavaScript". Verified in the Duo
     simulator: the bar follows scroll-driven animations on the body.
   - Impact: `index.astro` generates keyframes and the animation lists from `apps.json`.
3. **Hard switch, no blending:** The colour jumps at the row boundary.
   - Why: Matches the chosen behaviour. A CSS transition on the body colour would be
     followed by the bar, but it cannot be combined with the scroll-driven animation
     that sets the colour here.
   - Impact: Keyframes are a single constant colour (`from, to`).
4. **No device check:** Applies in every browser.
   - Why: On a regular iPhone the page already runs under Safari's bars; the body
     colour is only visible in the status bar area at scroll position 0, where the hero
     is centred and the colour is `--bg`. Verified in a regular iPhone simulator.
   - Impact: No screen/viewport width heuristic.
5. **Tint formula is duplicated:** The generated keyframes repeat
   `color-mix(in srgb, <accent> var(--tint-strength), var(--bg))` from `AppRow.astro`.
   - Why: Keyframes on the body cannot read a custom property set on a row.
   - Impact: A comment at both places points to the other.

## Current State

```
screen 466px
┌──────────────────────────────┬───────┐
│ viewport 382px               │ Safari│
│                              │ bar   │
│ Hero        (no own bg)      │       │
│──────────────────────────────│ one   │
│ AppRow PlayTales  (orange)   │ colour│
│──────────────────────────────│  =    │
│ AppRow PhotoMemo+ (blue-grey)│ body  │
│──────────────────────────────│ --bg  │
│ Talks / Contact (no own bg)  │       │
└──────────────────────────────┴───────┘
```

- `src/layouts/Base.astro:41-44` — `<body>` contains `<slot />` and `<Footer />` directly.
- `src/layouts/Base.astro:101` — `body { background: var(--bg) }`.
- `src/layouts/Base.astro:211-216` — reduced-motion rule sets
  `animation-duration: 0.01ms !important` on `*`.
- `src/components/AppRow.astro:19` — row sets `--accent` inline.
- `src/components/AppRow.astro:66` — row tint
  `color-mix(in srgb, var(--accent) var(--tint-strength), var(--bg))`.
- `src/pages/index.astro:32` — renders one `AppRow` per entry of `apps.json`.
- `README.md:6` — "no client-side JavaScript".

## Desired End State

```
<body>          background-color: var(--bg), overridden by the active row animation
  <div.page>    background: var(--bg) — hides the body colour behind all content
    Hero / Apps / Talks / Contact / Footer

┌───────────────────┬─────┐        ┌───────────────────┬─────┐
│ PlayTales (orange)│orang│ scroll │ PlayTales (orange)│blue │
│ ·····centre·······│     │   ⇣    │───────────────────│     │
│───────────────────│     │        │ ·····centre·······│     │
│ PhotoMemo+ (blue) │     │        │ PhotoMemo+ (blue) │     │
└───────────────────┴─────┘        └───────────────────┴─────┘
```

Generated CSS on the index page (shape, one entry per app):

```css
@keyframes edge-playtales { from, to {
  background-color: color-mix(in srgb, #FF9500 var(--tint-strength), var(--bg)); } }
/* … */
@supports (animation-timeline: view()) {
  body {
    timeline-scope: --app-playtales, --app-photomemo /* … */;
    animation-name: edge-playtales, edge-photomemo /* … */;
    animation-timeline: --app-playtales, --app-photomemo /* … */;
    animation-range: cover;
    animation-timing-function: linear;
    animation-fill-mode: none;
  }
}
```

With `view-timeline-inset: 50%` the row's view progress visibility range collapses to
the viewport centre line, so `cover 0% – 100%` is exactly "row covers the centre".
`fill-mode: none` makes the body fall back to `--bg` outside that range.

## Abstractions and Code Reuse

No new components. Existing `--accent`, `--tint-strength` and `--bg` tokens are reused.

- `src/layouts`
  - `Base.astro` - add `<slot name="head" />` at the end of `<head>`; wrap default
    slot and `<Footer />` in `<div class="page">`; add `.page { background: var(--bg) }`
- `src/components`
  - `AppRow.astro` - inline `view-timeline-name: --app-<id>`; scoped
    `view-timeline-axis: block; view-timeline-inset: 50%` on `.app-row`
- `src/pages`
  - `index.astro` - build `edgeCss` string from `apps` and emit it via
    `<style slot="head" is:inline set:html={edgeCss} />`
- `README.md` - short section on the side bar tint

## Logging & Observability

None — static CSS only.

## Implementation

Dependencies: None

Single vertical slice: wrapper, row timelines and generated body animations only
produce a visible result together.

**Tasks**:
- [x] `src/layouts/Base.astro`: add `<slot name="head" />` as last child of `<head>`.
- [x] `src/layouts/Base.astro`: wrap `<slot />` and `<Footer />` in `<div class="page">`
  and add a global rule with a one-line comment on why it exists:
  ```css
  /* Hides the body colour, which only tints Safari's side bar (see index.astro) */
  .page { background: var(--bg); }
  ```
- [x] `src/components/AppRow.astro`: extend the inline style to
  `--accent: ${app.accent}; view-timeline-name: --app-${app.id}` and add
  `view-timeline-axis: block; view-timeline-inset: 50%;` to `.app-row`.
  Add a comment at the tint line that `index.astro` repeats the formula.
- [x] `src/pages/index.astro`: build `edgeCss` in the frontmatter from `apps`
  (one `@keyframes edge-<id>` per app plus the `@supports` body block shown above)
  and emit it with `<style slot="head" is:inline set:html={edgeCss} />` inside `<Base>`.
- [x] `src/pages/index.astro`: in the frontmatter, throw a build error when an app
  `id` does not match `/^[a-z0-9-]+$/`, because the id becomes part of CSS names.
- [x] `README.md`: add a short "Safari side bar tint" paragraph under Content: the
  bar colour comes from the body background, driven by per-row view timelines
  generated from `apps.json`; still no client-side JavaScript. Note in the
  `apps.json` table row that `id` must match `[a-z0-9-]+`.
- [x] If the reduced-motion check below fails: exempt the body in the
  `prefers-reduced-motion` block of `Base.astro` (`body { animation-duration: auto !important; }`).
  Not needed — the check passed.

**Automated Verification**:
- [x] `npm run build` succeeds.
- [x] `npm run check` succeeds.
- [x] `grep -o '@keyframes edge-' dist/index.html | wc -l` equals
  `node -p "require('./src/data/apps.json').length"`.
- [x] `! grep -q '<script' dist/index.html` succeeds.
- [x] `! grep -q 'edge-' dist/imprint/index.html dist/privacy/index.html dist/404.html`
  succeeds.
- [x] Start the dev server with `astro dev --background` for the simulator checks
  below and stop it with `astro dev stop` afterwards.
- [x] Duo simulator (`Probe iPhone Duo`, closed), page opened via
  `xcrun simctl openurl <udid> http://localhost:4321/`, screenshots via
  `xcrun simctl io <udid> screenshot`: at scroll position 0 the bar pixel
  (x = width − 40 px, y = 60 % height) equals the page `--bg`.
- [x] Same setup, scrolled with `rocketsim interact swipe` so that an app row covers
  the centre: the bar pixel equals the pixel at (x = 20 px, y = 50 % height). Repeat
  for a second row with a different accent.
- [x] Same two checks after `xcrun simctl ui <udid> appearance dark`; reset to `light`
  afterwards.
- [x] Scrolled to the talks section: the bar pixel equals `--bg` and the area left of
  the bar shows no row tint outside app rows.

**Manual Verification**:
- [x] On the closed iPhone Duo simulator, scroll through the app list: the bar switches
  to each row's tint as the row passes the screen centre and returns to the page
  background above and below the list.
- [x] With Settings → Accessibility → Motion → Reduce Motion enabled, the bar still
  follows the rows.
- [x] On a regular iPhone simulator the page looks as before, including the status bar
  area at the top of the page.

## Implementation Notes

During implementation, document user feedback, problems, and decisions here.

- RocketSim's first calls after launch can take over a minute; `xcrun simctl openurl`
  and `xcrun simctl io screenshot` work as a fallback.
- Simulator checks were run against the production build (`astro preview`), 6 scroll
  positions × 5 runs in light and dark: bar colour equals the row at the viewport
  centre every time, and `--bg` at the top and in the talks section.
- Dev server caveat: after editing styles, Safari in the simulator kept showing stale
  CSS from `astro dev` for a while (first the `.page` rule, later the row inset were
  missing), which looked like real bugs. Verify against `astro preview` when in doubt.
- Reduce Motion pre-checked by toggling `ReduceMotionEnabled` via `simctl spawn defaults`
  (confirmed to reach Safari's media query): the bar still follows the rows, so the
  body exemption was not added.
- Regular iPhone simulator pre-checked: page looks as before at the top and scrolled.
- The dev server was left running for the manual checks.
- Edge case accepted: on a wide, short viewport an app row can cover the centre at
  scroll position 0, so the top overscroll area would show that row's tint.

## References

- Simulator probes from the planning session (2026-10-06, iOS 27.1, Probe iPhone Duo):
  `viewport-fit=cover` no effect; `body` background sampled; `html` background ignored;
  thin fixed strips ignored; CSS transitions and scroll-driven animations followed.
- `src/layouts/Base.astro`, `src/components/AppRow.astro`, `src/pages/index.astro`
- MDN: `view-timeline-inset`, `timeline-scope`, `animation-range`
