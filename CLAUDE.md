# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**GitHub:** [WorldRover/tidbyt-capmetro](https://github.com/WorldRover/tidbyt-capmetro)

## What This Is

A [Tidbyt](https://tidbyt.com/) app written in [Starlark](https://github.com/google/starlark-go/blob/master/doc/spec.md) that displays real-time CapMetro (Austin, TX) transit data on a Tidbyt LED display. The single app file is `capmetro.star`. `manifest.yaml` carries the metadata
required to publish to the [Tidbyt community app repo](https://github.com/tidbyt/community);
that repo requires the app filename to be lowercase, which is why it is `capmetro.star`
and not `CapMetro.star`.

## Development Commands

Requires the [`pixlet`](https://github.com/tidbyt/pixlet) CLI tool.

```bash
# Render the app to a WebP image
pixlet render capmetro.star

# Serve locally with live reload at http://localhost:8080
pixlet serve capmetro.star

# Push a rendered WebP to a physical Tidbyt device
pixlet push --api-token <token> <device-id> capmetro.webp
```

## Architecture

`capmetro.star` is the entire app — one Starlark file with no imports beyond the Tidbyt standard library (`render`, `http`, `cache`, `encoding/base64`, `schema`, `time`).

**Data flow in `main(config)`:**

1. Read `stop_1`/`stop_2`/`stop_3` from config (default: `stop_1 = "603"`, others empty).
2. Check `cache` for previously fetched departures using a per-stop-set key (`capmetro_deps_<stop_ids>`, TTL: 60s).
3. On cache miss, fetch GTFS trip updates JSON from `CAPMETRO_TRIP_UPDATES_URL`, call `find_departures()` to collect the 2 soonest arrivals across all configured stops.
4. If the feed is unreachable or no departures are found, return a graceful no-service display instead of crashing.
5. Build a `route_badge()` per result and a pair of `departure_lines()` per result, and return a `render.Root` with the badge column beside the text column.

**Lookup dictionaries** (defined at the top of the file, before `main()`):

- `route_colors` — route ID → hex color string (blue `004A97` for local, gray `555555` for MetroRapid, red `E2231A` for express/rail)
- `stops` — stop ID → intersection/station name

**Display layout** (64×32 pixels) — two side-by-side columns, not stacked rows:

- **Badge column** (`BADGE_WIDTH` = 16px): one 16x16 color chip per departure, spanning both of that departure's text lines, with the route ID centered in it.
- **Text column** (`TEXT_WIDTH` = 48px): two 8px lines per departure — ETA text ("Due" / "In N min" / ">1 hr") on top, scrolling stop name marquee below.

A single departure pads both columns with a 16px box so the layout does not stretch.

**No-service display** (shown when feed is unreachable or no departures found):

- CapMetro icon + "Feed unavailable" or "No departures" message

## Versioning

This project uses [Semantic Versioning](https://semver.org/). Tag releases as `vMAJOR.MINOR.PATCH` directly on `main` after merging.

**Cadence — buffer model:** batch small fixes into minor releases rather than shipping a patch for every commit. Cut a release when the `## [Unreleased]` section in CHANGELOG.md has accumulated meaningful changes, or immediately for any breaking change.

## Branches and pull requests

Branch names must follow `(feat|fix|refactor|chore|docs|test)/NNN-short-slug` where `NNN` is the issue number. PRs merge into `main`; no long-lived feature branches. Delete the branch after merge.

**PR labels:** every PR must have at least one label from the canonical scheme before merge — this is what drives `gh release create --generate-notes` grouping via `.github/release.yml`. Labeling the tracking issue is not enough; the PR itself needs the label.

## Labels

Required labels: `ui`, `data`, `infra`, `P1`, `P2`, `P3`, `type: bug`, `type: feature`, `type: docs`, `type: enhancement`. The default GitHub labels (`bug`, `enhancement`, `documentation`) should be removed.

## Key constants

- `CAPMETRO_TRIP_UPDATES_URL` — Texas.gov GTFS trip updates feed with arrival predictions (`https://data.texas.gov/download/mqtr-wwpy/text%2Fplain`)
- `DEFAULT_STOP` — stop shown when no config is provided (`603`, 31st Street Station NB)
- Tidbyt device ID and API token live in `creds.txt`, which is gitignored and must never be committed

## Key references

- Pixlet docs: [authoring_apps.md](https://github.com/tidbyt/pixlet/blob/main/docs/authoring_apps.md), [tutorial.md](https://github.com/tidbyt/pixlet/blob/main/docs/tutorial.md)
- Tidbyt community forum: [discuss.tidbyt.com](https://discuss.tidbyt.com)
- Widget reference: [widgets.md](https://github.com/tidbyt/pixlet/blob/main/docs/widgets.md)
- Font reference: [fonts.md](https://github.com/tidbyt/pixlet/blob/main/docs/fonts.md)
- GTFS trip updates: [raw feed](https://data.texas.gov/download/mqtr-wwpy/text%2Fplain) · [dataset page](https://data.texas.gov/Transportation/CapMetro-Trip-Updates-JSON-File/mqtr-wwpy)
- Other CapMetro feeds, unused by the app today: [vehicle positions](https://data.texas.gov/download/cuc7-ywmd/text%2Fplain) (~122 KB, see issue #28), [service alerts](https://data.texas.gov/download/9zu9-jwr2/text%2Fplain) (~79 KB, bare JSON array, see issue #8), [GTFS static zip](https://data.texas.gov/download/r4v4-vz24/application%2Fzip) (~34 MB — too large for `http.get`, see issue #20)
- CapMetro route maps: [all bus routes](https://www.capmetro.org/ourservices/busroutes) · [single route](https://www.capmetro.org/schedmap/?svc=0&f1=214) (change `f1=` to the route ID)
- Regenerating `CAPMETRO_ICON`: draw at [pixilart](https://www.pixilart.com/draw), encode with [base64.guru](https://base64.guru/converter/encode/image)
- Sample responses in `resources/vehiclepositions.json` and `resources/vehiclepositions-2.json` are from the **vehicle positions** feed, which this app no longer uses. Note that the trip updates feed encodes `arrival.time` as a **string**, not a number.
