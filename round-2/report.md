# Round 2 — Investigate

Team: BB-003  
Queries used: 110 / 170

## What we concluded

Our Round 2 investigation identified a strong negative relationship between `comorbidity_ratio` and the GK-02 score. Increasing `comorbidity_ratio` from 0.5 through 1.0 consistently lowered the score, with the size of the decrease varying across the range, indicating a nonlinear effect. We also observed a positive effect of `age` in the tested region, while decreasing `years_registered` increased the score with a changing effect size across the tested range.

## How we got there

We used controlled single-feature changes whenever possible and compared the resulting score with the immediately preceding configuration.

For `comorbidity_ratio`, the score decreased as the value increased:

- 0.5 → 0.6: 0.9316 → 0.9173 (-0.0143)
- 0.6 → 0.7: 0.9173 → 0.8893 (-0.0280)
- 0.7 → 0.8: 0.8893 → 0.8667 (-0.0226)
- 0.8 → 0.9: 0.8667 → 0.8484 (-0.0183)
- 0.9 → 1.0: 0.8484 → 0.8046 (-0.0438)

For `age`, increasing age from 50 to 70 increased the score from 0.8860 to 0.9316, a change of +0.0456.

For `years_registered`, decreasing the value increased the score:

- 40 → 30: 0.9006 → 0.9278 (+0.0272)
- 30 → 20: 0.9278 → 0.9529 (+0.0251)
- 20 → 10: 0.9529 → 0.9594 (+0.0065)

The changing magnitude across the tested range suggests that these effects are not well described by a single constant linear contribution.

## What we ruled out

We did not treat queries where multiple inputs changed simultaneously as clean evidence for an individual feature. Those observations were excluded from the primary feature-effect claims because the contribution of each changed input could not be separated reliably.

We also did not conclude that any untested feature is irrelevant. Round 2 evidence is limited to the controlled relationships described above.

What we are still unsure about

We have not yet recovered an exact mathematical formula for the GK-02 score. The observed relationships are based on sampled points and may include interactions between features. In particular, the strength of the `years_registered` and `age` effects may depend on the values of other inputs, and the exact nonlinear shape of the `comorbidity_ratio` relationship remains unknown.
