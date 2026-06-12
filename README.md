# GroupKSimulator2026WC - Overview
Monte Carlo simulation modelling Portugal's probability of advancing from the group matches and reaching each knockout stage in the 2026 FIFA World Cup using historical match data.

# Phase 1 - Data cleaning / feature prep
- Load results.csv, parse date to datetime.
- Reconcile names via former_names.csv — map each former → current only within its [start_date, end_date] window, so historical matches attach to the right modern team.
- Keep a wide era (e.g. 1990+) but rely on time-decay weighting (Phase 2) rather than a hard cutoff to down-weight old games.
- Add a neutral flag (already in the data) — you'll zero out home advantage for neutral matches, which is what World Cup games effectively are.
- Don't filter to just Portugal. The model needs the whole network of matches to estimate every team's strength on a common scale.
First sanity check: Portugal's win rate over the last ~5 years, goals for/against. Make sure name reconciliation didn't drop matches.

# Phase 2 - Match-outcome model: Dixon–Coles Poisson model
The model gives each team an attack strength αᵢ and defense strength βᵢ, plus a global home advantage γ:

Expected home goals: λ = exp(α_home − β_away + γ)
Expected away goals: μ = exp(α_away − β_home)
Goals ~ Poisson, with the Dixon–Coles τ correction that inflates/deflates the four low-score cells (0-0, 1-0, 0-1, 1-1) to fix Poisson's underestimation of draws.
Two things make it tournament-grade:

Time decay: weight each match by exp(−ξ · Δt). Tune ξ so recent form dominates without throwing away signal. ξ is your main hyperparameter.
Neutral-venue handling: set γ = 0 for neutral matches so WC predictions aren't biased by home advantage.
Fit by maximizing the time-weighted log-likelihood with scipy.optimize.minimize (L-BFGS-B), constraining mean attack = 0 for identifiability.

Validate before trusting it: hold out the last ~12 months, score predictions with log-loss / RPS against a baseline.

Output: a function score_matrix(home, away, neutral=True) → a matrix of P(home_goals=i, away_goals=j). Everything downstream draws from this.

# Phase 3 -  Monte Carlo loop
- Define Group K: Portugal + 3 opponents.
- One simulation = sample a scoreline for all 6 group matches from the score matrices.
- Build the standings: 3/1/0 points, then goal difference → goals scored as tiebreakers (these come for free because we simulate scorelines, not just W/D/L).
- Record whether Portugal finishes 1st, 2nd, or 3rd.
- Repeat N=50k–100k times; aggregate to P(1st), P(2nd), P(top-2).
The third-place wrinkle: in 2026, 8 of the 12 third-placed teams also advance. Since we're doing group-only first, model that with a simplified rule — e.g. estimate a points/GD threshold a third-place team typically needs to be a top-8 third, and report P(advance as best third | finishes 3rd) as a separate, clearly-labeled approximation.

# Phase 4 - Report
Probability table (1st / 2nd / 3rd / advance), a bar chart, etc.

## Data
**International football results (1872–present)**
Jürisoo, M. (2023). *International football results from 1872 to 2026* [Dataset]. Kaggle. https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017
