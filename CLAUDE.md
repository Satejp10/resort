# Resort

A **design prototype** of a phone photo app, built as a web app that runs
full-screen from an Android home screen. It covers one loop only: photo grid →
open a photo → select many. The target feel is Google Photos on Android, minus
the forced AI.

This is a hobby demo. Front end only, fake data, no backend.

## Hard rules

- **`reference/` is read-only and never committed.** It holds an upstream clone
  of `immich-app/immich` for reading. Do not modify anything inside it. It is
  gitignored; keep it that way.
- **Never paste Immich source into this repo.** Immich is AGPL-3.0; copying it
  would make this repo a derivative work. Re-implement its patterns instead.
- **No glass or glassmorphism.** Liquid Glass was vetoed for this project. Do not
  apply the `liquid-glass-design` skill here, even though it is the default
  elsewhere.
- **No third-party logos or brand assets** — Immich's, Google's or anyone else's.
  Every screen carries a "Design prototype" label.
- **No backend, accounts, uploads, encryption or real APIs.** Fake data only.
- **No AI surfaces** — no "Ask", no suggestion cards, no auto-creations.
- **Never commit the build output.** `app/build/` is gitignored; the Pages
  workflow builds the site from source on every push to `main`.

## Out of scope

Albums and Search content, the Feed tab, settings, memories, editing, sharing.
Tab stubs at most.

## Stack and commands

SvelteKit 2 + Svelte 5 + Tailwind 4, static build (the same stack Immich web
uses, so its timeline code reads as reference 1:1).

```bash
cd app
npm install
npm run dev      # http://localhost:5173/resort/
npm run build    # writes the static site to app/build
npm run check    # svelte-check; keep this at zero errors
```

The build is prerendered with `ssr = false`: the grid's geometry depends on a
real viewport, and a server has no viewport to measure. Base path is `/resort`;
override with `BASE_PATH=''` for a root-hosted preview.

Append `?debug` to the URL for a live count of tiles in the DOM.

## Deployment

Live at <https://satejp10.github.io/resort/> once Pages is switched on.

`.github/workflows/pages.yml` rebuilds and publishes on every push to `main`;
it typechecks first, so a push that fails `npm run check` does not deploy.

Three things that cost a cycle to learn, so do not re-derive them:

- **Pushing to `.github/workflows/` from a cloud session works.** Verified
  2026-09-21.
- **A workflow cannot enable Pages.** `actions/configure-pages` with
  `enablement: true` fails with *"Create Pages site failed: Resource not
  accessible by integration"* — the Actions token is refused on that API. A
  human sets Settings → Pages → Source → GitHub Actions once. The workflow
  checks the Pages API first and skips the deploy with a warning until then,
  so an unconfigured repository leaves `main` green rather than permanently red.
- **Pages URLs are case-sensitive and follow the repo name.** The repo is
  `resort`, lowercase, to match the base path `/resort`. Renaming the repo moves
  the site, and GitHub does not redirect a project site's old URL.

## Testing in headless Chromium

- **Do not route the browser through the sandbox's `HTTPS_PROXY`** to fetch
  thumbnails. The proxy swallows loopback requests too, so the page under test
  comes back blank; `bypass: '127.0.0.1,localhost'` does not help. Pre-fetch
  photos with curl and fulfil `**/picsum.photos/**` from disk inside the test.
  A real phone reaches picsum directly.
- **`NODE_PATH` does not work for ESM imports.** Import a globally installed
  `playwright` by absolute path.

## Design tokens — "Ash"

Defined once in `app/src/app.css`. Every token has **separate light and dark
values, status colors included** — that is deliberate: a status color shared
unchanged between the two schemes is tuned for one background and wrong on the
other. Ash also avoids untinted pure-black/white neutrals and any accent
borrowed from a brand.

Fonts: Space Grotesk for headings, Inter for UI.

Dark mode follows the OS setting only. There is no in-app theme switch.

## Where things are

| Path | What |
|---|---|
| `app/src/lib/data/library.ts` | Seeded synthetic photo library and day grouping |
| `app/src/lib/timeline/layout.ts` | Pure geometry: group placement, visible window, scrubber segments |
| `app/src/lib/timeline/pinch.ts` | Two-finger pinch reported as discrete density steps |
| `app/src/lib/components/Timeline.svelte` | Scroll container, virtualization, pinch anchoring |
| `.claude/context/LOG.md` | Append-only session log — add an entry every session |
| `.claude/context/reports/` | Status reports for pasting back into chat (`/project-status`), numbered from `SR-resort-001` |
