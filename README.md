# CapMetro Tidbyt App

## Overview

Displays real-time Capital Metro (Austin, TX) departure predictions on a [Tidbyt](https://tidbyt.com/) device.

![CapMetro app preview](capmetro.webp)

Shows the next 2 bus or rail arrivals across up to 3 configured stops — pulled live from the CapMetro GTFS-RT trip updates feed. Each row shows a color-coded route badge, arrival ETA ("Due" / "In N min" / ">1 hr"), and a scrolling stop name.

## Getting started

In the Tidbyt app, set **Stop 1** (required), **Stop 2**, and **Stop 3** (optional) to the numeric CapMetro stop IDs you want to monitor. For example, stop `603` is 31st Street Station northbound.

Find your stop ID on [CapMetro's trip planner](https://www.capmetro.org/planner) — it appears in the stop URL and on the printed sign at the stop — or in the `stops.txt` file of the [GTFS static feed](https://data.texas.gov/dataset/CapMetro-GTFS/eiei-9rpf).

## Usage

Requires the [pixlet](https://github.com/tidbyt/pixlet) CLI.

```bash
# Preview locally
pixlet serve capmetro.star

# Render to image
pixlet render capmetro.star

# Push to device
pixlet push --api-token <token> <device-id> capmetro.webp
```

## Disclaimer

This is an unofficial, independently built app. It is **not affiliated with,
endorsed by, or supported by** the Capital Metropolitan Transportation Authority
(CapMetro). "CapMetro" and the CapMetro logo are trademarks of their respective
owner, used here only to identify the transit system whose public data the app
displays.

Arrival times come from CapMetro's public GTFS-Realtime feed and are
**predictions, not guarantees**. The feed may be delayed, incomplete, or
unavailable, and the app caches results for up to 60 seconds. Use at your own
risk — do not rely on it to catch a bus or train, or to make any time-critical
decision. The software is provided "as is", without warranty of any kind, as set
out in the [LICENSE](LICENSE).

## License

MIT © 2026 Dan Ziegler
