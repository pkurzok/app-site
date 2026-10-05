---
date: 2026-10-05T15:14:04.770439+00:00
git_commit: a876e96001da12ac8dea439353e6ae5f1b2fc3e4
branch: main
topic: "Add Karacho to the app list"
tags: [plan, apps-json, karacho]
status: ready
---

# PLAN: Add Karacho to the app list

Add Karacho, Peter's GPS speedometer for iPhone, as the fifth entry of the "Apps" section on
apps.peterkurzok.de. Version 1.0 went to App Review on 2026-10-05 and is not in the App Store
yet; its own site, https://karacho.peterkurzok.de, is live and says "Coming soon".

## Acceptance Criteria

- apps.peterkurzok.de lists five apps under "Apps", Karacho last.
- The Karacho row shows its icon, the name "Karacho", the tagline "Your GPS speed in big amber
  digits" and the description given under *Desired End State*.
- The icon and the call to action "karacho.peterkurzok.de →" link to
  https://karacho.peterkurzok.de. No badge is shown.
- The row is tinted with `#FFB020` and is readable in light and in dark mode.
- `npm run build` and `npm run check` pass, and `dist/icons/karacho.png` exists.
- The four existing entries are unchanged.

## Technical Key Decisions and Tradeoffs

1. **Timing:** The entry goes live now, with a link instead of a "Coming soon" badge.
   - Why: The Karacho site is reachable and states the launch status itself.
   - Impact: Nothing on this site changes when the app is released. `AppRow.astro` shows a
     badge only for `link: null`, so `badge` stays `null`.
2. **Texts:** Short name "Karacho", the tagline of the Karacho site, a description that makes no
   claim of availability.
   - Why: Both sites read the same, and the text is true before and after release.
   - Impact: One new object in `src/data/apps.json`; no component changes.
3. **Accent and position:** `#FFB020`, appended as the last entry.
   - Why: It is the amber of the app's digits and of its site. The list is chronological, and
     three rows of other colours separate it from PlayTales' `#FF9500`.
   - Impact: The object is appended to the array.
4. **Icon:** Copy `~/ws-privat/Speedo/Screenshots/app-icon.png` to `public/icons/karacho.png`.
   - Why: It is the current 1024×1024 export, full-bleed with transparent rounded corners, the
     same shape as `public/icons/inquizitive.png`. It is byte-identical to the icon on the
     Karacho site.
   - Impact: One new binary file, no image editing.
5. **Publishing:** One commit "Add Karacho", then a push to `main`.
   - Why: Cloudflare Pages builds on every push; there is deliberately no local deploy.
   - Impact: The push is its own step and happens only after Peter has checked the local
     preview.

## Current State

Adding an app is a data-only edit (`README.md`, *Content*). The last one, commit `a876e96`
"Add Inquizitive", touched exactly `public/icons/inquizitive.png` and `src/data/apps.json`.

```
src/data/apps.json ──► src/pages/index.astro:33 ──► src/components/AppRow.astro
   (4 entries)           apps.map(AppRow)             icon │ name
                                                           │ tagline
public/icons/<id>.png  (1024×1024)                         │ description
                                                           │ link ? CTA "linkLabel →" : badge
scripts/check-assets.mjs ──► every asset referenced from dist/*.html exists
```

| # | App | Accent | Link |
|---|---|---|---|
| 1 | PlayTales | `#FF9500` | play-tales.app |
| 2 | PhotoMemo+ | `#617B92` | photo-memo.peterkurzok.de |
| 3 | TV Graphs | `#8B5CF6` | tvgraphs.peterkurzok.de |
| 4 | Inquizitive | `#0B6E8A` | inquizitive.peterkurzok.de |

## Desired End State

A fifth row after Inquizitive:

```
┌──────┐  Karacho
│ icon │  Your GPS speed in big amber digits
└──────┘
          A full-screen digital speedometer for iPhone: your speed in
          large seven-segment digits you can read at a glance, with a
          symbol for walking, cycling or driving. Free to use, no
          account, no analytics. Pro, a one-time purchase, adds the
          Siri answer and your speed on the Lock Screen.

          karacho.peterkurzok.de →
```

The entry, appended to the array in `src/data/apps.json`:

