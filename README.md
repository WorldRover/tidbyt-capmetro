# CapMetro Tidbyt App

Displays real-time Capital Metro (Austin, TX) bus position data on a [Tidbyt](https://tidbyt.com/) device.

![CapMetro app preview](CapMetro.webp)

Shows the current route name, vehicle status (In Transit / At Stop), speed in MPH, and the next stop — all pulled live from the CapMetro GTFS vehicle positions feed.

## Usage

Requires the [pixlet](https://github.com/tidbyt/pixlet) CLI.

```bash
# Preview locally
pixlet serve CapMetro.star

# Render to image
pixlet render CapMetro.star

# Push to device
pixlet push --api-token <token> <device-id> capmetro.webp
```

## License

MIT © 2026 Dan Ziegler
