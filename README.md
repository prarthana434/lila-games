# LILA Player Journey Visualization Tool

A browser-based tool for LILA Games' Level Design team to visually explore how players move, fight, loot, and die across maps using real production telemetry data from LILA BLACK.

**Live URL:** [Click here to view the project](https://taupe-baklava-efbb9c.netlify.app/)

## What It Does

- **Heatmap overlays** for kills, deaths, loot pickups, movement density, and storm deaths — each as a separately toggleable layer
- **Events view** — scatter plot of individual events, colour-coded by type and bot vs human
- **Paths view** — traces player movement trails per match
- **Timeline playback** — scrub or animate through a match chronologically
- **Per-match filtering** — select any of the top 8 matches per map
- **Bot / Human toggle** — show or hide each population independently
- **Live coordinate readout** — hover anywhere to see real-world X/Z game coordinates
- **Event feed** — live log of what happened at the current timeline position

## Tech Stack

| Layer | Choice | Why |
|---|---|---|
| Frontend | Vanilla HTML + Canvas 2D | Zero build step, instant deploy, fast canvas rendering for 89k events |
| Data pipeline | Python (pandas, numpy) | Fast coordinate binning and JSON serialisation |
| Hosting | Netlify | Static file — drag and drop, live in 30 seconds |
| Fonts | Space Grotesk + JetBrains Mono | Readable at small sizes, fits the game analytics aesthetic |

## Setup

### Run locally
Open `index.html` in any browser. No server needed.

```bash
# Optional local server
python3 -m http.server 8080
# then open http://localhost:8080
```

### Regenerate from source data
If you have new telemetry, re-run the preprocessing script to rebuild `index.html`:

```bash
pip install pandas numpy
python3 preprocess.py
```

`preprocess.py` reads `out.csv`, bins coordinates into a 60×60 heatmap grid per map per event type, extracts timeline events for the top 8 matches per map, and writes everything as an inline JSON blob into the HTML template.

## Environment Variables
None. Everything is self-contained in `index.html`.

## Repo Structure

```
/
├── index.html        # Complete tool — HTML + embedded data + JS (~300KB)
├── preprocess.py     # Data pipeline: CSV → heatmap grids → embedded JSON
├── out.csv           # Source telemetry (89,104 events, 3 maps, 5 days)
├── README.md
├── ARCHITECTURE.md
└── INSIGHTS.md
```