```json
{
  "id": "karacho",
  "name": "Karacho",
  "tagline": "Your GPS speed in big amber digits",
  "description": "A full-screen digital speedometer for iPhone: your speed in large seven-segment digits you can read at a glance, with a symbol for walking, cycling or driving. Free to use, no account, no analytics. Pro, a one-time purchase, adds the Siri answer and your speed on the Lock Screen.",
  "icon": "/icons/karacho.png",
  "accent": "#FFB020",
  "link": "https://karacho.peterkurzok.de",
  "linkLabel": "karacho.peterkurzok.de",
  "badge": null
}
```

## Abstractions and Code Reuse

Everything is reused; nothing new is introduced. `AppRow.astro` renders the entry from its
`App` interface, `Base.astro` derives the row tint and the ink colour from `accent`, and
`scripts/check-assets.mjs` covers the new icon reference.

- `public/icons/`
  - `karacho.png` - new; copy of `~/ws-privat/Speedo/Screenshots/app-icon.png`
- `src/data/`
  - `apps.json` - append the Karacho object

`README.md`, `privacy.astro` and `imprint.astro` name no individual app and need no change.

## Logging & Observability

None. The site is static and has no client-side JavaScript.

## Implementation

Dependencies: None.

**Tasks**:
- [ ] Copy `~/ws-privat/Speedo/Screenshots/app-icon.png` to `public/icons/karacho.png`.
- [ ] Append the Karacho object from *Desired End State* to `src/data/apps.json`, after the
      Inquizitive entry.
- [ ] Run `npm run build` and `npm run check`, then the checks under *Automated Verification*
      up to the commit.
- [ ] Start `npm run preview` in the background and hand the URL to Peter for the checks under
      *Manual Verification*. Stop the preview afterwards.
- [ ] After Peter's confirmation, commit both files as "Add Karacho".
- [ ] After the commit, push to `main`. This publishes the site.
- [ ] Wait for the Cloudflare Pages build and run the live checks.

**Automated Verification**:
- [ ] `sips -g pixelWidth -g pixelHeight public/icons/karacho.png` reports 1024 × 1024.
- [ ] `cmp ~/ws-privat/Speedo/Screenshots/app-icon.png public/icons/karacho.png` reports no
      difference.
- [ ] `npm run build` exits 0.
- [ ] `npm run check` exits 0 and `dist/icons/karacho.png` exists.
- [ ] `jq 'length' src/data/apps.json` prints `5`, and `jq -r '.[-1].id' src/data/apps.json`
      prints `karacho`.
- [ ] The existing entries are unchanged:
      `diff <(git show HEAD:src/data/apps.json | jq '.[0:4]') <(jq '.[0:4]' src/data/apps.json)`
      prints nothing.
- [ ] `grep -o 'class="app-row"' dist/index.html | wc -l` prints `5`.
- [ ] `dist/index.html` contains `id="app-karacho"`, `Your GPS speed in big amber digits`,
      `--accent: #FFB020` and two occurrences of `href="https://karacho.peterkurzok.de"`.
- [ ] After the push: `curl -s https://apps.peterkurzok.de/ | grep -c 'id="app-karacho"'`
      prints `1`, and `curl -sI https://apps.peterkurzok.de/icons/karacho.png` answers 200.

**Manual Verification**:
- [ ] In the local preview, the Karacho row sits below Inquizitive, and its tagline and call
      to action are readable in light and in dark mode.
- [ ] The row looks right at phone width (icon above the text) and at desktop width (icon
      beside the text).
- [ ] The icon and the call to action both open https://karacho.peterkurzok.de.

## Implementation Notes

During implementation, document user feedback, problems, and decisions here.

## References

- Commit `a876e96` "Add Inquizitive" - the same change for the previous app.
- `README.md`, *Content* and *Deployment*.
- `~/ws-privat/Speedo/Packages/lib-DesignSystem/Sources/DesignSystem/Palette.swift:29` - the
  amber `#FFB020`.
- `~/ws-privat/karacho-web/src/data/site.ts` - tagline and launch status of the Karacho site.
- `~/ws-privat/Speedo/metadata/version/1.0/en-US.json` - the App Store description the entry's
  claims come from.
