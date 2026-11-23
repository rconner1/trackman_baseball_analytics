# Expected Total Bases from TrackMan Batted-Ball Data

Predicting how many total bases a batted ball produces (0–4) from TrackMan ball-flight measurements, and
testing how much of that prediction comes from quality of contact alone.

**Main finding:** A random forest using only exit speed, launch angle and direction (RMSE 0.730) nearly matched
one that also knows where the ball landed (RMSE 0.724). Once you know how a ball was hit, its distance and hang
time add very little.

## Data

TrackMan batted-ball data (`trackman.xlsx`). `PlayResult` was converted to total bases: single = 1,
double = 2, triple = 3, home run = 4, everything else = 0. Rows with missing tracking data were removed.

Predictors: `ExitSpeed`, `Angle` (launch angle), `Direction`, `HitSpinRate`, `Distance`, `HangTime`.

## Approach

- 70/30 train/test split (seeded for reproducibility)
- Predictor selection with nested linear models and ANOVA
- Polynomial degree selection with ANOVA for each predictor
- Interaction models for pairs of predictors
- Two random forests: all six measurements, and contact only (`ExitSpeed`, `Angle`, `Direction`)

Distance and hang time describe where the ball ended up, so they partly reveal the result (a 400-foot fly ball is
almost always a home run). The contact-only model follows the same logic as MLB's expected statistics, which use
contact measurements for this reason.

## Results (test set)

| Model | RMSE (bases) | Test R² |
| --- | --- | --- |
| Random Forest (all six) | 0.7241 | 0.504 |
| Random Forest (contact only) | 0.7298 | 0.496 |
| Polynomial | 0.7934 | 0.405 |
| Exit Speed × Distance | 0.8959 | 0.241 |
| Linear | 0.9290 | 0.184 |
| Distance × Angle | 0.9394 | 0.165 |
| Exit Speed × Angle | 0.9628 | 0.123 |
| Empty (training mean) | 1.0289 | −0.001 |

Variable importance: in the full model, distance was the most important predictor. In the contact-only model,
exit speed and launch angle were nearly tied, followed by direction.

## Limitations

- Total bases is treated as a continuous outcome; a model of the probability of each outcome would be a natural next step.
- Results come from a single train/test split; cross-validation would give a more reliable comparison.
- The data has no information on fielder positioning, ballpark or batter speed.

## Files

| File | Description |
| --- | --- |
| `Baseball_Analytics_Code.Rmd` | Full analysis: cleaning, EDA, models, evaluation |
| `Baseball_Analytics_Paper.pdf` | Written report of the methods and results |
| `trackman.xlsx` | TrackMan batted-ball data |

## How to run

Put `trackman.xlsx` in the same folder as the `.Rmd`, then knit it in RStudio. Requires the R packages
`readxl`, `tidyverse`, `randomForest` and `knitr`:

```r
install.packages(c("readxl", "tidyverse", "randomForest", "knitr"))
```

Knitting also saves the two fitted models as `rf_full_model.rds` and `rf_contact_model.rds`.
