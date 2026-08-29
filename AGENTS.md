# AGENTS.md

Guidance for coding agents working in this repository.

**GitHub:** [WorldRover/tidbyt-capmetro](https://github.com/WorldRover/tidbyt-capmetro)

Read [README.md](README.md) first — what the app is, how to run it, how the
render pipeline and the five layouts work, and the feed/reference links all live
there and are not repeated here. This file covers only the conventions an agent
needs that a reader of the README does not.

## Versioning

Semantic Versioning, tagged `vMAJOR.MINOR.PATCH` directly on `main` after the PR
merges. A pixlet `manifest.yaml` has no version field, so the tag and the
topmost released heading in `CHANGELOG.md` are the only source of truth — keep
them in step.

**Cadence — buffer model:** batch small fixes into one minor release rather than
shipping a patch per commit. Cut a release when `## [Unreleased]` has
accumulated something meaningful, or immediately for a breaking change.

## Branches and pull requests

Branch names must match `(feat|fix|refactor|chore|docs|test)/short-slug`, with an
issue number prefix (`fix/20-regenerate-stops`) when there is a tracking issue.
Sync `main` before branching. PRs merge into `main`; no long-lived branches;
delete the branch after merge. Include `Closes #N` when a PR resolves an issue.

**Feature branches never touch `CHANGELOG.md`** — release notes are generated at
release time from PR labels via `.github/release.yml`.

**PR labels:** every PR needs at least one label from the set below before merge.
Labeling the tracking issue is not enough; `--generate-notes` groups by *PR*
label.

## Labels

`ui`, `data`, `infra` · `P1`, `P2`, `P3` · `type: bug`, `type: feature`,
`type: docs`, `type: enhancement`, `type: refactor`. The stock GitHub `bug`,
`enhancement`, and `documentation` labels are removed. `scripts/init-labels.sh`
installs the set.

## Key constants

- `CAPMETRO_TRIP_UPDATES_URL` — the only network call the app makes
- `DEFAULT_STOP` (`603`) — used when Stop 1 is blank
- `BADGE_W` (16) / `ETA_W` (12) / `SPACE_W` (4) — column widths for the badge,
  the right-aligned compact ETA, and the gap before it
- `FALLBACK_ROUTE_COLOR` (`000000`) — badge color for a route `route_colors`
  does not know
- `DOME` — the CapMetro dome icon, a base64 PNG inlined in the source. 7×12
  RGBA, two pixel values only: opaque white and fully transparent. It is drawn
  over the route's color bar, so it must stay white-on-transparent — a white
  background would show as a box. Used in the no-service screens and above the
  route number in `dome_bar()`.

## Render functions

`chip()` 16px badge · `dome_bar()` full-height colored bar · `tall_lines()` the
two-line text column · `compact_row()` badge + name + right-aligned
`eta_short()` · `stop_label()` name lookup with the `Stop <id>` fallback.

## Gotchas

- `route_colors` (71 entries) and `stops` (2348) are inlined dicts at the top of
  `capmetro.star`, generated from the GTFS static feed. There are no data files
  in the repo; regenerating them is manual today (issue #20).
- The trip updates feed encodes `arrival.time` as a **string**. Parse with
  `int()` before comparing.
- Starlark has no `list.sort()` — use `sorted()`.
- Do not name a function `schema()`; it shadows the `schema` module. The schema
  entry point is `get_schema()`.
- Cache absolute arrival times, never relative minutes, or ETAs go stale.
- The service alerts feed contains CapMetro staff names and email addresses.
  Never display or log them.
- The repo holds no credentials. `pixlet login` stores them outside the project;
  `$TIDBYT_API_TOKEN` is the scripted fallback. Never write a token into a file
  in this repo, and never pass one on a command line you'd leave in history.
