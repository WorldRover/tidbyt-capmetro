# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
