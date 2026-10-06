# Round 1 — Observe

**Team:** BB-003  
**Queries used:** 66 / 150

## What we concluded

Our black box was **GK-02**, a synthetic hospital admission triage system returning a score between 0 and 1 and an APPROVE/DECLINE decision.

From controlled experiments, we observed:

- **age** increases the score strongly.
- **baseline_score** increases the score strongly.
- **prior_visits** increases the score strongly.
- **comorbidity_ratio** decreases the score strongly.
- **recent_admissions** decreases the score strongly.
- **years_registered** decreases the score moderately.
- **ward** has only a small effect under the tested baseline configuration.
- **dependents**, **requested_beds**, and **vitals_index** showed no observable effect in our baseline tests.

Our best observed score during Round 1 was **0.9711 (APPROVE)**.

## How we got there

We started with a baseline configuration:

**Score = 0.6612, APPROVE**

We then varied individual numerical inputs while keeping the remaining baseline values fixed whenever possible.

### Age

- Age 18 → **0.5053**
- Age 46.5 → **0.6612**
- Age 75 → **0.7714**

This showed a strong positive relationship between age and score.

### Baseline score

- baseline_score 300 → **0.4509**
- baseline_score 600 → **0.6612**
- baseline_score 900 → **0.7264**

Increasing baseline_score increased the score, although the change was not perfectly linear across the tested points.

### Comorbidity ratio

- comorbidity_ratio 0 → **0.8347**
- comorbidity_ratio 0.5 → **0.6612**
- comorbidity_ratio 1 → **0.4475**

This was one of the strongest observed effects.

We then investigated the APPROVE/DECLINE transition at the high end. Under the baseline configuration:

- 0.975 → **0.4612, APPROVE**
- 0.975195 → **0.4612, APPROVE**
- approximately 0.97539 → **0.4593, DECLINE**
- 0.978125 → **0.4567, DECLINE**
- 0.9875 → **0.4529, DECLINE**

Therefore, the transition under this particular baseline configuration was located very close to **0.975–0.976**.

### Prior visits

- prior_visits 0 → **0.4897**
- prior_visits 10 → **0.6612**
- prior_visits 20 → **0.7951**

This showed a strong positive effect.

### Recent admissions

- recent_admissions 0 → **0.7567**
- recent_admissions 2.5 → **0.6612**
- recent_admissions 5 → **0.5726**

This showed a clear negative effect.

### Years registered

- years_registered 0 → **0.7552**
- years_registered 20 → **0.6612**
- years_registered 40 → **0.5881**

This showed a moderate negative effect.

### Dependents

Changing dependents from 0 to 3 to 6 produced the same observed score:

**0.6612**

This suggests no observable effect under the tested configuration.

### Requested beds

Changing requested_beds from 0 to 50 to 100 produced:

**0.6612**

No observable effect was detected in these tests.

### Vitals index

Changing vitals_index from 0 to 50 to 100 produced:

**0.6612**

No observable effect was detected in these tests.

### Ward

With the other baseline values held constant:

- Ward A → **0.6612**
- Ward B → **0.6602**
- Ward C → **0.6522**
- Ward D → **0.6604**

The differences were small compared with the major numerical features.

We also explored combinations of favorable variables. The best observed result during our investigation reached **0.9711 (APPROVE)**, showing that combined feature settings can produce a substantially higher score than the baseline.

## What we ruled out

We ruled out the idea that all input fields have equally strong effects.

Our experiments showed that several fields had large and repeatable effects, while other fields remained unchanged across the tested range under the baseline configuration.

We also found that random querying is inefficient compared with changing one variable at a time and then narrowing interesting regions.

## What we are still unsure about

We cannot determine the exact internal model, training data, preprocessing pipeline, feature transformations, or complete scoring formula from our observations alone.

The features that showed no observable effect may still have conditional or interaction effects outside the configurations we tested.

The decision boundary identified for comorbidity_ratio is specific to the tested baseline configuration and should not be assumed to be a universal threshold.

We also observed that changing multiple favorable variables together can substantially alter the score, so interaction and nonlinear effects remain possible.
