# An NBA Squad Under the Salary Cap: Stats First, Salaries as the Constraint

Can a 15-man squad built only from player ratings and contract data fit under the salary cap and still be
clearly stronger than real NBA teams? And how much of that edge survives when the squad is tested on the
**next** season?

*Third project in a series. Earlier projects: [Finding Similar Players](https://github.com/thesixthman0-sports/scout-analytics-football) and [Scouting by the Numbers: Under-23 Players](https://github.com/thesixthman0-sports/u23-scouting).*

## Summary

- Squads were picked with **season N** ratings (BPM) and **season N+1** contracts, then scored with the players'
  **actual season N+1** ratings. Two season pairs: 2024 to 2025 and 2025 to 2026.
- Rules: salary cap, a 10-player rotation with two players per position, plus five depth places at near-minimum
  salary; at most 2 all-stars, 3 rookie-scale contracts and 2 max-like contracts.
- **In a simplified model, the optimised squads beat the best real team in all 10 tested set-ups** (five minutes
  templates, two season pairs), by 2.5 to 7.9 points of team strength. They also beat greedy selection of the best
  BPM players by 4 to 6 points, and every one of 2,000 random squads under the same rules.
- **The picked squads fell short of expectations:** averaged over the four base runs, expected strength was
  26.9 and realised strength was 21.5 (about 20% lower). Selecting on one season picks some lucky seasons.
- **The edge depends on a few underpaid stars.** In the 2025 to 2026 test, removing the two best players
  (Jokić and Gilgeous-Alexander) leaves 15.9, the same as the best real team (15.6) and greedy selection (15.6).
- **Money has diminishing returns:** raising the budget from the cap to the luxury-tax line (+17 to +29M)
  added only 1.5 to 2.9 points.

These are results of a simplified model (see Limitations), not a prediction of what a real team could do.

## Data

Files are not included in this repository. Download them and place them in one folder:

- Kaggle [`sumitrodatta/nba-aba-baa-stats`](https://www.kaggle.com/datasets/sumitrodatta/nba-aba-baa-stats)
  (Basketball-Reference season data): `Advanced.csv`, `Player Season Info.csv`, `All-Star Selections.csv`,
  `Draft Pick History.csv`
- Kaggle [`deocheng/nba-data-1946-2026`](https://www.kaggle.com/datasets/deocheng/nba-data-1946-2026):
  `public.player_salaries_historical.csv`, `public.player_contracts.csv`

Seasons are named by the year they end (2025 means 2024/25). Salaries for 2025 come from the historical salary
table and salaries for 2026 from the contracts table. Earlier salaries in the historical table did not match
known contracts, so seasons before 2025 were not used.

## Method

1. **Rating:** BPM (Box Plus/Minus), the estimated points per 100 possessions a player adds over an average player.
   Players need 1,000+ minutes in season N. Main position is the first label in the position field.
2. **Team strength** = sum over players of BPM × minutes / 48. In the base case the starters play 36 minutes and
   the bench 12. In each position the better of the two chosen players (by season N BPM) starts.
3. **Squad rules:** total salary at most the cap of season N+1 (including five depth places at the 10th percentile
   salary of the season); at most 2 all-stars of season N; at most 3 rookie-scale contracts (first-round pick in the
   first four seasons, an approximation); at most 2 max-like contracts (salary of 25% of the cap or more).
4. **Usage constraint:** the minutes-weighted usage of the five players on the court is kept at or below 100,
   as in a real lineup. Without it, five high-usage stars can be picked, and their BPM does not simply add up.
5. **Optimisation:** mixed-integer program (SciPy `milp`).
6. **Evaluation:** season N+1 BPM of the chosen players. Players who play fewer than 500 minutes in season N+1
   get replacement-level BPM (-2.0).
7. **Comparisons:** real teams evaluated with the same minutes template (their ten players with the most minutes),
   2,000 random squads under the same rules, and greedy selection.

## Results

### 1. Base case: 36 / 12 minutes

| Season pair | Usage limit | Expected | Realised | Payroll (M) | Best real team | Median real team |
|---|---|---|---|---|---|---|
| 2024 to 2025 | no | 26.2 | 20.3 | 138.9 | 16.2 | -0.1 |
| 2024 to 2025 | yes | 23.8 | 24.1 | 140.3 | 16.2 | -0.1 |
| 2025 to 2026 | no | 29.9 | 19.3 | 151.4 | 15.6 | 2.8 |
| 2025 to 2026 | yes | 27.6 | 22.3 | 154.5 | 15.6 | 2.8 |

Expected uses season N ratings, realised uses season N+1 ratings.
The real-team columns use the same 36 / 12 minutes template as the squads (their ten players with the most minutes). The first cell of the notebook prints real-team strength with actual minutes, which gives lower values (best team 13.9 and 11.9) because real players miss games.

### 2. Comparisons with simpler approaches (36 / 12 minutes)

| | 2024 to 2025 | 2025 to 2026 |
|---|---|---|
| Random squads under the same rules (median, 90th percentile; no usage limit) | 1.0, 6.1 | -0.7, 4.7 |
| Greedy selection of the best BPM players (expected, realised) | 20.0, 14.0 | 22.3, 15.6 |
| Optimiser, no usage limit (realised) | 20.3 | 19.3 |
| Optimiser, usage limit (realised) | 24.1 | 22.3 |
| Optimiser, usage limit, without the best player | 22.0 | 18.3 |
| Optimiser, usage limit, without the two best players | 22.0* | 15.9 |

\* In the first pair the second removed player was not in the squad, so the result did not change.

### 3. Minutes templates and budget (usage limit on)

| Minutes template | 2024 to 2025: optimiser / best real team | 2025 to 2026: optimiser / best real team |
|---|---|---|
| 36 / 12 | 24.1 / 16.2 | 22.3 / 15.6 |
| 34 / 14 | 22.6 / 16.0 | 20.3 / 15.3 |
| 32 / 16 | 21.8 / 15.7 | 17.5 / 15.0 |
| Descending ladder (36, 34, ... 8) | 23.8 / 16.4 | 22.8 / 15.5 |
| 30 / 14 / 4 per position (15 players) | 20.9 / 14.4 | 17.5 / 13.8 |
| Budget up to the luxury-tax line, 36 / 12 | 27.0 (payroll 169.7M) | 23.8 (payroll 171.9M) |

### 4. The base-case squads (usage limit on)

2024 to 2025 (payroll 140.3M, of which 9.5M is the depth places):

| Role | Player | Position | Salary (M) | BPM, season N | BPM, season N+1 |
|---|---|---|---|---|---|
| Starter | Nikola Jokić | C | 51.4 | 13.2 | 13.3 |
| Starter | Trayce Jackson-Davis | PF | 1.9 | 2.8 | 1.2 |
| Starter | Shai Gilgeous-Alexander | PG | 35.9 | 9.0 | 11.5 |
| Starter | Trey Murphy III | SF | 5.2 | 2.5 | 0.8 |
| Starter | Alex Caruso | SG | 9.9 | 2.5 | 4.0 |
| Bench | Al Horford | C | 9.5 | 3.6 | 1.2 |
| Bench | Nicolas Batum | PF | 4.7 | -0.4 | 0.8 |
| Bench | Dyson Daniels | PG | 6.1 | 0.6 | 1.5 |
| Bench | Sam Hauser | SF | 2.1 | 1.2 | 0.8 |
| Bench | Nickeil Alexander-Walker | SG | 4.3 | 0.2 | -0.4 |

2025 to 2026 (payroll 154.5M, of which 10.0M is the depth places):

| Role | Player | Position | Salary (M) | BPM, season N | BPM, season N+1 |
|---|---|---|---|---|---|
| Starter | Nikola Jokić | C | 55.2 | 13.3 | 14.2 |
| Starter | Nicolas Batum | PF | 5.6 | 0.8 | -1.0 |
| Starter | Shai Gilgeous-Alexander | PG | 38.3 | 11.5 | 11.7 |
| Starter | Amen Thompson | SF | 9.7 | 4.1 | 2.6 |
| Starter | Alex Caruso | SG | 18.1 | 4.0 | 1.9 |
| Bench | Jalen Duren | C | 6.5 | 3.1 | 5.0 |
| Bench | Tari Eason | PF | 5.7 | 3.6 | -1.6 |
| Bench | Mike Conley | PG | 0.7 | 1.2 | -1.3 |
| Bench | Justin Champagnie | SF | 2.3 | -0.5 | -0.4 |
| Bench | Keon Ellis | SG | 2.3 | 1.9 | -0.5 |

## What this shows

- Paying attention to contracts matters: the same ratings, used greedily, give a clearly weaker squad than an
  optimisation that treats salary as a budget.
- The gain comes mostly from two very good players whose salaries are below their value, surrounded by cheap role
  players. Without those stars the squad is no better than a good real team.
- Ratings of one season overstate what the same players deliver the next season, here by about 20%.
- Concentrating minutes on the starters raises the result (36/12 beats 34/14 and 32/16 in both tests), which
  is also what makes the model optimistic about star-heavy squads.

## Limitations

- **BPM is not additive.** Players share the ball and the court. The usage limit corrects this only partly, and
  the model cannot see fit or spacing.
- **Every player is assumed to be healthy** and to play fixed minutes all season. Real stars miss games.
  Players with fewer than 500 minutes in the test season get replacement-level BPM.
- **Availability:** the squad is picked from the whole league, ignoring trades, team needs and whether a
  player could be acquired. The salary cap in the NBA is also soft, and real teams often go above it.
- **Two season pairs only,** and one season of ratings per pair. Cap and tax-line values for 2025 were entered by hand.
- **Rookie-scale and max-like contracts are approximations** based on draft round, years in the league
  and 25% of the cap.
- **Data:** open and scraped data, not official records. Salaries before 2025 were excluded because they did
  not match known contracts. Players without a known salary and two-way contracts are excluded.

## Files

- [`nba_cap_squad.ipynb`](nba_cap_squad.ipynb): the full analysis, run top to bottom in Google Colab.

## Data attribution

Season statistics from Basketball-Reference, via the Kaggle dataset by sumitrodatta. Contract and salary tables from
the Kaggle dataset by deocheng. Please check the original terms of the sources before reusing the data.
