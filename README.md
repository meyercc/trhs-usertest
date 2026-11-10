# trhs-usertest

Frozen snapshots of the Treehouse prototype for user-testing sessions. Each
subfolder is a self-contained build of the React app (`trhs/web`) at one
commit, published with GitHub Pages.

**Landing page:** https://meyercc.github.io/trhs-usertest/

Every snapshot is independent: shipping new builds or changing `main` in
`trhs/` never changes a snapshot that's already shipped.

## Workflow

All commands run from the `trhs/` repo (sibling directory).

### Ship a new snapshot

```bash
npm run ship:usertest -- <slug> [--label "..."] [--note "..."] [--dry] [--allow-dirty]
```

| flag | what it does | default |
| --- | --- | --- |
| `<slug>` | kebab-case folder name, e.g. `v2-user-testing` | required |
| `--label` | human-readable title on the landing page | the slug |
| `--note` | one-line subtitle on the landing card | none |
| `--dry` | build and copy here, but don't commit or push | off |
| `--allow-dirty` | build even with uncommitted changes in `trhs/` | off |

A snapshot is built from **committed** code, and its `versions.json` entry
records the commit, so you can always tell what participants saw.
**Re-shipping the same slug overwrites that snapshot in place**, which is
useful for quick fixes during a test round.

Example:

```bash
npm run ship:usertest -- v2-user-testing \
  --label "General User Testing · Round 2" \
  --note "Treehouse React app — Nova agent stories, devices, Play"
```

### Unship a snapshot

```bash
npm run unship:usertest -- <slug>
```

Deletes the folder, removes the manifest entry, commits, and pushes. The URL
starts returning 404 once Pages rebuilds (the CDN may briefly serve the old
build).

### What each command does under the hood

`ship:usertest`: `npm run build` in `web/` → `cp -R web/dist/ → trhs-usertest/<slug>/`
→ update `versions.json` → commit + push.

`unship:usertest`: `rm -rf trhs-usertest/<slug>/` → remove from
`versions.json` → commit + push.

## Repo layout

```
trhs-usertest/
├── index.html        landing page; reads versions.json + lists builds
├── versions.json     manifest, newest first; written by the ship scripts
├── .nojekyll         lets Pages serve folders whose names start with _
└── <slug>/           one per shipped build; serves <slug>/index.html
```

## Notes for testers

The prototype is fully static, with no backend. Features that depend on
local services (RGB sync, sensor telemetry, a local Nova model) no-op or fall
back to scripted behavior in the hosted build. The scripted Nova stories run
without a model.

**Start each session clean.** Every snapshot, and the agency reference, is
served from `meyercc.github.io`, so they all share one localStorage. State
from one session or snapshot carries into the next in the same browser. Use a
fresh browser profile or a private window per participant, or the story's
own reset (for example `#/?demo=autocast`, `#/?demo=catch-up`).

## Caveats

- **CDN cache:** Pages serves through a CDN, so an unshipped URL may briefly
  remain reachable.
- **Git history:** unshipped snapshots remain in the repo's git history. If
  something ever needs to be scrubbed entirely, that's a separate
  `git filter-repo` job.
- **Asset size:** each React snapshot is about 110 MB (3D models and the
  gameplay clips). No single file is near GitHub's 100 MB limit; keep an eye
  on the repo total (Pages sites are capped at 1 GB) and unship old rounds.
- **Public:** the repo and its Pages site are public, gameplay clips
  included.
