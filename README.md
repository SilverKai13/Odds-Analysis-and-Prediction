# Odds Analysis and Prediction

An exploration of a sports-betting dataset, and an honest attempt to
predict match outcomes from the bookmaker odds. The short version: on this
particular data, you can't and the interesting part is showing exactly
why, rather than pretending the numbers say something they don't.

## The data

`sports_predictive_analysis.csv` covers 1,369 matches across five sports
(basketball, baseball, tennis, hockey, football), with odds for each
outcome, a predicted winner, and the actual winner. It comes from a public
[Kaggle dataset](https://www.kaggle.com/datasets/pratyushpuri/sports-betting-predictive-analysis-dataset),
and it's **synthetic** — the team names are computer-generated ("Gonzalezmouth
Tigers", "Lake Samantha Eagles"), and, as this project shows, the odds
don't actually have any real relationship to the results.

The data file itself is now included in this repo, so the notebook can run
right away on a fresh copy. (Previously it referenced a file that wasn't
included, so the notebook couldn't actually be run — that's fixed.)

## What the notebook does

1. Basic exploration — how many matches per sport, how the odds are
   distributed, average odds per sport, which teams won the most.
2. Reframes the prediction problem to ask a question that's actually
   answerable (see below).
3. Checks directly whether the odds contain any useful information at all.
4. Trains six different prediction methods, compared against a simple
   baseline, using two different scoring measures.

## The mistake that mattered

The original version of this notebook tried to predict the exact winning
team — out of hundreds of different team names. That's the wrong question
to ask:

- It's a problem with hundreds of possible answers, where most teams only
  appear in the data a handful of times, so it's close to unlearnable. The
  original attempt scored **3 to 9% accuracy** — barely better than
  guessing at random among hundreds of options.
- On top of that, there was a coding mistake: team names were converted
  into numbers separately for "home team," "away team," and "winner,"
  which meant the same team could end up with three different numbers
  depending on its role. The model couldn't even represent the basic idea
  of "the home team won."

The right question for a dataset like this is simply: did the home team
win, the away team win, or was it a draw? That's a much simpler,
answerable question, and the odds are the natural information to use to
answer it.

## Results

Asking the right question, and using the odds as the input:

| model | accuracy | how confidently wrong it was |
|---|---:|---:|
| Decision Tree | 45.99% | very (18.95) |
| Support Vector Machine | 45.62% | not much (0.97) |
| Random Forest | 45.26% | not much (1.13) |
| **Baseline (always guess "home win")** | **44.89%** | — |
| Naive Bayes | 43.80% | not much (0.99) |
| K-Nearest Neighbors | 43.43% | somewhat (2.93) |
| Logistic Regression | 41.24% | not much (0.99) |

("How confidently wrong it was" is a technical measure called log-loss.
Lower is better — it doesn't just check whether a model's top guess was
right, it also penalizes a model heavily for being very sure about a wrong
answer.)

Nothing here meaningfully beats just guessing "home win" every time, which
needs no data or model at all. Only three approaches edge above that
baseline, and by less than a percentage point. The Decision Tree is a
useful cautionary example: it has the highest raw accuracy, but a very
poor score on the "confidently wrong" measure — meaning when it's wrong,
it's wrong with a lot of unearned confidence. Accuracy alone would have
hidden that; the second measure catches it. That's exactly why relying on
accuracy alone can be misleading.

## Why the odds carry no useful signal

Checked directly in the notebook:

- The average odds for the home team are essentially identical whether the
  home team wins (3.056) or doesn't (3.061). If odds meant anything, a team
  that's actually about to win should generally have shorter (lower) odds
  than one that's about to lose. Here, there's no difference at all.
- The team with better ("favourite") odds only wins 43.1% of the time —
  barely different from the 45.1% base rate for home teams winning.
  Betting on the favourite is no better than just always picking the home
  team.

The odds in this particular dataset were generated without any real
connection to the outcomes, so there's genuinely nothing here to learn.
That's the honest conclusion: you can't predict these results from these
odds, because the odds are just noise. Asking the right question doesn't
rescue the prediction — nothing could, on this data — but it does show
clearly where the ceiling is and why it's there, which is the actual point
of doing this exercise properly.

## Doing this on real data

If you want odds that actually carry information, use a real historical
source, and evaluate the results properly against what the bookmakers
themselves implied. The companion project,
[Bundesliga-Match-Analysis-and-Prediction](https://github.com/SilverKai13/Bundesliga-Match-Analysis-and-Prediction),
does exactly that from start to finish, using real football data and real
bookmaker odds.

## Running it

```bash
pip install -r requirements.txt
jupyter notebook oddsAI.ipynb
```
