# Workflow: Conglomerate Test

## Before starting

Load `functions/policy.md` if not already loaded for this session — it governs how calculations in this workflow should be performed (use real PmagPy functions; do not improvise statistics).

## What this workflow is for

The conglomerate test asks whether a set of paleomagnetic directions measured from clasts within a conglomerate (or breccia) are randomly oriented — as would be expected if each clast was magnetised prior to deposition and then tumbled randomly during transport. If the directions are random, this is strong evidence that the remanent magnetisation in the clasts is pre-depositional (primary), and therefore ancient. If the directions are clustered rather than random, they may record a post-depositional remagnetisation event, or the clasts may not have been adequately randomised during deposition.

The test is named for this geological application but is formally a test of randomness (uniformity) of a set of directional data on the sphere, using the Watson (1956) criterion based on the resultant vector length R.

Trigger this workflow when the user asks to:
- "run a conglomerate test"
- "test if my clast directions are random"
- "check whether conglomerate clasts were remagnetised"
- "do a Watson randomness test on directions"

## Step 1: Clarify the geological context

Before loading data or running any test, confirm the following with the user if not already stated:

1. **What kind of clasts are being tested?** The conglomerate test is most meaningful for clasts of a lithology known (or expected) to carry a stable remanence — e.g. igneous clasts, or clasts derived from a unit whose magnetic properties are understood.
2. **Are the clasts from a true conglomerate or breccia?** The test assumes clasts were tumbled and deposited randomly — this is less defensible for in-situ breccias, debris flows with preferred fabric, or any deposit where clast orientations may have been partially constrained during emplacement.
3. **Has a tilt correction been applied?** For the conglomerate test, directions from clasts should generally be used in their *in situ* (geographic) orientation — the premise of the test is that the clasts were randomly reoriented during deposition, so no tectonic correction is applied to the clasts themselves. If the user is uncertain, ask before proceeding.

State any assumptions arising from this step clearly in all subsequent responses.

## Step 2: Load the data and present the test

Load the clast directions from the MagIC data per `conventions/data_loading.md`. The directions should be at the specimen or sample level (one direction per clast), not site means.

```python
import pandas as pd
import pmagpy.ipmag as ipmag
import pmagpy.pmag as pmag

# Load directions — adjust path and column names to match the uploaded MagIC zip
# Clast directions are typically in the specimens or samples table
specimens = pd.read_csv('specimens.txt', sep='\t', header=1)

# Filter to the relevant site/location if needed
clasts = specimens[specimens['site'] == 'YOUR_SITE'].copy()

# Extract dec/inc pairs — column names may vary; common options are
# 'dir_dec'/'dir_inc' or 'measurement_dec'/'measurement_inc'
dec = clasts['dir_dec'].tolist()
inc = clasts['dir_inc'].tolist()
n   = len(dec)
print(f"Number of clast directions: {n}")
```

Confirm the number of directions and column names with the user before proceeding. If column names differ from the above, inspect the available columns:

```python
print(clasts.columns.tolist())
```

### The available test

PmagPy provides one implementation of the conglomerate test:

**`ipmag.conglomerate_test_Watson(R, n)`** — Watson (1956) test of randomness

- Compares the resultant vector length R of the clast directions against tabulated critical values for the Watson (1956) test
- If R is **less than** the critical value Ro, the null hypothesis of randomness is not rejected — consistent with pre-depositional (primary) magnetisation
- If R **exceeds** Ro, the null hypothesis is rejected — directions are significantly clustered, suggesting post-depositional remagnetisation or inadequate randomisation of clasts during deposition
- **Note:** this is a classical critical-value test. There is currently no bootstrap conglomerate test implemented in PmagPy. For small N (see Step 3), results should be interpreted with additional caution, and this limitation should be noted when reporting

The function requires R and n to be computed first, using `pmag.fisher_mean()`, which is the required upstream step — do not pass R values computed by any other means.

