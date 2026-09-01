# Zetamac Ledger

A single-file score tracker for [Zetamac](https://arithmetic.zetamac.com/) arithmetic drills —
log every run, and watch the trend rather than the last score.

## What it tracks

- **Current form** — rolling average over your last 3/5/10 runs, with a sparkline and the
  delta against the same window earlier.
- **Target progress** — your goal against current form, plus a projection of how many more
  runs (and roughly how many days) it takes at your current rate of improvement.
- **Stat tiles** — personal best, last-10 average vs the 10 before, spread of the last 10
  (consistency), seconds per problem, runs and days logged, practice streak, total problems
  solved and minutes drilled, and points gained per 10 runs.
- **Charts** — every run as a point with the rolling average through it (x-axis toggles
  between run number and calendar date), a score distribution histogram, and a practice
  heatmap.
- **Run log** — full table with inline editing, delete, and CSV export.

## Using it

Type a score into the tape at the top and press Enter. Before you commit, a preview line
shows the pace, what the score does to your rolling average, whether it's a personal best,
and how far it leaves you from your goal. The panel at the bottom bulk-imports past scores,
one per line:

```
54
58, 2026-08-03
61, 2026-08-05 19:30
62, 2026-08-14, first try after a break
```

Zetamac scores are only comparable within one game configuration, so each config is tracked
separately and the charts filter by it. The standard interview-prep setup — 120 seconds,
addition 2–100, multiplication 2–12 × 2–100, and both inverses — is preloaded; add others
under **Game settings**.

## How it stores data

`zetamac-ledger.html` is self-contained: no build step, no dependencies, no backend. Runs
are kept in a JSON block inside the file itself:

```html
<script id="log" type="application/json">{ ... }</script>
```

Published as a Claude Artifact, the page saves new versions of itself, so scores persist
across sessions and devices. Opened as a plain local file it still works, falling back to
`localStorage` on that browser.

**This means the committed file carries whatever runs were logged when it was last synced.**
If you edit the page locally and republish, pull the live data block across first, or you'll
overwrite your history.

## Notes

The benchmark scale on the target card is informal community folklore, not a hiring cutoff.
Scores only compare against themselves — the useful signal is the slope of your own line.
