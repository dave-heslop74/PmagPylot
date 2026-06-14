# Workflow: Reversal Test / Common Mean Test

## Before starting

Load `functions/policy.md` if not already loaded for this session — it governs how calculations in this workflow should be performed (use real PmagPy functions; do not improvise statistics).

## What this workflow is for

A reversal test (also called a common mean test) asks whether a set of "normal" polarity paleomagnetic directions and a set of "reversed" polarity directions are consistent with sharing a common mean direction, once the reversed population is flipped to its antipode. This is a standard check in paleomagnetic studies: if the directions reliably record an ancient geocentric axial dipole field, normal and reversed populations should be antipodal.

Trigger this workflow when the user asks to:
- "run a reversal test"
- "do a common mean test"
- "compare my normal and reversed directions"
- "check if my directions are antipodal"

## Step 1: Determine polarity assignment

Using the site mean directions loaded per `conventions/data_loading.md`, determine how sites are split into normal and reversed polarity populations.

**Preferred: use a recorded polarity column if present** (e.g. `dir_polarity`, with values such as `'n'`/`'r'`):

```python
normal    = sites[sites['dir_polarity'] == 'n'].copy()
reversed_ = sites[sites['dir_polarity'] == 'r'].copy()
```

**Fallback: inclination-sign split**, only if no recorded polarity column is available:

```python
normal    = sites[sites['dir_inc'] > 0].copy()
reversed_ = sites[sites['dir_inc'] < 0].copy()
```

**Caution for the LLM — this is not a minor detail.** In real data, recorded `dir_polarity` and the inclination-sign heuristic can disagree for *all* sites (observed for one MagIC contribution during development of this knowledge base — see `community/edge_cases.md`). The inclination-sign heuristic implicitly assumes a reference field where normal polarity means steep positive inclination, which does not hold for all paleolatitudes/declinations. If you must use the fallback, **state this explicitly as an assumption** in your response, and note that the resulting polarity assignment has not been independently verified.

If the split (by either method) produces a very small or empty population on one side, do not proceed silently — see Step 3.

## Step 2: Present the available tests

PmagPy offers several approaches. Present these to the user **before** picking one, so they can make an informed choice — unless the user has already specified which test they want. See `functions/statistics_and_tests.md` for full details on each function, including verified signatures and return value conventions.

### Option 1: Watson V test — `ipmag.reversal_test_MM1990()` or `ipmag.common_mean_watson()`

