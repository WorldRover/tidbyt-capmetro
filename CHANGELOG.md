# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
