# Architecture — LILA Player Journey Tool

## What I Built and Why

A single-file static web app (`index.html`) using the HTML Canvas 2D API for rendering, with all data pre-processed and embedded at build time.

**Why this stack:**
The primary user is a Level Designer opening a link in their browser — not a developer running a server. A static file with no backend, no login, no dependencies means zero friction between them and the data. Canvas 2D handles 89k events smoothly without a heavy charting library. The entire tool deploys in seconds on Netlify.

---

## Data Flow

```
out.csv (89,104 rows)
    │
    ▼
preprocess.py
    ├── Parse timestamps, detect bots, categorise events
    ├── Per map × per event type:
    │       bin X/Z into 60×60 grid → normalise to [0,1] → sparse array of (x, y, weight)
    └── Per map × top 8 matches:
            extract all non-position events + sampled positions → (nx, nz, cat, is_bot, t)
    │
    ▼
Inline JSON blob (~267KB) written into index.html
    │
    ▼
Browser loads index.html
    ├── drawBg()      — grid lines, axis labels, map name
    ├── drawHeatmap() — radial gradients from pre-binned grid points
    └── drawEvents()  — dots / paths from per-match event arrays
```

`t` in each event is a normalised timestamp: `(event_ts - match_start_ts) / match_duration`, so the timeline slider at 0–100% maps directly to match progression regardless of absolute timestamp values.

---

## Coordinate Mapping

This was the trickiest part. The data has world-space X/Z coordinates with no minimap image provided, so the approach was:

**1. Compute per-map bounds from the data itself:**
| Map | X range | Z range |
|---|---|---|
| AmbroseValley | −325 → 302 | −380 → 361 |
| Lockdown | −407 → 348 | −285 → 329 |
| GrandRift | −226 → 257 | −194 → 170 |

**2. Normalise to [0, 1]:**
```
nx = (x - x_min) / (x_max - x_min)
nz = (z - z_min) / (z_max - z_min)
```

**3. Map to canvas pixels:**
```
canvas_x = nx × canvas_width
canvas_y = nz × canvas_height
```

**4. Reverse mapping for the live coordinate readout (hover):**
```
world_x = x_min + (canvas_x / canvas_width) × (x_max - x_min)
```

**Assumption:** X is horizontal and Z is the depth/vertical axis. This matches the spatial clustering of events visually — kills and loot concentrate in distinct zones rather than appearing random, which validates the mapping. If the Z-axis is inverted relative to the actual minimap, flipping `nz = 1 - nz` corrects it.

---

## Assumptions Made

| Ambiguity | Assumption Made |
|---|---|
| No minimap images provided | Drew spatial canvas from coordinate bounds with a reference grid |
| Timestamps show epoch 1970-01-21 | Likely a game-internal timer reset to epoch. Used relative `t` within each match (0→1) so absolute dates don't matter for playback |
| Bot detection not documented | `event.startsWith('Bot')` OR `user_id` is a plain integer (e.g. `1409`) vs a UUID. Human players consistently have UUID-format user_ids |
| No explicit "date" filter field | All data falls on a single calendar date; match-level filtering is more meaningful here |
| Parquet mentioned but CSV provided | Treated CSV as canonical. Preprocessing is identical for parquet — swap `pd.read_csv` → `pd.read_parquet` |

---

## Major Tradeoffs

| Decision | Chosen | Alternative | Reason |
|---|---|---|---|
| Architecture | Single static HTML | React + API backend | No server cost, zero setup for end user, faster to ship |
| Rendering | Canvas 2D | WebGL / D3 SVG | Handles 89k events without frame drops; simpler than WebGL for this data scale |
| Data delivery | JSON embedded in HTML at build time | Fetch from API at runtime | Works offline, no CORS, no latency, fine at 300KB |
| Heatmap method | Pre-bin 60×60 grid + radial gradients | Raw KDE per render | Pre-binning makes re-renders instant; per-frame KDE over 50k points is too slow |
| Timeline | Normalised t (0→1) per match | Absolute timestamps | Match durations vary; normalised t makes the slider work consistently across all matches |
| Match selection | Top 8 by event count | All 250+ matches | All matches would push file size to ~5MB+ and slow the browser |
