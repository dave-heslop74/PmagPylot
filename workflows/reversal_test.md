# Workflow: Reversal Test / Common Mean Test

## What this workflow is for

A reversal test (also called a common mean test) asks whether a set of "normal" polarity paleomagnetic directions and a set of "reversed" polarity directions are consistent with sharing a common mean direction, once the reversed population is flipped to its antipode. This is a standard check in paleomagnetic studies: if the directions reliably record an ancient geocentric axial dipole field, normal and reversed populations should be antipodal.

Trigger this workflow when the user asks to:
- "run a reversal test"
- "do a common mean test"
- "compare my normal and reversed directions"
- "check if my directions are antipodal"

## Step 1: Separate the data by polarity

Using the site mean directions loaded per `conventions/data_loading.md`, separate sites into normal and reversed polarity populations. A simple convention (appropriate for many studies, but state this assumption to the user):

```python
normal    = sites[sites['dir_inc'] > 0].copy()
reversed_ = sites[sites['dir_inc'] < 0].copy()
```

**Caution for the LLM:** This simple inclination-sign split is a reasonable default for many datasets but is not universally correct — for studies near the equator, or where there is a strong overprint, polarity assignment may already be recorded in the data (e.g. a `dir_polarity` column or method code) and that should be preferred if present. If the split produces a very small or empty population on one side, flag this to the user rather than proceeding silently.

## Step 2: Present the available tests

PmagPy offers three approaches. Present these to the user **before** picking one, so they can make an informed choice — unless the user has already specified which test they want.

### Option 1: Watson V test — `ipmag.reversal_test_MM1990()`

- The standard parametric test following McFadden & McElhinny (1990)
- Computes Watson's V statistic and compares it to a critical value
- Returns a classification grade (A, B, C, or Indeterminate) based on the angle between the population means
- **Most widely used in the published literature**
- Assumes both populations are Fisher-distributed

### Option 2: Bootstrap common mean test — `ipmag.common_mean_bootstrap()`

- Non-parametric — makes no distributional assumption
- Resamples the data to build confidence regions for each mean direction and checks for overlap
- More robust for scattered or non-ideal data
- Recommended minimum N ≈ 10 per population
- Slower to compute than the Watson test

### Option 3: Parametric common mean test — `ipmag.common_mean_parametric()`

- An alternative parametric approach
- Less commonly cited in the literature than the Watson V test
- Useful as a cross-check alongside Option 1

**Recommendation logic:** If prerequisites for the Watson V test (Step 3) are satisfied, recommend Option 1 as the primary result, since it is the most widely recognised in the literature. If prerequisites fail, recommend Option 2. Option 3 can be offered as an optional cross-check but should not be the default recommendation.

## Step 3: Check prerequisites for the Watson V test

Before running the Watson V test, check:

1. **Sample size.** Recommend N ≥ 5 per population as an absolute minimum; note that larger N gives more reliable results.
2. **Fisher distribution.** Each population should be consistent with a Fisher distribution. A Rayleigh test can be used as an approximate check.
3. **Comparable concentration.** The κ (kappa) values of the two populations should not differ wildly — a ratio of less than about 3–4 is generally considered acceptable. Large differences in κ can affect the reliability of the Watson V critical value.

```python
import numpy as np

normal_fish   = pmag.fisher_mean(
    ipmag.make_di_block(normal['dir_dec'], normal['dir_inc']))
reversed_fish = pmag.fisher_mean(
    ipmag.make_di_block(reversed_['dir_dec'], reversed_['dir_inc']))

print(f"Normal:   N={normal_fish['n']}, κ={normal_fish['k']:.1f}, α95={normal_fish['alpha95']:.1f}")
print(f"Reversed: N={reversed_fish['n']}, κ={reversed_fish['k']:.1f}, α95={reversed_fish['alpha95']:.1f}")

# Approximate Rayleigh test for each population
for label, fish in [('Normal', normal_fish), ('Reversed', reversed_fish)]:
    R = fish['r']; N = fish['n']
    p_val = np.exp(-2 * N * (R / N) ** 2)
    print(f"{label} Rayleigh p = {p_val:.3f}")
```

### How to report prerequisite results

For each check, report a clear pass/fail/caution status and a brief reason. For example:

