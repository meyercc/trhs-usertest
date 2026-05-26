# trhs-usertest

Frozen snapshots of the [Treehouse prototype](https://github.com/meyercc/trhs)
for user-testing sessions. Each subfolder is a self-contained `vite build` at a
point in time. Published via GitHub Pages.

**Landing page:** https://meyercc.github.io/trhs-usertest/

Every snapshot is independent — shipping new builds or editing styles on `main`
in `trhs/` never changes a snapshot that's already shipped.

## Workflow

All commands run from the `trhs/` repo (sibling directory).

### Ship a new snapshot

```bash
npm run ship:usertest -- <slug> [--theme default|kintsugi] [--label "..."] [--note "..."]
```

| flag | what it does | default |
| --- | --- | --- |
| `<slug>` | kebab-case folder name, e.g. `v2-mouse-modal` | required |
| `--theme` | which prototype the landing card links to | `default` |
| `--label` | human-readable title on the landing page | the slug |
| `--note` | one-line subtitle on the landing card | none |

Both themes (`prototype.html` and `prototype-kintsugi.html`) are always built
into every snapshot — `--theme` just decides which one the landing card opens.
Participants can manually swap the URL's filename to switch themes mid-session.

**Re-shipping the same slug overwrites that snapshot in place.** Useful for
quick fixes during a test session.

Example:

```bash
npm run ship:usertest -- v2-mouse-modal \
  --theme kintsugi \
  --label "Mouse Modal · Round 1" \
  --note "Testing remap flow"
```

### Unship a snapshot

```bash
npm run unship:usertest -- <slug>
```

Deletes the folder, removes the manifest entry, commits, and pushes. URL goes
404 within ~30s (CDN may briefly serve the old build).

### What each command does under the hood

`ship:usertest` → `vite build` → `cp -R dist/ → trhs-usertest/<slug>/` →
update `versions.json` → commit + push.

`unship:usertest` → `rm -rf trhs-usertest/<slug>/` → remove from
`versions.json` → commit + push.

## Repo layout

```
trhs-usertest/
├── index.html        landing page; reads versions.json + lists builds
├── versions.json     manifest, newest first; written by the ship scripts
├── .nojekyll         lets Pages serve vite's _bundle/ folder
└── <slug>/           one per shipped build; serves <slug>/prototype.html
```

## Notes for testers

The prototype is fully static — there's no backend. Features that depend on
local sidecar services (RGB sync, game library, sensor telemetry) will
gracefully no-op in the hosted build. Everything else (navigation,
peripheral modals, lighting presets, 3D viewers) works as designed.

## Caveats

- **CDN cache:** Pages serves through a CDN, so an unshipped URL may briefly
  remain reachable. New page loads start 404'ing once Pages rebuilds.
- **Git history:** unshipped snapshots remain in the repo's git history. For
  user testing that's fine; if something ever needs to be scrubbed entirely,
  that's a separate `git filter-repo` job.
- **Asset size:** each snapshot is ~96 MB. The `copy-static-assets` plugin
  in `trhs/vite.config.js` greps source for `Assets/*.ext` references and
  only ships referenced files — so adding a new asset to a JS/CSS/HTML
  source path is enough; the build picks it up. For paths the scanner
  can't see (fully runtime-computed), add them to `FORCE_INCLUDE_ASSETS`
  at the top of `vite.config.js`. The remaining bulk is the four 3D `.glb`
  models in `Assets/3d-devices/` (~38 MB combined); Draco-compressing them
  is the next lever if size becomes an issue.
