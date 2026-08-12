---
date: 2026-08-12T14:15:10.908556+00:00
git_commit: ""
branch: ""
topic: "App portfolio site at apps.peterkurzok.de"
tags: [plan, astro, cloudflare, portfolio-site]
status: ready
---

# PLAN: App portfolio site at apps.peterkurzok.de

Build a small, custom-designed portfolio site that lists and promotes Peter's apps (PlayTales, PhotoMemo+, TVGraphs), his conference talks, and a contact sheet — similar in content to https://strasser.app but with a distinct, icon-driven design. The site gets its own public git repository and is built + hosted on Cloudflare, served at apps.peterkurzok.de.

## Acceptance Criteria

- apps.peterkurzok.de shows, in order:
  - **Hero**: name, tagline ("I build apps for Apple platforms" or similar), portrait photo (square source, rendered round-masked), contact icon row
  - **Three app rows**, each tinted with its icon's accent color, showing icon + title + tagline + link:
    - PlayTales → https://play-tales.app (the English site of the app also sold as Hörspieler in German; the English portfolio links the English brand)
    - PhotoMemo+ → https://photo-memo.peterkurzok.de
    - TVGraphs → "Coming soon" badge, no link
  - **Talks section** with three cards (one card per talk × event), ordered upcoming first, then past talks reverse-chronologically:
    - *Mobile Security Fundamentals: Build Apps That Fight Back* — iOSDevUK 2026 (Sept 7–10, Aberystwyth, https://www.iosdevuk.com/), upcoming, image from `~/Downloads/PeterKurzok.heic` (iOSDevUK speaker card)
    - *Mobile Security Fundamentals: Build Apps That Fight Back* — swiftCon 2026 (Oct 7–9, Berlin, https://www.nextappcon.com/swiftcon), upcoming, image from `~/Downloads/speaker-card-landscape-1200x628.png` (swiftCon speaker card)
    - *DeviceCheck – Securing your App's Communication* — DoiOS 2025 (https://www.do-ios.com/2025/#speakers), link to https://www.youtube.com/watch?v=iucdTgH0Jwo, image: YouTube thumbnail of that video
  - **Contact sheet**: apps@peterkurzok.de, GitHub (https://github.com/pkurzok), Mastodon (https://kind.social/@filmaniac), Instagram (https://instagram.com/filmaniac)
  - **Footer**: links to /imprint and /privacy pages
- All content is English
- Apps, talks, and contact channels live in data files — adding an app or flipping TVGraphs to "released with link" is a data-only edit
- Site is fully static (no client JS required), responsive, supports light + dark mode
- Public repo at github.com/pkurzok/app-site; pushing to `main` auto-deploys via Cloudflare git-integrated build

## Technical Key Decisions and Tradeoffs

1. **Astro with a fully custom design (no theme):**
   - Why: A pre-made theme fights the distinctive icon-driven design the user wants; Astro gives component-based templating, content as data files, zero shipped JS, and first-class Cloudflare support.
   - Impact: All layout/CSS is written from scratch; slightly more design work than a theme, full control in return.
2. **Tinted app rows design:** Each app gets a full-width section subtly tinted with an accent color drawn from its icon.
   - Why: Puts icons + titles front and center (user's explicit wish), scales when apps are added, clearly distinct from strasser.app's card grid.
   - Impact: Accent colors are stored per app in the data file (hand-picked from the icons, not computed at build time).
3. **English only:** App detail pages keep their own market language; the audio-play app is listed under its English brand PlayTales (play-tales.app) rather than Hörspieler (hoerspieler.app) — same app, English site.
   - Impact: Single set of copy; no i18n machinery.
4. **TVGraphs as "Coming soon", no link:**
   - Impact: `link: null` + `badge: "Coming soon"` in the data file; swapped at launch with a one-line edit.
5. **Own /imprint and /privacy pages**, content adapted from the existing PhotoMemo+ pages (https://photo-memo.peterkurzok.de).
   - Why: German law (§5 DDG) requires an Impressum; the site is static with no tracking/cookies, so the privacy page is short.
   - Impact: Two extra Astro pages; content fetched and adapted during implementation.
6. **Cloudflare Workers static assets + git-integrated builds** (not legacy Pages):
   - Why: Workers is Cloudflare's current recommended path for static sites; git integration deploys on push.
   - Impact: `wrangler.jsonc` with an `assets` block pointing at Astro's `dist/`; repo connected + custom domain bound via the Cloudflare dashboard by the user (guided steps provided).

## Current State

- `~/ws-privat/app-site/` is empty (no git repo yet).
- Reference site strasser.app: header, app cards (banner + tagline + "Learn more"), talks/podcasts, contact, German Impressum — the user wants the same content shape, different design.
- Existing app pages (linked, out of scope): play-tales.app (English site of the app also branded Hörspieler/hoerspieler.app in German), photo-memo.peterkurzok.de (EN/DE).
- TVGraphs is an unreleased universal SwiftUI app (`~/ws-workshop/TVGraphs/`, see its README): browse popular TMDB TV series, search, per-season episode-rating charts, favourites. Icon available at `~/ws-workshop/TVGraphs/TVGraphs/Assets.xcassets/AppIcon.appiconset/AppIcon-1024.png`.

## Desired End State

```
github.com/pkurzok/app-site  ──push──▶  Cloudflare git build  ──▶  apps.peterkurzok.de
app-site/
├── package.json / astro.config.mjs / wrangler.jsonc
├── src/
│   ├── data/
│   │   ├── apps.json          # name, tagline, description, icon, accent, link|null, badge?
│   │   ├── talks.json         # flat entries: title, event, date, location, eventLink, videoLink?, image
│   │   └── contact.json       # email, github, mastodon, instagram
│   ├── components/            # Hero, AppRow, TalkCard, ContactSheet, Footer …
│   ├── layouts/Base.astro     # <head>, global CSS, OG/meta tags
│   └── pages/
│       ├── index.astro        # hero → app rows → talks → contact
│       ├── imprint.astro
│       └── privacy.astro
├── public/
│   ├── icons/    playtales.png, photomemo.png, tvgraphs.png (1024px sources → optimized)
│   ├── images/   portrait.jpg, talk-iosdevuk.jpg, talk-swiftcon.png, talk-doios.jpg
│   └── favicon.svg / og-image
└── docs/agents/plans/         # this plan
```

Page layout (agreed mockup):

```
┌────────────────────────────────────┐
│         Peter Kurzok               │
│   [portrait]  I build apps for     │
│               Apple platforms      │
│      [✉ gh mastodon ig icons]      │
├────────────────────────────────────┤
│ ▒▒▒ PlayTales accent tint ▒▒▒▒▒▒▒ │
│  ╭────╮  PlayTales                 │
│  │icon│  Audio play player for kids│
│  ╰────╯  → play-tales.app          │
├────────────────────────────────────┤
│ ▒▒▒ PhotoMemo+ accent tint ▒▒▒▒▒▒ │
│  ╭────╮  PhotoMemo+ …              │
├────────────────────────────────────┤
│ ▒▒▒ TVGraphs accent tint ▒▒▒▒▒▒▒▒ │
│  ╭────╮  TVGraphs …  [Coming soon] │
├────────────────────────────────────┤
│  Talks                             │
│  [card] Mobile Security Fund. …    │
│         iOSDevUK 2026  (upcoming)  │
│  [card] Mobile Security Fund. …    │
│         swiftCon 2026  (upcoming)  │
│  [card] DeviceCheck … DoiOS 2025   │
│         ▶ Watch on YouTube         │
├────────────────────────────────────┤
│  Contact: ✉ apps@peterkurzok.de    │
│  GitHub · Mastodon · Instagram     │
├────────────────────────────────────┤
│  © Peter Kurzok · Imprint · Privacy│
└────────────────────────────────────┘
```

## Abstractions and Code Reuse

Greenfield project — no existing code to reuse. New abstractions:

- `src/data/*.json` — single source of truth for all listable content; components never hard-code app/talk facts
- `src/components/AppRow.astro` — renders one app from data (icon, title, tagline, accent tint, link or badge)
- `src/components/TalkCard.astro` — renders one talk-at-an-event card (a talk given at N events appears as N cards; "upcoming" derived from event date vs. build date — accepted limitation: the badge only updates on the next push/rebuild, so after Sept/Oct 2026 a commit is needed to drop it)
- `src/layouts/Base.astro` — shared head/meta/OG tags + global CSS (custom properties for light/dark)

Asset sources (copied + converted during implementation, originals untouched):

- PlayTales icon: from play-tales.app (or App Store listing) — highest-res available; if only low-res is obtainable, ask the user for the original 1024px icon asset
- PhotoMemo+ icon: from photo-memo.peterkurzok.de (or App Store listing) — same fallback
- Note: the conference speaker-card images (swiftCon/iOSDevUK branding) and the YouTube thumbnail are third-party artwork committed to a public repo — created for/from promoting these talks, assumed OK
- TVGraphs icon: `~/ws-workshop/TVGraphs/TVGraphs/Assets.xcassets/AppIcon.appiconset/AppIcon-1024.png`
- Portrait: `~/Downloads/_DSC7117_Square.jpeg` (1882×1882) → resized/optimized web JPEG/WebP
- iOSDevUK talk image: `~/Downloads/PeterKurzok.heic` (1280×720, iOSDevUK speaker card) → converted via `sips` to `talk-iosdevuk.jpg`
- swiftCon talk image: `~/Downloads/speaker-card-landscape-1200x628.png` → `talk-swiftcon.png` (landscape fits card layout better than the square variant)
- DoiOS talk image: YouTube thumbnail of the talk video, downloaded once and committed (`https://img.youtube.com/vi/iucdTgH0Jwo/maxresdefault.jpg` → `talk-doios.jpg`; fall back to `hqdefault.jpg` if maxres doesn't exist)

## Logging & Observability

None — static site. Build failures surface in Cloudflare's build logs; no runtime logging.

## Implementation

### Phase 1: Scaffold, assets, and data

Dependencies: None

Astro project with real content wired through data files; builds locally.

**Tasks**:
- [x] `git init` in `~/ws-privat/app-site`; scaffold Astro (`npm create astro@latest` minimal template, no example content); add `.gitignore` (node_modules, dist, .astro)
- [x] Collect and convert assets into `public/`:
  - [x] Download PlayTales + PhotoMemo+ icons from their sites/App Store listings
  - [x] Copy TVGraphs `AppIcon-1024.png`
  - [x] `sips` convert `PeterKurzok.heic` → `talk-iosdevuk.jpg`; copy swiftCon landscape card → `talk-swiftcon.png`; download YouTube thumbnail `img.youtube.com/vi/iucdTgH0Jwo/maxresdefault.jpg` → `talk-doios.jpg`; resize/optimize portrait `_DSC7117_Square.jpeg` → ~800px web image
- [x] Create `src/data/apps.json` with all three apps (name, tagline, short description, icon path, accent color hand-picked from each icon, link, `badge: "Coming soon"` + `link: null` for TVGraphs). TVGraphs tagline/description derived from its README (episode-rating charts for TV series, iPhone/iPad/Mac)
- [x] Create `src/data/talks.json` with three flat entries: Mobile Security Fundamentals @ iOSDevUK 2026 (Sept 7–10, Aberystwyth, eventLink iosdevuk.com, image talk-iosdevuk.jpg), Mobile Security Fundamentals @ swiftCon 2026 (Oct 7–9, Berlin, eventLink nextappcon.com/swiftcon, image talk-swiftcon.png), DeviceCheck @ DoiOS 2025 (eventLink do-ios.com/2025/#speakers, videoLink youtube.com/watch?v=iucdTgH0Jwo, image talk-doios.jpg)
- [x] Create `src/data/contact.json` (email apps@peterkurzok.de, GitHub, Mastodon, Instagram URLs)
- [x] Set `site: "https://apps.peterkurzok.de"` in `astro.config.mjs` (needed for absolute OG URLs later)
- [x] Create `src/pages/index.astro` rendering all data (unstyled or minimally styled is fine in this phase)
- [x] Write `README.md` (what the site is, how to develop/build/deploy)
- [x] Commit

**Automated Verification**:
- [x] `npm run build` succeeds
- [x] Built `dist/index.html` contains "PlayTales", "PhotoMemo+", "TVGraphs", "Coming soon", "DeviceCheck", "Mobile Security Fundamentals", and "apps@peterkurzok.de" (grep)
- [x] All referenced asset files exist in `dist/` — check with a small script that extracts `src`/`href` asset paths from `dist/**/*.html` and stats the files (Astro does NOT warn about dangling `public/` references)

### Phase 2: Design implementation

Dependencies: Phase 1

The agreed "tinted app rows" design: hero with portrait, accent-tinted app sections, talks cards, contact sheet, footer. Responsive, light + dark mode.

**Tasks**:
- [x] `src/layouts/Base.astro`: global CSS with custom properties (light palette on `:root`, dark overrides via `prefers-color-scheme`), system font stack or a single self-hosted font, meta/OG tags (title, description, OG image)
- [x] `src/components/Hero.astro`: name, tagline, round-masked portrait, contact icon row (inline SVG icons for mail/GitHub/Mastodon/Instagram)
- [x] `src/components/AppRow.astro`: full-width section, background tinted from the app's accent color (subtle, works in both color schemes), large rounded icon (Apple-style squircle radius), title, tagline, description, arrow link — or "Coming soon" badge when `link` is null
- [x] `src/components/TalkCard.astro`: talk image, title, event name + date + location (linked to the event page), "Upcoming" badge when the event date is in the future, "Watch on YouTube" link when `videoLink` is present
- [x] `src/components/ContactSheet.astro` + `Footer.astro` (copyright, Imprint/Privacy links)
- [x] Responsive layout: single column on mobile, comfortable max-width on desktop; icons/images with explicit dimensions (no layout shift)
- [x] Favicon (SVG) and OG image
- [x] Commit

**Automated Verification**:
- [x] `npm run build` succeeds
- [x] Zero-JS check passes: `! grep -rq '<script' dist/` (note: plain grep exits 1 on "no match" = success, hence the `!`)
- [x] Asset-existence script from Phase 1 still passes

**Manual Verification**:
- [ ] `npm run dev` — page matches the agreed mockup: hero with portrait, three tinted app rows with icons, talks, contact, footer
- [ ] Check mobile width (responsive), light and dark mode, and that all external links open the right targets
- [ ] All visible copy is English

### Phase 3: Legal pages and 404

Dependencies: Phase 2 (uses `Base.astro` and the footer)

**Tasks**:
- [x] Fetch imprint + privacy content from photo-memo.peterkurzok.de, adapt for this site (English, "static site, no tracking, no cookies, hosted on Cloudflare" privacy note; Cloudflare as hosting processor)
- [x] `src/pages/imprint.astro` and `src/pages/privacy.astro` using `Base.astro`
- [x] `src/pages/404.astro` — minimal styled 404 using `Base.astro`
- [x] Verify the footer's Imprint/Privacy links (created in Phase 2) point at the new pages
- [x] Commit

**Automated Verification**:
- [x] `npm run build` succeeds; `dist/imprint/index.html`, `dist/privacy/index.html`, and `dist/404.html` exist
- [x] `dist/index.html` links to both pages (grep for `imprint` and `privacy` hrefs)
- [x] Zero-JS check across all pages: `! grep -rq '<script' dist/`

### Phase 4: Repository and Cloudflare deployment

Dependencies: Phases 1–3

**Tasks**:
- [x] Confirm the peterkurzok.de zone is actually in the user's Cloudflare account (`dig NS peterkurzok.de` shows Cloudflare nameservers + user confirmation) — the photo-memo subdomain only proves it's served somewhere — **FAILED, see Implementation Notes**
- [x] Add `wrangler.jsonc`: `name: "app-site"` (must match the Worker name chosen in the dashboard import), NO `main` (assets-only Worker), `assets: { directory: "./dist", not_found_handling: "404-page" }`, compatibility date
- [x] Add `wrangler` to `devDependencies` (pins the version Workers Builds uses via `npx wrangler deploy`). Do NOT add a local `deploy` npm script — deploys go through git builds only, and a local `wrangler deploy` would clobber the git-deployed version
- [x] Create public repo `pkurzok/app-site` via `gh repo create`, push `main`
- [ ] Provide the user a short checklist for the dashboard (cannot be automated without account auth):
  - Workers & Pages → Create → Import repository `pkurzok/app-site`
  - Set the Worker name to `app-site` (must match `name` in `wrangler.jsonc`, or builds fail)
  - Build command: `npm run build`; deploy command: `npx wrangler deploy` (reads `wrangler.jsonc`)
  - Custom domain `apps.peterkurzok.de` **deferred** — the zone is not on Cloudflare (see Implementation Notes); the site serves on `app-site.<subdomain>.workers.dev` until that is resolved
- [ ] After the user connects: verify live site

**Automated Verification**:
- [x] `npm run build && npx wrangler deploy --dry-run` validates the config (dry-run needs `dist/` to exist)
- [x] `gh repo view pkurzok/app-site` shows the pushed repo
- [ ] After connection: `curl -sI https://apps.peterkurzok.de` returns 200 (deferred with the custom domain — check the `workers.dev` URL instead)

**Manual Verification**:
- [ ] User completes the Cloudflare dashboard connection + custom domain binding
- [ ] Open https://apps.peterkurzok.de and confirm the deployed site renders correctly

## Implementation Notes

During implementation, document user feedback, problems, and decisions here.

### 2026-08-12 — peterkurzok.de is not a Cloudflare zone (Phase 4 blocker)

`dig NS peterkurzok.de` returns `ns.inwx.de` … `ns5.inwx.de` — the domain's DNS is
authoritative at INWX, not Cloudflare. `photo-memo.peterkurzok.de` turned out to be a
CNAME to `pkurzok.github.io` (GitHub Pages, `server: GitHub.com`), which is why the
subdomain existing proved nothing about Cloudflare.

Cloudflare docs confirm a Workers **Custom Domain** requires "an active Cloudflare zone"
(<https://developers.cloudflare.com/workers/configuration/routing/custom-domains/>); the
same holds for Pages custom domains. The CNAME/partial zone setup that would avoid moving
nameservers is Business/Enterprise only. So `apps.peterkurzok.de` cannot be bound to the
Worker until the zone is on Cloudflare — this needs a user decision (move nameservers to
Cloudflare vs. host elsewhere).

Everything else in Phase 4 that does not depend on that decision is done: `wrangler.jsonc`,
the pinned `wrangler` devDependency, and `npx wrangler deploy --dry-run` validating clean.

### 2026-08-12 — data corrections found while implementing

- The plan wrote "DoiOS 2025" as a German event; it is **Do iOS 2025**, held in
  **Amsterdam** (conference 12–13 November 2025). `talks.json` uses the corrected values.
- Accent colors were sampled from the icons themselves: PlayTales `#FF9500`, PhotoMemo+
  `#617B92` (the slate coaster), TVGraphs `#8B5CF6` (the purple chart line).
- The Do iOS YouTube `maxresdefault.jpg` thumbnail is a mid-talk frame showing a slide
  rather than the speaker — sharp, but arguably off-topic as a card image. Flagged for the
  user; the low-res `1.jpg` frame shows the title slide but is only 120×90.

## References

- Design reference (content shape): https://strasser.app
- App pages: https://play-tales.app (German sibling: https://hoerspieler.app), https://photo-memo.peterkurzok.de
- TVGraphs: `~/ws-workshop/TVGraphs/README.md`
- Talks: https://www.youtube.com/watch?v=iucdTgH0Jwo + https://www.do-ios.com/2025/#speakers (DoiOS 2025), https://www.iosdevuk.com/ (Sept 7–10 2026), https://www.nextappcon.com/swiftcon (Oct 7–9 2026, CityCube Berlin)
- Cloudflare Workers static assets: https://developers.cloudflare.com/workers/static-assets/
