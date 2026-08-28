# Security Policy

## Reporting a vulnerability

Please do not open a public GitHub issue for security vulnerabilities. Instead, email the maintainer directly or use GitHub's private vulnerability reporting feature (Security → Report a vulnerability).

Expect an acknowledgement within 48 hours and a resolution or mitigation timeline within 14 days.

## Scope

This project is a Tidbyt display app. The attack surface is limited to:

- The CapMetro GTFS API response (`data.texas.gov`) — malformed or malicious payloads processed by `CapMetro.star`
- The Tidbyt API token and device ID stored outside this repository — if leaked, an attacker could push arbitrary images to the display

Out of scope: denial-of-service against third-party APIs, physical access to the Tidbyt device.

## Known security considerations

- **API token**: The Tidbyt API token is a bearer credential. It should be treated like a password and never committed to this repository.
- **User input**: The app accepts up to three stop IDs from the Tidbyt app config. These are used to filter the feed and to build a cache key; they are never interpolated into the upstream request URL.
- **Cache TTL**: Trip update predictions are cached for 60 seconds, keyed by the configured stop set. There is no authentication on the upstream feed.
