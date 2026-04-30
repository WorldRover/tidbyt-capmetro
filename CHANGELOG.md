# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added

- ETA display from GTFS trip updates feed (`mqtr-wwpy`): row 2 now shows "In X min" / "Due" / ">1 hr" instead of speed when a prediction is available; falls back to MPH when not
- `get_eta_minutes()` — fetches trip updates, matches on `tripId` + `stopId`, returns minutes to arrival
- `eta_text()` — formats ETA integer into display string
- ETA cached per-route alongside speed/status/stop (60s TTL)

### Added

- `schema()` function — users can now configure their route in the Tidbyt app
- `main()` now accepts `config`; filters vehicles by the configured route ID (default: 803)
- Per-route cache keys so multiple users watching different routes don't stomp each other
- `no_service_display()` helper renders a graceful "Not in service" screen instead of crashing when no vehicle matches the route
- Feed HTTP errors now return a "Feed unavailable" display instead of crashing

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
