# Odds Analysis and Prediction

Exploratory analysis of a sports-betting dataset, and an honest attempt to
predict match outcomes from bookmaker odds. The short version: on this
data, you can't — and the interesting part is showing exactly why, and not
pretending otherwise.

## The data

`sports_predictive_analysis.csv` — 1,369 matches across five sports
(Basketball, Baseball, Tennis, Hockey, Football), with home/away/draw odds,
a provided `Predicted_Winner`, and the `Actual_Winner`. It comes from the
[Sports Betting Predictive Analysis Dataset 2025](https://www.kaggle.com/datasets/pratyushpuri/sports-betting-predictive-analysis-dataset)
on Kaggle, and it is **synthetic** — the team names are procedurally
generated ("Gonzalezmouth Tigers", "Lake Samantha Eagles"), and, as the
notebook shows, the odds bear no real relationship to the results.

The CSV is committed to this repo now, so the notebook runs on a clean
clone. (The original version referenced a dataset that wasn't included,
so it couldn't actually be run — that's fixed.)

## What the notebook does

1. Standard EDA — match counts by sport, odds distributions, average odds
   per sport, most successful teams.
2. Reframes the prediction problem correctly (see below).
3. Checks whether the odds contain any signal at all.
4. Trains six classifiers with an honest majority-class baseline, scored on
   accuracy **and** log-loss.

## The bug that mattered

The original notebook tried to predict `Actual_Winner` — *which specific
team* won, out of hundreds of distinct team names. That's the wrong target:

- It's a several-hundred-class problem where most teams appear only a few
  times, so it's close to unlearnable. The original models scored **3–9%**.
- The label encoding was broken on top of that: `Home_Team`, `Away_Team`,
  and `Actual_Winner` were each encoded with a *separate* `LabelEncoder`,
  so a team's integer as "winner" didn't match its integer as "home"/"away"
  team. The model couldn't even represent "the home team won."

The right question for a betting dataset is the **outcome**: home win (H),
away win (A), or draw (D). That's a clean 3-class problem, and the odds are
the natural predictors.

## Results

Reframed as H/A/D outcome prediction, using the odds as features:

| model | accuracy | log-loss |
|---|---:|---:|
| Decision Tree | 45.99% | 18.95 |
| Support Vector Machine | 45.62% | 0.97 |
| Random Forest | 45.26% | 1.13 |
| **Baseline (always predict "home win")** | **44.89%** | — |
| Naive Bayes | 43.80% | 0.99 |
| K-Nearest Neighbors | 43.43% | 2.93 |
| Logistic Regression | 41.24% | 0.99 |

Nothing meaningfully beats the base rate. "Always guess home win" (44.89%)
needs no data at all, and only three models edge above it, by less than a
percentage point. The Decision Tree is a nice trap: it has the highest
accuracy (45.99%) but a log-loss of 18.95 — it's confidently *wrong*, which
accuracy alone hides and log-loss exposes. This is exactly why accuracy is
a weak metric for probabilistic prediction.

## Why the odds carry no signal

Directly checked in the notebook:

- Mean home-team odds when the home team **wins**: 3.056. When it **doesn't
  win**: 3.061. Essentially identical — the odds don't move with the result.
- The "favourite" (lower odds) wins 43.1% of the time, which is basically
  the home-win base rate (45.1%) — picking the favourite is no better than
  picking home every time.

The odds in this synthetic dataset were generated without a real link to
the outcomes, so there's no signal to learn. That's the honest conclusion:
you cannot predict these results from these odds, because the odds are
noise. Reframing the problem correctly doesn't rescue the prediction —
nothing can on this data — but it does reveal the real ceiling and *why*
it's there, which is the actual point.

## Doing this on real data

If you want odds that actually mean something, use a real historical source
(e.g. football-data.co.uk) and evaluate with log-loss against the de-vigged
bookmaker probabilities. The companion project
[Bundesliga-Match-Analysis-and-Prediction](https://github.com/SilverKai13/Bundesliga-Match-Analysis-and-Prediction)
does exactly that, end to end.

## Running it

```bash
pip install -r requirements.txt
jupyter notebook oddsAI.ipynb
```
