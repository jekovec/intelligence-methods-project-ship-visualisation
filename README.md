# Ship voyages 1750–1850: visualising the CLIWOC logbooks

Master's project for the Intelligence Methods course. We build two visualisations from **CLIWOC**, a database of weather observations transcribed from ship logbooks: about 287k daily records from Dutch, British, Spanish and French ships, mostly 1750–1850 ([source](https://historicalclimatology.com/cliwoc), [field definitions](https://stvno.github.io/page/cliwoc/)).

**Status (2 Nov 2026):** data explored; idea B (sailing performance) in progress; idea A (monthly wind atlas) planned.

## Quick start

```bash
# 1. Get the data (160 MB, not in git)
mkdir -p data && curl -L -o data/CLIWOC21.tsv "https://media.githubusercontent.com/media/stvno/stvno.github.io/master/page/cliwoc/CLIWOC21.tsv"

# 2. Python 3.11+ with pandas, numpy, matplotlib and Jupyter
#    (we use pyenv's 3.11.9, see .python-version)
python3 -m pip install pandas numpy matplotlib jupyter

# 3. Open the notebooks and run them in order:
#    01 writes data/cliwoc_clean.pkl, which 02 reads
python3 -m jupyter lab
```

## What is where

```
data/                            not in git: CLIWOC21.tsv (download) and cliwoc_clean.pkl (made by notebook 01)
notebooks/
  01_inspect_data.ipynb          what is in the data: coverage, quality, biases; saves the clean subset
  02_sailing_performance.ipynb   idea B: how fast ships sailed depending on the wind
pyproject.toml, .python-version  Python version and dependencies
main.py                          placeholder from `uv init`, not used
```

Each notebook explains its sections in markdown cells and ends with a summary of findings.

### Notebook 01: first look at the data

| Section | What it does |
|---|---|
| 1. Raw file | all 180 columns and how full they are (41 are completely empty) |
| 2. Clean subset | keeps ~30 useful columns, fixes broken rows, adds wind features (Beaufort force, u/v wind vectors) |
| 3. Who is in the data | records per year by nationality, year × month coverage |
| 4. Where | map of all observations, one map per nationality |
| 5. Voyages | voyage length, most common routes, whether positions are daily, daily distance |
| 6. Wind | completeness, Beaufort by nationality, wind roses, monsoon teaser |
| 7. Instruments and text | pressure/temperature coverage, original wind wording per language |
| 8. Save | writes `data/cliwoc_clean.pkl` |

### Notebook 02: sailing performance (idea B)

| Section | What it does | Key result / decision |
|---|---|---|
| 1. One record per voyage per day | removes duplicate records | some logbook IDs hold several ships, so the voyage key includes the ship name |
| 2. Day-to-day steps | from two consecutive noon positions: distance (nm), course (°), speed (knots), plus the wind at both noons | 250k steps; median speed ≈ 4 knots |
| 3. Wind angle vs speed | angle between the wind and the ship's course (0° = wind from ahead, 180° = from behind) | speed rises from ~1.6 to ~5.2 knots; the mean of both wind readings fits best |
| 4. Position quality | was each position observed, dead-reckoned or interpolated? | interpolated steps excluded; the measurement type itself makes little difference within a voyage |
| Whole voyages | 7 nearly gap-free voyages: track + speed vs wind, coloured by position quality | good example voyages for the presentation |

## The data: what to watch out for

- **It's a sample of surviving logbooks,** not of all ships at sea. Who is in it changes over time (British logs end in 1829, after 1830 almost only Dutch), so trends over time are mixed with changes in nationality.
- **Wind direction** (`D`, degrees the wind blows *from*, true north) is reliable and in 96% of the records. `361` = calm, `362` = variable.
- **Wind force** (`W`, in 0.1 m/s) was translated from words like "fresh gales" or "bramzeilskoelte", and each language was translated differently. Compare it between nationalities with care.
- **Positions** are noon positions, either observed (sun/stars), dead-reckoned (calculated from logged speed and course) or interpolated by the people who transcribed the logbooks (`LatInd` / `LonInd`). Before ~1800 most longitudes were dead reckoning.

## The two ideas

- **B. Sailing performance (in progress).** Compute each ship's daily speed and course from consecutive positions, compare with the logged wind, and draw how speed depends on wind angle and force (a sailing "polar diagram"). Then look at ship types, nations and decades, and later fit a model (regression → small neural network). Course topics: feature engineering, perceptrons/MLP.
- **A. Monthly wind atlas + PCA/SVD (next).** Average the wind vectors per month in 5° cells and show them as 12 maps. Then decompose the space × month matrix with SVD (in climate science called EOF analysis); the leading pattern should be the seasonal cycle, including the Indian Ocean monsoon reversal. Course topics: PCA/SVD.

Ideas we dropped because the data is too thin: Australia (≈1,000 records nearby), the "year without a summer" 1816 in the English Channel (23 records), storm case studies (too few ships close together).

## Next steps

1. Add wind force to idea B: heatmap of wind angle × Beaufort, then a polar diagram with one line per wind band.
2. Check the wind-force translation per nationality using ship speed.
3. Performance index (speed relative to what is typical for that wind) by ship type, nationality and decade.
4. Start idea A in a new notebook.
