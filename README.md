# trhs-usertest

Frozen snapshots of the [Treehouse prototype](https://github.com/meyercc/trhs)
for user-testing sessions. Each subfolder is a self-contained vite build at a
point in time. Published via GitHub Pages.

**Landing page:** https://meyercc.github.io/trhs-usertest/

## Adding a new test build

From the `trhs/` repo (sibling directory):

```bash
npm run ship:usertest -- v2-mouse-modal --label "Mouse Modal · Round 1" --note "Testing remap flow"
```

The script:
1. Runs `vite build` (outputs to `trhs/dist/`)
2. Copies `dist/` → `trhs-usertest/<slug>/`
3. Updates `versions.json`
4. Commits + pushes — GitHub Pages picks up the change in ~30s

Re-shipping the same slug overwrites that version in place.

## What lives here

- `index.html` — landing page; reads `versions.json` and lists every shipped build
- `versions.json` — manifest of all shipped versions (newest first)
- `<slug>/` — one per shipped build; serves `<slug>/prototype.html`

## Notes for testers

The prototype is fully static — there's no backend. Features that depend on
local sidecar services (RGB sync, game library) will gracefully no-op in the
hosted build. Everything else (assignments, lighting presets, navigation,
3D viewers) works as designed.
