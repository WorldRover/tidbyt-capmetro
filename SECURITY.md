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
- **No user input**: The app accepts no user-supplied input at runtime; all data comes from the CapMetro public feed.
- **Cache TTL**: Vehicle position data is cached for 1 second. There is no authentication on the upstream feed.