- The standard parametric test following McFadden & McElhinny (1990)
- Computes Watson's V statistic via Monte Carlo simulation and compares it to a critical value
- Returns a classification grade (A, B, C, or indeterminate) based on the angle between the population means
- **Most widely used in the published literature**
- Assumes both populations are Fisher-distributed
- `reversal_test_MM1990` takes a single combined dataset and does its own polarity splitting/flipping; `common_mean_watson` takes two pre-separated populations (use this if polarity was determined via Step 1's recorded-column method, and remember to flip one population to its antipode first)

### Option 2: Heslop et al. (2023) bootstrap test — `ipmag.reversal_test_bootstrap_H23()`

- Non-parametric — makes no distributional assumption
- Extends the earlier Tauxe (2010) bootstrap approach with a proper null-hypothesis-significance-testing framework, yielding an interpretable **p-value**
- **Preferred non-parametric option** — recommended over the older `common_mean_bootstrap` (Tauxe 2010), which has no associated significance level
- Reasonable for smaller N than the Watson test requires, though very small N (≲10) should still be noted when reporting

### Option 3: Bayesian common mean test — `ipmag.common_mean_bayes()`

- Bayesian framework (Heslop & Roberts, 2018) — returns a Bayes factor and posterior probability rather than a p-value
- Useful as a third perspective, particularly for small or borderline datasets
- Not a replacement for Options 1/2, but a valuable cross-check

**Recommendation logic:** If prerequisites for the Watson V test (Step 3) are satisfied, present it as the primary, most widely recognised result, but also run the H23 bootstrap test (Option 2) as a standard cross-check — both are cheap to compute. If Watson V prerequisites fail, lead with Option 2 and explain why. Option 3 can be offered for borderline/small-N cases. If results from different tests disagree, see `functions/statistics_and_tests.md`, "When tests disagree" — report all results, do not silently pick one.

## Step 3: Check prerequisites

Before running the Watson V test, check:

1. **Sample size.** Recommend N ≥ 5 per population as an absolute minimum for Watson V. For the H23 bootstrap test, very small N (≲10) should be noted but does not necessarily preclude running it.
2. **Fisher distribution.** Each population should be reasonably consistent with a Fisher distribution.
3. **Comparable concentration.** The κ (kappa) values of the two populations should not differ wildly — a ratio of less than about 3–4 is generally considered acceptable for Watson V.

```python
normal_fish   = ipmag.fisher_mean(di_block=ipmag.make_di_block(
    normal['dir_dec'].tolist(), normal['dir_inc'].tolist(), unit_vector=False))
reversed_fish = ipmag.fisher_mean(di_block=ipmag.make_di_block(
    reversed_['dir_dec'].tolist(), reversed_['dir_inc'].tolist(), unit_vector=False))

print(f"Normal:   N={normal_fish['n']}, κ={normal_fish['k']:.1f}, α95={normal_fish['alpha95']:.1f}")
print(f"Reversed: N={reversed_fish['n']}, κ={reversed_fish['k']:.1f}, α95={reversed_fish['alpha95']:.1f}")
```

(Note: `ipmag.fisher_mean`, not `pmag.fisher_mean` — both may exist, but prefer the `ipmag` wrapper per `functions/policy.md` / `functions/reference_index.md`.)

### How to report prerequisite results

For each check, report a clear pass/fail/caution status and a brief reason. For example:

- Sample size: pass — "Normal: N=24, Reversed: N=18, both above the minimum of 5 for Watson V."
- κ comparison: pass or caution — "κ ratio ≈ 1.3, within the acceptable range" or "κ ratio ≈ 5.2, which is high — Watson V results should be interpreted cautiously, consider the H23 bootstrap test instead."

### When a prerequisite fails: warn, then ask — do not stop automatically

If a prerequisite fails badly (e.g. one population has N < 5, or N < 3, well below any test's recommended minimum):

1. **State the issue clearly and specifically** — which check failed, the actual numbers, and why it matters (e.g. "with only 3 directions, the Fisher mean for this population is unlikely to be reliable, and any test result built on it should be treated with significant caution").
2. **Explain what this means for available tests** — e.g. if both Watson V's minimum (N≥5) and the bootstrap test's recommended minimum (N≥10) are failed, say so plainly: neither test is well-supported.
3. **Ask the user whether they want to proceed anyway**, with results explicitly caveated as unreliable — rather than either (a) proceeding silently with a misleadingly confident result, or (b) refusing to proceed at all without giving the user the choice.

Example phrasing:

> "Before proceeding: the normal-polarity population has only N=3 sites, which is below the recommended minimum for both the Watson V test (N≥5) and the bootstrap test (N≥10). Any reversal test result here would be statistically unreliable. Would you like me to proceed anyway — with results clearly marked as unreliable due to small N — or would you prefer to stop here, e.g. to investigate why so few sites have this polarity?"

If the user opts to proceed, carry the caveat through Steps 4–6 explicitly (e.g. prefix results with "⚠️ Unreliable due to small N:").

If all prerequisites pass cleanly, proceed to Step 4 directly (after confirming test choice with the user if not already specified).

## Step 4: State hypotheses and run the test(s)

Always state the hypotheses explicitly before presenting results:

- **H₀ (null hypothesis):** The normal and reversed polarity populations share a common mean direction (i.e. are antipodal), consistent with a geocentric axial dipole field.
- **H₁ (alternative hypothesis):** The populations do not share a common mean direction.

### Watson V test

If polarity was determined via the recorded-column method (Step 1, preferred), use `common_mean_watson` with pre-separated populations, flipping the reversed population to its antipode first:

```python
# Flip reversed population to its antipode for comparison
reversed_flipped_dec = (reversed_['dir_dec'] + 180) % 360
reversed_flipped_inc = -reversed_['dir_inc']

normal_di            = ipmag.make_di_block(normal['dir_dec'].tolist(), normal['dir_inc'].tolist(), unit_vector=False)
reversed_flipped_di  = ipmag.make_di_block(reversed_flipped_dec.tolist(), reversed_flipped_inc.tolist(), unit_vector=False)

result, angle, critical_angle, classification = ipmag.common_mean_watson(
    normal_di, reversed_flipped_di, random_seed=42
)
print(f"result={result}, angle={angle:.2f}, critical_angle={critical_angle:.2f}, classification='{classification}'")
```

If polarity was determined via the inclination-sign fallback, `reversal_test_MM1990` can be used instead, passing the *unflipped* combined data (it performs its own splitting and flipping internally):

```python
combined_di = ipmag.make_di_block(
    pd.concat([normal['dir_dec'], reversed_['dir_dec']]).tolist(),
    pd.concat([normal['dir_inc'], reversed_['dir_inc']]).tolist(),
    unit_vector=False
)
result, angle, critical_angle, classification = ipmag.reversal_test_MM1990(
    di_block=combined_di, random_seed=42
)
```

**Result convention for both functions:** `result == 1` → pass (cannot reject common mean); `result == 0` → fail.

### Heslop et al. (2023) bootstrap test (recommended cross-check)

```python
result_h23, Lmin, Lmin_c, p = ipmag.reversal_test_bootstrap_H23(
    di_block=normal_di + reversed_flipped_di,  # or combined_di if using the unflipped form with reversal=True via common_mean_bootstrap_H23
    num_sims=10000, alpha=0.05, plot=False, random_seed=42
)
print(f"result={result_h23}, Lmin={Lmin:.2f}, Lmin_c={Lmin_c:.2f}, p={p:.4f}")
```

**Result convention — note this is the OPPOSITE of the Watson V functions above:** `result_h23 == 1` → fail to reject H₀ (common mean not rejected); `result_h23 == 0` → reject H₀. Always report `p` alongside `result_h23` to avoid ambiguity (see `functions/statistics_and_tests.md`).

## Step 5: Interpret the results

### Decision rules

**Watson V (`reversal_test_MM1990` / `common_mean_watson`):**
- `result == 1` (angle < critical_angle): fail to reject H₀ — consistent with a common mean direction. Report the `classification` grade (see table below).
- `result == 0`: reject H₀ — not consistent with a common mean direction.

**H23 bootstrap (`reversal_test_bootstrap_H23`):**
- `result_h23 == 1` (p ≥ alpha): fail to reject H₀.
- `result_h23 == 0` (p < alpha): reject H₀. Report `p` explicitly.

### Classification grade (McFadden & McElhinny, 1990) — for Watson V results only

| Angle between means | Grade | Meaning |
|---|---|---|
| < 5° | A | Excellent — directions essentially antipodal |
| 5°–10° | B | Good |
| 10°–20° | C | Marginal — interpret with caution |
| Test fails or angle very large | Indeterminate | No meaningful reversal test result |

### Plain-language interpretation templates

**Watson V — Grade A or B (result=1):**

> "The normal and reversed polarity directions are statistically [indistinguishable from / consistent with being] antipodal (Grade [A/B]; angle between means = [X]°, critical angle = [Y]°). This is [strong / reasonable] evidence that the remanent magnetisation reliably records the ancient geomagnetic field, with no significant overprinting or remagnetisation detected by this test."

**Watson V — Grade C (result=1 but marginal):**

> "While the test does not formally reject a common mean direction, the angle between the population means ([X]°) is large enough (Grade C) that this result should be treated as marginal. Consider examining individual site data for outliers, running the H23 bootstrap test as a cross-check, or reporting this result with appropriate caveats."

**Watson V — result=0 (H₀ rejected):**

> "The normal and reversed polarity directions are **not** consistent with a common mean direction (angle between means = [X]° > critical angle = [Y]°). This could indicate: (1) incomplete removal of a secondary magnetisation component (overprint) in one or both populations, (2) a true asymmetry in the recorded field, or (3) data quality issues. Further investigation is recommended before drawing conclusions from these directions."

**H23 bootstrap — result_h23=0 (H₀ rejected, p < alpha):**

> "The Heslop et al. (2023) bootstrap test rejects the null hypothesis of a common mean direction (p = [X], below the α = [Y] threshold). [If this disagrees with a Watson V result obtained above]: note that this differs from the Watson V result — see the discussion of test disagreement below."

### When tests disagree

If Watson V and H23 give different conclusions (e.g. Watson V "fail to reject, Grade C" vs H23 "reject, p<0.05"), do not pick one silently:

> "The two tests give different conclusions: the Watson V test does not reject a common mean direction (though only at the marginal Grade C level), while the Heslop et al. (2023) bootstrap test rejects it (p=[X]). This kind of disagreement is most likely for borderline cases and is itself informative — it suggests the data are not strongly inconsistent with a common mean, but nor do they strongly support one. Consider running the Bayesian common mean test (`common_mean_bayes`) as a third perspective, and examining individual site directions for outliers."

**Important for the LLM:** Always tie the statistical outcome (test statistic, p-value or angle/critical_angle, and grade where applicable) to the plain-language interpretation explicitly — do not give one without the other. Avoid stating a conclusion more strongly than the statistics support (e.g. do not call a Grade C result "strong evidence", and do not ignore a disagreement between tests).

## Step 6: Suggest a citation-ready summary

If results are clear and positive (Grade A/B and/or H23 fails to reject with reasonable p), offer a draft sentence suitable for a methods/results section, e.g.:

> "A reversal test was performed using PmagPy (Tauxe et al., 2016). The Watson V test (McFadden & McElhinny, 1990) yields a positive result at the Grade [A/B] level (angle between means = [X]°, critical angle = [Y]°), and the bootstrap common mean direction test of Heslop et al. (2023) does not reject a common mean direction (p = [Z]), indicating that the normal and reversed polarity populations are consistent with a common mean direction."

For Grade C, marginal, negative, or disagreeing results, draft language that reflects this honestly — e.g. describing the result as "marginal" or noting the disagreement between tests — rather than overstating. A negative or marginal result is itself a meaningful finding worth reporting, not a failure. If small-N caveats applied (Step 3), include them in the draft text.

## Step 7: Suggest next steps

After presenting and interpreting results, offer relevant next steps depending on the outcome, for example:
- If positive: calculating a paleomagnetic pole, or proceeding to a fold test
- If negative, marginal, or disagreeing: examining individual site/specimen data for problems, checking demagnetization data for incomplete component isolation, running `common_mean_bayes` as a third perspective
- Exporting the session script for reproducibility (see below)

## Reproducibility note

Every code block actually executed in this workflow should be retained in the session's running script in the order executed, so that the full analysis can be exported as a reproducible Python script or notebook at the end of the session. Per `functions/policy.md`, this script should consist of real PmagPy function calls — not manual reimplementations — so that it is genuinely reproducible by anyone with the same PmagPy version and input data.
