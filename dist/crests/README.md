# Crest library

Club and county crests used by the operator console and by the ProBoards match
centre posts. The Roscommon county crest in `brand/` remains the service
identity; this directory is the per-team image library.

## Layout

- `clubs/<slug>.png` — one crest per affiliated Roscommon club, plus a
  112px square `-post.png` derivative sized for the forum medallion.
- `counties/<slug>.png` — the 32 counties plus London, New York, Lancashire,
  and Warwickshire.
- `fallback/gaa-logo.png` — the GAA mark used for amalgamations, second teams
  and any team name with no crest of its own.

The register itself lives in `apps/api/src/crest-manifest.ts`. Every entry
records the source page, the retrieval date, the original image size and
format, the licence note, and SHA-256 checksums for both stored files.

## Sources

- **Clubs** — the official Roscommon GAA club directory
  (<https://www.gaaroscommon.ie/clubs/>), which publishes one crest per club.
  Most club crests are published at 137×137, so the stored file keeps the
  original resolution rather than upscaling. `Pádraig Pearses` and `Tremane`
  publish larger artwork.
- **Counties** — the Wikipedia crest file for each county board, preferring
  vector artwork and the most recent file. Roscommon itself comes from the
  county board's own crest; Lancashire and Warwickshire use the GAA Fixture
  Centre images published for their official inter-county fixtures.
  (`gaaroscommon.ie/.../logo-gaa-roscommon.svg`).
- **Fallback** — `File:Logo of GAA.svg` (public domain).

Each crest remains the property of its club or county board, and inclusion here
identifies the team in results coverage only. It does not imply affiliation or
endorsement. Images are served from this deployment rather than hot-linked so
that an operator page view or a forum post never makes a third-party request.

## Refreshing

```powershell
pnpm crests:scrape           # refresh every asset, then rewrite the manifest
pnpm crests:scrape --only=counties
pnpm crests:scrape --force   # re-download even when a cached copy exists
```

Downloads are cached under `.data/crest-downloads/` and Wikimedia API responses
under `.data/crest-scrape-cache.json`, so a re-run is quick and polite to the
upstream sources. The script re-encodes every image to PNG (maximum 192px for
the portal, with forum artwork optically fitted inside a fixed 112px square),
keeps transparency, and fails the run if any team ends up without a usable
image. The forum fit uses the artwork's bounding-circle diameter so narrow
shields and round crests retain comparable visual weight while preserving their
original aspect ratios.

## How a team resolves to a crest

`apps/api/src/crests.ts` resolves a fixture name in this order:

1. a registered name or alias (`Pádraig Pearses`, `Pearses`);
2. a Roscommon club matched on the post's own abbreviation rules;
3. a county name, including the Irish form (`Ros Comáin`);
4. otherwise the GAA mark.

A name that splits into member clubs, such as `Oran/St. Croan's`, is treated as
an amalgamation and takes the GAA mark rather than one member's crest.

Posts stay text-only when the deployment has no public asset origin. Set
`CREST_ASSET_BASE_URL` (defaults to `WEB_ORIGIN`) to the public HTTPS origin
that serves this directory.
