# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.5.0] - 2026-08-28

### Added

- **Departures shown** setting — a dropdown choosing how many arrivals appear at once, 1 to 4 (default 2). ([#33](https://github.com/WorldRover/tidbyt-capmetro/pull/33))
- **Stop 4** — a fourth optional stop field. ([#33](https://github.com/WorldRover/tidbyt-capmetro/pull/33))
- `AGENTS.md` — the canonical model-neutral agent guidance file, with `CLAUDE.md` and `GEMINI.md` as symlinks to it. ([#35](https://github.com/WorldRover/tidbyt-capmetro/pull/35))

### Changed

- The display now picks one of five layouts based on how many departures were actually found, rather than always drawing two rows, so the screen is never half-empty. One departure gets a full-height bar in the route's color with the dome above the route number; two on the same route merge into one bar; three and four break into compact rows with the abbreviated ETA (`Due` / `8m` / `>1h`) right-aligned in a fixed column so the ETAs read as a column against ragged stop names. ([#33](https://github.com/WorldRover/tidbyt-capmetro/pull/33))
- The no-service screens ("No stops set", "No departures", "Feed unavailable") use the same full-height color bar with the dome vertically centered, instead of leaving the left column black. ([#33](https://github.com/WorldRover/tidbyt-capmetro/pull/33))
- An unknown stop ID renders as `Stop 1145` rather than the bare number. ([#33](https://github.com/WorldRover/tidbyt-capmetro/pull/33))
- README now carries the project documentation a contributor would look for — how the app works, the layout table, the conventions, and the feed and reference links — instead of keeping it in an agent-tool file. ([#35](https://github.com/WorldRover/tidbyt-capmetro/pull/35))
- Credentials are documented as `pixlet login`, which stores them outside the repo, with `$TIDBYT_API_TOKEN` as the scripted fallback. ([#34](https://github.com/WorldRover/tidbyt-capmetro/pull/34), [#35](https://github.com/WorldRover/tidbyt-capmetro/pull/35))

### Removed

- `resources/` — three 2022 captures of the vehicle positions feed, which this app does not read, and three unreferenced PNGs. ([#35](https://github.com/WorldRover/tidbyt-capmetro/pull/35))
- `route_colors["000"]` sentinel, replaced by `FALLBACK_ROUTE_COLOR`. ([#33](https://github.com/WorldRover/tidbyt-capmetro/pull/33))

## [0.4.0] - 2026-08-28

### Changed

- Route badges now span both lines of their departure. The display is two side-by-side columns — a 16px badge column and a 48px text column — rather than two stacked departure rows. Each badge is a 16x16 color chip with the route ID vertically centered. ([#25](https://github.com/WorldRover/tidbyt-capmetro/pull/25))
- README preview regenerated to show the route color coding and the stop-name marquee. The previous image used two MetroRapid stops, whose shared `555555` brand color made every badge gray, and both stop names were long enough that any still frame caught both marquees mid-scroll. ([#24](https://github.com/WorldRover/tidbyt-capmetro/pull/24))

### Removed

- The 17px spacer box that padded the stop-name marquee into alignment with the ETA text. The badge column now provides that offset structurally, so the marquee gets the full text column. ([#25](https://github.com/WorldRover/tidbyt-capmetro/pull/25))

## [0.3.0] - 2026-08-28

### Fixed

- App failed to load at all: `def schema()` shadowed the `schema` module loaded at the top of the file. Renamed to `get_schema()`, which is also the name pixlet expects.
- `find_departures()` crashed with `unknown binary op: string - int` — the trip updates feed encodes `arrival.time` as a JSON string, not a number. It is now parsed with `int()`.
- `find_departures()` called `deps.sort()`, which does not exist in Starlark. Replaced with `sorted()`.
- ETAs went stale for the life of a cache entry: the relative minute count was computed once at fetch time and cached for 60s. Absolute arrival times are now cached and the ETA is recomputed on every render.
- Buses that had already departed within the last 60 seconds displayed as "Due", because `int()` truncates `-0.5` to `0`. Departures are now filtered on absolute arrival time before the ETA is computed.
- A feed response that returned HTTP 200 without an `entity` key crashed instead of showing the no-service display.
- "Feed unavailable" and "No departures" were clipped by the 47px of space left beside the icon. The message is now a marquee.
- Routes `320`, `454`, `800`, and `837` were missing from `route_colors` and rendered a black badge instead of their brand color. `800` and `837` are active MetroRapid routes. ([#21](https://github.com/WorldRover/tidbyt-capmetro/pull/21))

### Added

- `manifest.yaml` — metadata required to publish to the Tidbyt community app repo.
- README disclaimer stating the app is unaffiliated with CapMetro and that arrival times are predictions, not guarantees.

### Removed

- `route_names` lookup dict — 79 entries that were never referenced; the route badge renders the raw route ID. Tracked for reinstatement in the scrolling row ([#19](https://github.com/WorldRover/tidbyt-capmetro/issues/19)).

### Changed

- Renamed `CapMetro.star`/`CapMetro.webp` to `capmetro.star`/`capmetro.webp`. The community app repo requires lowercase filenames, and the mixed case meant `pixlet render` wrote a second file on case-sensitive filesystems.
- `.claude/settings.json` (WorldRover branch-name pre-push hook) and `.claude/.wr-canon` are no longer tracked — personal tooling that would block outside contributors.
- CI now actually renders the app. The job named `render` only ran `pixlet format`, which never executes the applet and so caught none of the above.
- Corrected `SECURITY.md`, which claimed the app takes no user input and cached for 1 second.

## [0.2.0] - 2026-05-01

### Changed

- Redesigned as a **departures board**: shows the next 2 arrivals across up to 3 configured stops instead of tracking a single route's vehicle
- Schema now accepts `stop_1`/`stop_2`/`stop_3` stop IDs instead of a `route_id`
- Data source switched from GTFS vehicle positions to GTFS trip updates feed exclusively — ETA is always a real prediction, never a speed estimate
- Display layout changed to two 16-px departure rows (route badge + ETA + scrolling stop name) filling the full 64×32 canvas
- Cache key is now per stop-set (`capmetro_deps_<stop_ids>`) instead of per route

### Added

- `find_departures(stop_ids, entities, now_unix)` — scans trip updates for any of the configured stops and returns the 2 soonest arrivals
- `encode_deps()` / `decode_deps()` — compact string serialization for caching departure tuples
- `dep_eta_text()` — formats an ETA integer into "Due" / "In N min" / ">1 hr"
- `departure_row()` — renders one 16-px departure row (colored route badge, ETA, scrolling stop name)

### Removed

- GTFS vehicle positions feed (`CAPMETRO_GTFS_URL`) — no longer used
- `get_eta_minutes()`, `eta_text()` — replaced by `find_departures()` + `dep_eta_text()`
- `statuses` lookup dict and speed/MPH display — departures board doesn't show vehicle status
- `MS_TO_MPH`, `ETA_NONE`, `DEFAULT_ROUTE` constants

## [0.1.0] - 2026-04-30

### Fixed

- Speed was displayed in m/s (raw GTFS value) instead of MPH — now converted correctly
- Crash when no vehicle in the feed has an active trip — now fails with a clear message
- Route IDs were cast through `int()` before caching, corrupting IDs like `"000"` to `"0"`
- Cache TTL was 1 second (effectively no caching) — raised to 60 seconds to match `max_age`
- Partial cache hit (only `route` key checked) could cause a `None` dereference on other keys

### Changed

- Removed debug `print()` statements
- Removed dead `main_align = "space_between"` on single-child `render.Row`