## Step 3: Check prerequisites

Before running the test, check:

1. **Sample size.** Watson (1956) critical values are tabulated for specific N ranges. A minimum of N = 5 is required for the test to be meaningful; N ≥ 20 is preferable for reliable inference. Very small N (< 10) should always be flagged when reporting.
2. **Independence of directions.** Each direction should represent a separate clast. Multiple specimens from the same clast are not independent and should not be included as separate directions — confirm with the user if this is unclear.
3. **No within-clast averaging.** If multiple measurements per clast are available, confirm which level (specimen, sample) the directions represent.

```python
# Compute Fisher mean statistics — this gives us R and n
di_block   = ipmag.make_di_block(dec, inc, unit_vector=False)
fish       = pmag.fisher_mean(di_block)

R          = fish['r']
n          = fish['n']
k          = fish['k']
alpha95    = fish['alpha95']
mean_dec   = fish['dec']
mean_inc   = fish['inc']

print(f"N={n}, R={R:.3f}, κ={k:.1f}, α95={alpha95:.1f}°")
print(f"Mean direction: Dec={mean_dec:.1f}°, Inc={mean_inc:.1f}°")
```

### How to report prerequisite results

For each check, report a clear pass/caution/fail status and a brief reason. For example:

- Sample size: pass — "N=28 clast directions, above the recommended minimum of 20 for reliable inference."
- Sample size: caution — "N=8 clast directions, above the absolute minimum of 5 but below the preferred minimum of 20. Results should be interpreted cautiously."
- Independence: pass — "Each direction represents a separate clast per the MagIC metadata."

### When a prerequisite fails: warn, then ask — do not stop automatically

If a prerequisite fails badly (e.g. N < 5):

1. **State the issue clearly and specifically** — which check failed, the actual numbers, and why it matters.
2. **Explain what this means for the test** — e.g. "with only 4 directions, the Watson (1956) critical values are not well-defined and any result should be treated as essentially uninformative."
3. **Ask the user whether they want to proceed anyway**, with results explicitly caveated as unreliable, rather than either proceeding silently or refusing to proceed without giving the user the choice.

Example phrasing:

> "Before proceeding: only N=4 clast directions are available, which is below the minimum of 5 required for the Watson (1956) conglomerate test to be meaningful. Any result here would be statistically unreliable. Would you like me to proceed anyway — with results clearly marked as unreliable due to small N — or would you prefer to stop here, e.g. to check whether additional clasts are available in the dataset?"

If the user opts to proceed, carry the caveat through Steps 4–5 explicitly (e.g. prefix results with "⚠️ Unreliable due to small N:").

If all prerequisites pass cleanly, proceed to Step 4 directly.

## Step 4: State hypotheses and run the test

Always state the hypotheses explicitly before presenting results:

- **H₀ (null hypothesis):** The clast directions are randomly (uniformly) distributed on the sphere — consistent with pre-depositional magnetisation and adequate randomisation of clasts during transport and deposition.
- **H₁ (alternative hypothesis):** The clast directions are significantly clustered — inconsistent with random orientation, suggesting post-depositional remagnetisation or non-random clast deposition.

```python
# Run the Watson (1956) conglomerate test
# R and n must come from pmag.fisher_mean() computed in Step 3
test_result = ipmag.conglomerate_test_Watson(R, n)
print(test_result)
```

The function prints its result directly and returns a dictionary containing the Watson (1956) critical values (Ro) for this N. Capture this for reporting:

```python
# The returned dictionary contains the critical value(s) for the given n
# Print the full result for inspection
print(f"\nR = {R:.4f}")
print(f"Watson (1956) test result: {test_result}")
```

**Important:** `ipmag.conglomerate_test_Watson` requires R and n from `pmag.fisher_mean()` only — do not substitute R values computed manually or from other sources, per `functions/policy.md`.

## Step 5: Interpret the results

### Decision rule

The Watson (1956) test compares R to a critical value Ro:

