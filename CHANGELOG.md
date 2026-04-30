# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

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
