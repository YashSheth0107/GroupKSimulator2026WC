# GroupKSimulator2026WC
Monte Carlo simulation modelling Portugal's probability of advancing from the group matches and reaching each knockout stage in the 2026 FIFA World Cup using historical match data.

# Match-outcome model
This is the engine. The standard, well-validated choice for tournament simulation is a Dixon–Coles Poisson model: each team gets an attack and defense strength, you model home/away goals as Poisson, with a low-score correction and a time-decay weighting so recent matches count more. Crucially, it produces scorelines, which is needed for group-stage tiebreakers (points → goal difference → goals scored).

## Data

**International football results (1872–present)**
Jürisoo, M. (2023). *International football results from 1872 to 2026* [Dataset]. Kaggle. https://www.kaggle.com/datasets/martj42/international-football-results-from-1872-to-2017
