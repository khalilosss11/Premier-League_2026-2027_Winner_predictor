# Premier-League_2026_Winner_predictor

Predicts the probability that each club wins the Premier League, from match results and bookmaker odds, and evaluates the method with a walk-forward backtest over 17 past seasons.

## Method

1. **Match model.** A multinomial logistic regression (with standardized features) predicts home win / draw / away win for a fixture. Features are computed only from information available before kickoff:
   - Elo rating difference (K = 20, home advantage 60, 25% regression to the mean between seasons)
   - Form over each team's last 5 matches: points, goals for/against, shots on target for/against
   - "Market form": expected points implied by bookmaker odds over each team's last 10 *past* matches
2. **Title simulation.** From a chosen point in the season, the remaining fixtures are simulated thousands of times (Monte Carlo) using the match probabilities, starting from the real points table. The title probability is the share of simulations a club finishes first.

Data: [football-data.co.uk](https://www.football-data.co.uk/) English Premier League files, seasons 2005-06 to 2025-26, plus the current season for the live prediction. The data is downloaded by the script and is not stored in this repository.

## Evaluation (walk-forward backtest)

For each test season (2009-10 to 2025-26, 17 seasons) the model is trained only on earlier seasons (4 to 20 seasons of history), then the title race is simulated from match 50 and from match 100 of that season.

| Simulating from | Model's favourite = champion | Team leading the table = champion | Mean probability given to the champion | Mean log score |
|---|---|---|---|---|
| Match 50 | 7 / 17 (41%) | 6 / 17 (35%) | 0.397 | -1.489 |
| Match 100 | 10 / 17 (59%) | 8 / 17 (47%) | 0.481 | -1.292 |

Notes:
- The model beats "follow the table", but by a small margin that is within the noise of 17 seasons.
- The log score is dominated by Leicester 2015-16, where the model gave the champion almost no probability.
- At match level, bookmaker odds alone were better than the model in an earlier 8-season run (log loss 0.959 vs 0.974 for the model).

## Live prediction

Snapshot after 50 of 380 matches of 2026-27 (1 October 2026):

| Club | Title probability |
|---|---|
| Manchester City | 51.2% |
| Arsenal | 44.8% |
| Liverpool | 2.2% |
| Manchester United | 0.8% |
| Brighton | 0.7% |

Early in the season the model is overconfident: at match 50 its favourites averaged roughly 56% while winning the title 41% of the time in the backtest. Treat these as rough estimates. Dated weekly snapshots go in `predictions/`.

## Limitations

- No injury, lineup, transfer or manager information.
- Promoted clubs start from a default rating.
- 17 test seasons is a small sample; it cannot show that a small improvement is real.
- Surprise champions (Leicester 2015-16, Chelsea 2016-17) were missed.

## Repository structure

```
pl-title-prediction/
├── pl_title.py          # data loading, features, backtest, live prediction
├── requirements.txt     # python dependencies
├── predictions/         # dated weekly snapshots of the live prediction
└── README.md
```

## Usage

```bash
pip install -r requirements.txt
python pl_title.py backtest    # match-level + title backtests (a few minutes)
python pl_title.py live        # refresh current-season data, print title probabilities
python pl_title.py both        # backtest, then the live prediction
```

In Google Colab or Jupyter, paste `pl_title.py` into one cell, set `COMMAND` near the top (`"backtest"`, `"live"` or `"both"`) and run the cell.

`live` writes `live_title_probs.csv`. To keep a record, copy it to `predictions/live_YYYY-MM-DD.csv`.

## Data and credits

Match results and odds come from football-data.co.uk. Check their terms before redistributing the data files.

## Author

<Your name> - <course / institution>
