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

1. Read `stop_1`&ndash;`stop_4` and `max_departures` from config (defaults: `stop_1 = "603"`, others empty, `max_departures = "2"`).
2. If no stops are configured at all, return the "No stops set" display without touching the network.
3. Check `cache` for previously fetched departures using a per-stop-set, per-limit key (`capmetro_deps_<stop_ids>_<max>`, TTL: 60s).
4. On cache miss, fetch GTFS trip updates JSON from `CAPMETRO_TRIP_UPDATES_URL` and call `find_departures()` to collect the `max_departures` soonest arrivals across all configured stops.
5. If the feed is unreachable or no departures are found, return a graceful no-service display instead of crashing.
6. Dispatch on the number of departures **actually found**, never on `max_departures` — so the layout is always fully populated.

**Lookup dictionaries** (defined at the top of the file, before `main()`):

- `route_colors` — route ID → hex color string (blue `004A97` for local, gray `555555` for MetroRapid, red `E2231A` for express/rail). Generated from the GTFS feed; `FALLBACK_ROUTE_COLOR` covers a route the feed has but the dict does not.
- `stops` — stop ID → intersection/station name. Generated from the GTFS feed; an unknown ID renders as `Stop <id>`.

**Display layouts** (64×32 pixels). Stops are a net; `max_departures` decides how many of the soonest results are shown:

| departures | layout |
| --- | --- |
| 1 | `dome_bar()` — full-height bar in the route's color, dome above and route number below, with `tall_lines()` vertically centered beside it |
| 2, same route | one merged `dome_bar()`; the text column keeps both stop names, so merging on route alone stays unambiguous across two stops |
| 2, different routes | two 16px `chip()`s beside four 8px text lines |
| 3 | the soonest keeps the two-line treatment; the other two become `compact_row()`s |
| 4 | four `compact_row()`s |

`compact_row()` is 16px chip · 1px · marquee stop name · `SPACE_W` · `ETA_W` right-aligned `eta_short()` (`Due` / `8m` / `>1h`), so the ETAs read as a column against ragged stop names.

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
- `BADGE_W` (16) / `ETA_W` (12) / `SPACE_W` (4) — column widths for the chip, the right-aligned compact ETA, and the gap between the stop name and that ETA
- `FALLBACK_ROUTE_COLOR` (`000000`) — chip color for a route the feed reports but `route_colors` does not know
- `DOME` — base64 CapMetro dome icon, drawn in the no-service displays and above the route number in `dome_bar()`
- Tidbyt device ID and API token live in `creds.txt`, which is gitignored and must never be committed

## Key references

- Pixlet docs: [authoring_apps.md](https://github.com/tidbyt/pixlet/blob/main/docs/authoring_apps.md)
- Widget reference: [widgets.md](https://github.com/tidbyt/pixlet/blob/main/docs/widgets.md)
- Font reference: [fonts.md](https://github.com/tidbyt/pixlet/blob/main/docs/fonts.md)
- GTFS trip updates API: [data.texas.gov](https://data.texas.gov/download/mqtr-wwpy/text%2Fplain)
- Sample responses in `resources/vehiclepositions.json` and `resources/vehiclepositions-2.json` are from the **vehicle positions** feed, which this app no longer uses. Note that the trip updates feed encodes `arrival.time` as a **string**, not a number.