- **R < Ro (null hypothesis not rejected):** the directions are consistent with being randomly distributed — a **positive** conglomerate test result. This supports the interpretation that the remanent magnetisation in the clasts is pre-depositional (primary).
- **R ≥ Ro (null hypothesis rejected):** the directions are significantly clustered — a **negative** conglomerate test result. This suggests post-depositional remagnetisation, or that clasts were not adequately randomised during deposition.

The significance level used by `ipmag.conglomerate_test_Watson` follows Watson (1956) — report the relevant critical level (typically 95%) alongside R and Ro.

### Plain-language interpretation templates

**Positive result (R < Ro — null hypothesis not rejected):**

> "The Watson (1956) conglomerate test gives a positive result: the clast directions are consistent with being randomly distributed on the sphere (R = [X], Ro = [Y] at the 95% confidence level; N = [n]). This supports the interpretation that the remanent magnetisation in these clasts is pre-depositional, recorded prior to their incorporation into the conglomerate."

**Negative result (R ≥ Ro — null hypothesis rejected):**

> "The Watson (1956) conglomerate test gives a negative result: the clast directions are significantly clustered rather than randomly distributed (R = [X] ≥ Ro = [Y] at the 95% confidence level; N = [n]). This is inconsistent with pre-depositional (primary) magnetisation of the clasts, and suggests either post-depositional remagnetisation of the conglomerate, or that clasts were not adequately randomised during transport and deposition."

**Small-N caution (applies regardless of test outcome):**

> "Note: this result is based on only N = [n] clast directions. The Watson (1956) critical values are less reliable at small N, and the result should be treated with additional caution. There is currently no bootstrap implementation of the conglomerate test in PmagPy, which would be more appropriate for small or non-uniformly distributed samples."

### Limitation to note when reporting

The Watson (1956) conglomerate test assumes the null distribution of R under perfect randomness — it does not account for any anisotropy in the sampling of clast orientations (e.g. if large clasts tend to lie flat in the deposit). If there is reason to suspect non-random clast geometry in the deposit, note this limitation when reporting, and consider whether an independent sedimentological assessment of clast fabric is warranted.

## Step 6: Suggest a citation-ready summary

If the result is positive and prerequisites were satisfied, offer a draft sentence suitable for a methods/results section, e.g.:

> "A conglomerate test was performed on [n] clasts from [unit/location] using PmagPy (Tauxe et al., 2016). The Watson (1956) randomness test gives a positive result (R = [X], Ro = [Y]; N = [n]), indicating that clast directions are consistent with random orientation and supporting a pre-depositional origin for the remanent magnetisation."

For negative results or cases with small-N or geological caveats, draft language that reflects these honestly — e.g. describing the result as negative, or noting limitations — rather than overstating. A negative conglomerate test is itself an important finding, not a failure.

## Step 7: Suggest next steps

After presenting and interpreting results, offer relevant next steps depending on the outcome:

- **If positive:** the conglomerate test supports primary magnetisation of the clasts. Suggest this result be combined with other field tests (fold test, reversal test) to build a comprehensive case for palaeomagnetic reliability.
- **If negative:** suggest examining whether post-depositional remagnetisation of the host conglomerate is consistent with other observations (e.g. similar directions in clasts and matrix, overprinting temperatures from thermal demagnetisation). Consider whether the clast lithology or depositional setting makes adequate randomisation plausible.
- **If borderline or small N:** suggest collecting additional clast directions if field access permits, and note that this would be the single most effective way to improve the reliability of the test.
- Exporting the session script for reproducibility (see below).

## Reproducibility note

Every code block actually executed in this workflow should be retained in the session's running script in the order executed, so that the full analysis can be exported as a reproducible Python script or notebook at the end of the session. Per `functions/policy.md`, this script should consist of real PmagPy function calls — not manual reimplementations — so that it is genuinely reproducible by anyone with the same PmagPy version and input data.
