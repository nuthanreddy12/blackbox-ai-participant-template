# Round 4 — GK-02 Reconstruction

## Team

- **Team ID:** BB-003
- **Black box:** GK-02
- **System type:** Synthetic hospital admission triage system

## 1. Objective

The goal of Round 4 was to reconstruct the observed GK-02 black-box behavior from the query/response data collected during the earlier rounds and produce a surrogate model that can estimate the continuous score and reproduce the APPROVE/DECLINE decision.

The observed GK-02 interface uses these inputs:

- `vitals_index` (0–100)
- `age` (18–75)
- `years_registered` (0–40)
- `comorbidity_ratio` (0–1)
- `baseline_score` (300–900)
- `prior_visits` (0–20)
- `recent_admissions` (0–5)
- `requested_beds` (0–100)
- `dependants` (0–6)
- `ward` (A/B/C/D)

The output is a score in the range 0–1 and a binary decision, APPROVE or DECLINE.

## 2. Investigation carried into reconstruction

The reconstruction used the query observations collected in R1/R2. The exported dataset used for the final reconstruction contains **150 observations**.

Several relationships were identified during probing:

### Comorbidity

`comorbidity_ratio` has a clear nonlinear negative effect. In one controlled sequence, the observed score changed as follows:

| Comorbidity | Score |
|---:|---:|
| 0.5 | 0.9316 |
| 0.6 | 0.9173 |
| 0.7 | 0.8893 |
| 0.8 | 0.8667 |
| 0.9 | 0.8484 |
| 1.0 | 0.8046 |

This ruled out a simple constant linear penalty as a complete explanation.

### Age

Age contributes positively in the high-score region observed during investigation. Under the controlled comparison used in Round 2, changing age from 50 to 70 increased the observed score from **0.8860 to 0.9316**.

### Years registered

`years_registered` showed a nonlinear downward effect in the tested Ward B configuration. In the observed sequence, increasing years registered from 10 to 40 reduced the score, with the magnitude of the change varying by range.

### Ward

Ward has a relatively small but observable categorical effect. Near otherwise-matched high-score configurations, the observed scores differed by ward; Ward D was lower than Ward B in the tested comparison.

### Decision boundary

The Round 1 boundary probes placed the decision transition near **0.46**. The reconstruction therefore treats the decision as a threshold applied to the predicted continuous score rather than as an independent classification mechanism.

## 3. Reconstruction method

The final surrogate is a **degree-2 polynomial Ridge regression model**.

Pipeline:

1. Median-impute numeric features.
2. Standardize numeric features.
3. Generate degree-2 polynomial interactions for numeric inputs.
4. One-hot encode the categorical `ward` feature.
5. Fit Ridge regression with `alpha=2.0`.
6. Clip predicted scores to the valid `[0, 1]` interval.
7. Convert the predicted score to a decision using an empirically optimized threshold.

The threshold was searched over the observed decision-boundary region using the out-of-fold predictions from 5-fold cross-validation.

## 4. Validation

The final model was evaluated using **5-fold cross-validated out-of-fold predictions** on the 150-observation export.

Final measured results:

- **R²:** `0.998662`
- **MAE:** `0.001804`
- **Decision accuracy (0–1 form):** `0.9933`
- **Decision accuracy (% form):** `99.33%`
- **Correct decisions:** `149 / 150`
- **Wrong decisions:** `1 / 150`
- **Optimized decision threshold:** `0.4589`

The score metric is kept in the black box's native 0–1 scale. The decision-accuracy metric is reported both as a decimal and percentage for clarity.

## 5. Model selection

Several surrogate approaches were compared under the same 5-fold evaluation procedure. On the exported data, the degree-2 Ridge model produced the strongest combination of score reconstruction and decision reproduction.

The final model was selected because it reproduced the continuous score closely while also achieving **0.9933 cross-validated decision accuracy**.

## 6. Prediction behavior

For each new GK-02 input row, the reconstruction returns:

- a continuous predicted score between 0 and 1; and
- `APPROVE` when the predicted score is at or above `0.4589`, otherwise `DECLINE`.

Example from the observed data:

- observed score: `0.9789`
- reconstructed score: approximately `0.9776`
- reconstructed decision: `APPROVE`

## 7. Limitations

The training data is an observational sample collected through black-box probing, not a random sample of the full hidden rule space. The dataset is also strongly concentrated in APPROVE outcomes. The reported 0.9933 decision accuracy therefore describes reconstruction performance on the available observed dataset under 5-fold cross-validation; it is not a claim of perfect generalization to every unseen input.

The model deliberately does not use the observed `score` or `decision` as input features, preventing direct target leakage.

## 8. Files

- `gk02_reconstruction_colab.ipynb` — Google Colab reconstruction notebook
- `findings.json` — key reconstruction findings and model claims