- Sample size: pass — "Normal: N=24, Reversed: N=18, both above the minimum of 5."
- Fisher distribution: pass — "Both populations show significant clustering (Rayleigh p < 0.05), consistent with a Fisher distribution."
- κ comparison: pass or caution — "κ ratio ≈ 1.3, within the acceptable range" or "κ ratio ≈ 5.2, which is high — results should be interpreted cautiously" or recommend the bootstrap test instead.

If all prerequisites pass, proceed to Step 4 with the Watson V test (after confirming with the user). If prerequisites fail or are marginal, explain why and suggest the bootstrap common mean test as an alternative, explaining the tradeoff (no distributional assumption, but requires larger N and is slower).

## Step 4: State hypotheses and run the test

Always state the hypotheses explicitly before presenting results:

- **H₀ (null hypothesis):** The normal and reversed polarity populations share a common mean direction (i.e. are antipodal), consistent with a geocentric axial dipole field.
- **H₁ (alternative hypothesis):** The populations do not share a common mean direction.

```python
normal_di   = ipmag.make_di_block(normal['dir_dec'],   normal['dir_inc'])
reversed_di = ipmag.make_di_block(reversed_['dir_dec'], reversed_['dir_inc'])

result = ipmag.reversal_test_MM1990(
    di_block    = normal_di + reversed_di,
    plot_CDF    = True,
    plot_stereo = True
)
```

The function returns Watson's V statistic, the critical value V_crit, the angle between the two population means, and the critical angle.

## Step 5: Interpret the results

### Decision rule

- If **V < V_crit**: fail to reject H₀. The data are consistent with the normal and reversed populations sharing a common mean direction.
- If **V > V_crit**: reject H₀. The populations do not share a common mean direction at the tested confidence level.

### Classification grade (McFadden & McElhinny, 1990)

Based on the angle between the population means (γ) compared to the critical angle:

| Angle between means | Grade | Meaning |
|---|---|---|
| < 5° | A | Excellent — directions essentially antipodal |
| 5°–10° | B | Good |
| 10°–20° | C | Marginal — interpret with caution |
| Test fails or angle very large | Indeterminate | No meaningful reversal test result |

### Plain-language interpretation template

When V < V_crit and the result is Grade A or B:

> "The normal and reversed polarity directions are statistically [indistinguishable from / consistent with being] antipodal (Grade [A/B]). This is [strong / reasonable] evidence that the remanent magnetisation reliably records the ancient geomagnetic field, with no significant overprinting or remagnetisation detected by this test."

When V < V_crit but the result is Grade C:

> "While the test does not formally reject a common mean direction, the angle between the population means (γ = [X]°) is large enough (Grade C) that this result should be treated as marginal. Consider examining individual site data for outliers, or report this result with appropriate caveats."

When V > V_crit (H₀ rejected):

> "The normal and reversed polarity directions are **not** consistent with a common mean direction (V = [X] > V_crit = [Y]). This could indicate: (1) incomplete removal of a secondary magnetisation component (overprint) in one or both populations, (2) a true asymmetry in the recorded field (e.g. non-dipole contributions), or (3) data quality issues such as poor site mean determinations. Further investigation — for example examining Zijderveld diagrams for individual specimens, or checking demagnetization intervals — is recommended before drawing conclusions from these directions."

**Important for the LLM:** Always tie the statistical outcome (V vs V_crit, the grade) to the plain-language interpretation explicitly — do not give one without the other. Avoid stating a conclusion more strongly than the statistics support (e.g. do not call a Grade C result "strong evidence").

## Step 6: Suggest a citation-ready summary

If the result is Grade A or B (positive reversal test), offer a draft sentence suitable for a methods/results section, e.g.:

> "A reversal test following McFadden & McElhinny (1990), implemented in PmagPy (Tauxe et al., 2016), yields a positive result at the Grade [A/B] level (Watson's V = [X], V_crit = [Y]; angle between means = [Z]°), indicating that the normal and reversed polarity populations are consistent with a common mean direction."

Adjust language appropriately for Grade C or negative results — avoid overstating the result, and note that a negative or marginal result is itself a meaningful finding worth reporting, not a failure.

## Step 7: Suggest next steps

After presenting and interpreting results, offer relevant next steps depending on the outcome, for example:
- If positive: calculating a paleomagnetic pole, or proceeding to a fold test
- If negative or marginal: examining individual site/specimen data for problems, checking demagnetization data for incomplete component isolation
- Exporting the session script for reproducibility (see below)

## Reproducibility note

Every code block in this workflow should be retained in the session's running script in the order executed, so that the full analysis can be exported as a reproducible Python script or notebook at the end of the session.
