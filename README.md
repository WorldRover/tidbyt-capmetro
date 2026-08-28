# CapMetro Tidbyt App

## Overview

Displays real-time Capital Metro (Austin, TX) departure predictions on a [Tidbyt](https://tidbyt.com/) device.

![CapMetro app preview](capmetro.webp)

Shows the next bus or rail arrivals across up to four configured stops — pulled live from the CapMetro GTFS-RT trip updates feed. Choose how many departures to show (1–4); each one gets a color-coded route badge, an arrival ETA ("Due" / "In 8 min" / ">1 hr"), and its stop name.

## Getting started

In the Tidbyt app, set **Stop 1** through **Stop 4** to the numeric CapMetro stop IDs you want to monitor. All four are optional — stop `603` (31st Street Station northbound) is used if you leave Stop 1 blank.

**Departures shown** picks how many arrivals appear at once, 1 to 4. The stops act as one pool: the app collects every upcoming departure across them and shows that many of the soonest, so setting four stops and two departures is a perfectly normal way to use it. The layout adapts to how many departures were actually found, so a quiet stop never leaves blank rows.

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
