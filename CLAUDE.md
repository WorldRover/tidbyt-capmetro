# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

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

`CapMetro.star` is the entire app — one Starlark file with no imports beyond the Tidbyt standard library (`render`, `http`, `cache`, `encoding/base64`).

**Data flow in `main()`:**
1. Check `cache` for previously fetched vehicle data (TTL: 1 second).
2. On cache miss, fetch GTFS vehicle positions JSON from `CAPMETRO_GTFS_URL` (`data.texas.gov`), extract the first vehicle entity that has a trip.
3. Store route, speed, status, and stop in cache.
4. Resolve human-readable names/colors from the lookup dictionaries, then build and return a `render.Root` layout.

**Lookup dictionaries** (defined at the top of the file, before `main()`):
- `route_names` — route ID → display name
- `route_colors` — route ID → hex color string (blue `004A97` for local, gray `555555` for MetroRapid, red `E2231A` for express/rail)
- `stops` — stop ID → intersection/station name
- `statuses` — GTFS status code → display string

**Display layout** (64×32 pixels, three rows):
- Row 1: colored route number badge + scrolling route name marquee
- Row 2: CapMetro icon + vehicle status text + speed in MPH
- Row 3: stop ID badge + scrolling stop name marquee

## Key References

- Pixlet docs: https://github.com/tidbyt/pixlet/blob/main/docs/authoring_apps.md
- Widget reference: https://github.com/tidbyt/pixlet/blob/main/docs/widgets.md
- Font reference: https://github.com/tidbyt/pixlet/blob/main/docs/fonts.md
- GTFS vehicle positions API: https://data.texas.gov/download/cuc7-ywmd/text%2Fplain
- Sample responses are in `resources/vehiclepositions.json` and `resources/vehiclepositions-2.json`
