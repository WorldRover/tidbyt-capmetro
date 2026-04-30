# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

**GitHub:** https://github.com/WorldRover/tidbyt-capmetro

## What This Is

A [Tidbyt](https://tidbyt.com/) app written in [Starlark](https://github.com/google/starlark-go/blob/master/doc/spec.md) that displays real-time CapMetro (Austin, TX) transit data on a Tidbyt LED display. The single app file is `CapMetro.star`.

## Development Commands

Requires the [`pixlet`](https://github.com/tidbyt/pixlet) CLI tool.

```bash
# Render the app to a WebP image
pixlet render CapMetro.star

# Serve locally with live reload at http://localhost:8080
pixlet serve CapMetro.star

# Push a rendered WebP to a physical Tidbyt device
pixlet push --api-token <token> <device-id> capmetro.webp
```

## Architecture

`CapMetro.star` is the entire app — one Starlark file with no imports beyond the Tidbyt standard library (`render`, `http`, `cache`, `encoding/base64`, `schema`).

**Data flow in `main(config)`:**
1. Read `route_id` from config (default: `803`).
2. Check `cache` for previously fetched vehicle data using per-route keys (`capmetro_{route_id}_*`, TTL: 60s).
3. On cache miss, fetch GTFS vehicle positions JSON from `CAPMETRO_GTFS_URL`, find the first vehicle whose `trip.routeId` matches the configured route.
4. If the feed is unreachable or no matching vehicle is found, return a graceful no-service display instead of crashing.
5. Resolve human-readable names/colors from the lookup dictionaries, then build and return a `render.Root` layout.

**Lookup dictionaries** (defined at the top of the file, before `main()`):
- `route_names` — route ID → display name
- `route_colors` — route ID → hex color string (blue `004A97` for local, gray `555555` for MetroRapid, red `E2231A` for express/rail)
- `stops` — stop ID → intersection/station name
- `statuses` — GTFS status code → display string

**Display layout** (64×32 pixels, three rows — active vehicle):
- Row 1: colored route number badge + scrolling route name marquee
- Row 2: CapMetro icon + vehicle status text + ETA ("In X min" / "Due" / ">1 hr"), falling back to speed in MPH if no prediction available
- Row 3: stop ID badge + scrolling stop name marquee

**No-service display** (shown when route has no active vehicle):
- Row 1: colored route number badge + scrolling route name marquee
- Row 2: CapMetro icon + "Not in service" or "Feed unavailable" message

## Versioning

This project uses [Semantic Versioning](https://semver.org/). Tag releases as `vMAJOR.MINOR.PATCH` directly on `main` after merging.

**Cadence — buffer model:** batch small fixes into minor releases rather than shipping a patch for every commit. Cut a release when the `## [Unreleased]` section in CHANGELOG.md has accumulated meaningful changes, or immediately for any breaking change.

## Branches and pull requests

Branch names must follow `(feat|fix|refactor|chore|docs|test)/NNN-short-slug` where `NNN` is the issue number. PRs merge into `main`; no long-lived feature branches. Delete the branch after merge.

## Labels

Required labels: `ui`, `data`, `infra`, `P1`, `P2`, `P3`, `type: bug`, `type: feature`, `type: docs`, `type: enhancement`. The default GitHub labels (`bug`, `enhancement`, `documentation`) should be removed.

## Key constants

- `CAPMETRO_GTFS_URL` — Texas.gov GTFS vehicle positions feed (`https://data.texas.gov/download/cuc7-ywmd/text%2Fplain`)
- `CAPMETRO_TRIP_UPDATES_URL` — Texas.gov GTFS trip updates feed with arrival predictions (`https://data.texas.gov/download/mqtr-wwpy/text%2Fplain`)
- `DEFAULT_ROUTE` — route shown when no config is provided (`803`, MetroRapid Burnet/S Lamar)
- Tidbyt device ID and API token are stored outside the repo (see `creds.txt`, which is gitignored)

## Key references

- Pixlet docs: https://github.com/tidbyt/pixlet/blob/main/docs/authoring_apps.md
- Widget reference: https://github.com/tidbyt/pixlet/blob/main/docs/widgets.md
- Font reference: https://github.com/tidbyt/pixlet/blob/main/docs/fonts.md
- GTFS vehicle positions API: https://data.texas.gov/download/cuc7-ywmd/text%2Fplain
- Sample responses are in `resources/vehiclepositions.json` and `resources/vehiclepositions-2.json`
