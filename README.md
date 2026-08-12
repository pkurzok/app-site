# app-site

Portfolio site for Peter Kurzok's apps and conference talks — live at
[apps.peterkurzok.de](https://apps.peterkurzok.de).

Static [Astro](https://astro.build) site, no client-side JavaScript, built and hosted
on Cloudflare Pages.

## Development

```bash
npm install
npm run dev      # local dev server on http://localhost:4321
npm run build    # static build into dist/
npm run preview  # serve the built site
npm run check    # verify every asset referenced in dist/ actually exists
```

## Content

All listable content lives in `src/data/` — adding an app, a talk, or a contact
channel is a data-only edit, no component changes needed.

| File | Contents |
| --- | --- |
| `src/data/apps.json` | name, tagline, description, icon path, accent color, link (or `null` + `badge` for unreleased apps) |
| `src/data/talks.json` | one entry per talk-at-an-event: title, event, date, location, event link, optional video link, image |
| `src/data/contact.json` | name, tagline, portrait, contact channels |

Images live in `public/icons/` (app icons, 1024×1024) and `public/images/`
(portrait, talk cards).

The "Upcoming" badge on a talk is derived from its `date` versus the **build**
date, so it only updates on the next deploy.

## Deployment

Pushing to `main` triggers a Cloudflare Pages build:

| Setting | Value |
| --- | --- |
| Build command | `npm run build` |
| Build output directory | `dist` |

There is deliberately no local deploy script — deploying from a workstation would
overwrite the git-deployed version.

### Custom domain

`peterkurzok.de` is not a Cloudflare zone; its DNS is authoritative at INWX. Pages
supports that via its external-DNS flow, and the order matters:

1. In the Pages project, **Custom domains → Set up a domain →** `apps.peterkurzok.de`.
2. Only then add the `CNAME apps → app-site.pages.dev` record at INWX.

Adding the CNAME first, without registering the hostname in the dashboard, yields a
522. This is the same arrangement `playtales.peterkurzok.de` already uses. A Cloudflare
**Workers** custom domain would not work here — that requires the zone to live in the
Cloudflare account.
