# Workflow: Fold Test

## Before starting

Load `functions/policy.md` if not already loaded for this session — it governs how calculations in this workflow should be performed (use real PmagPy functions; do not improvise statistics).

## What this workflow is for

The fold test asks whether paleomagnetic directions become more tightly clustered as bedding is progressively "unfolded" back to horizontal — as would be expected if the remanence was acquired before folding (a pre-folding, primary magnetisation). If clustering is maximised at or near 100% unfolding, this is strong evidence that the remanence pre-dates the deformation. If clustering is maximised at 0% unfolding (in situ), it suggests a post-folding remanence. If clustering peaks somewhere between 0% and 100% (partial unfolding), this may indicate a syn-folding remanence or a mixture of components.

The test implemented in PmagPy is the **Tauxe and Watson (1994) bootstrap fold test**, which uses eigenvalue analysis of the orientation matrix to measure clustering, and bootstrap resampling to construct confidence bounds. It requires both paleomagnetic directions *and* bedding orientations (dip direction and dip) for each site or sample, and generates graphical output (plots) rather than a simple pass/fail number — interpreting the plots is a central part of this workflow.

Trigger this workflow when the user asks to:
- "run a fold test"
- "test whether my directions are pre- or post-folding"
- "do a Tauxe-Watson fold test"
- "check whether remanence was acquired before or after folding"
- "bootstrap fold test"

## Step 1: Clarify the geological and structural context

Before loading data or running the test, confirm the following with the user if not already stated:

1. **Is there genuine structural variation in bedding orientation across the dataset?** The fold test requires a range of bedding dips and orientations — it cannot distinguish pre- from post-folding remanence if all sites have the same (or very similar) bedding attitude. A minimum spread of dips (ideally spanning at least ~30–40°) is needed for the test to be meaningful. If the user's sites are all from a single limb of a fold or a gently tilted sequence, note this limitation upfront.
2. **Are bedding orientations measured and recorded?** The test requires `dip_direction` and `dip` for each site or sample. If the user's MagIC data does not include these columns, the test cannot be run — ask the user to confirm bedding data is available before proceeding.
3. **Which coordinate system are the paleomagnetic directions in?** The fold test should be run on directions in **geographic (in situ) coordinates**, not tilt-corrected coordinates. If the user's data is already in tilt-corrected coordinates, note this and ask for clarification — running the fold test on already-corrected directions would be circular.
4. **What level of the MagIC hierarchy should be used?** The fold test can be run on site mean directions (most common), sample directions, or specimen directions depending on what's available. Confirm the level with the user and note which level is being used in all subsequent steps.

State any assumptions arising from this step clearly in all subsequent responses.

## Step 2: Load the data

Load directions and bedding orientations from the MagIC data per `conventions/data_loading.md`. The fold test requires *both* paleomagnetic directions *and* bedding orientation for the same level (site, sample, or specimen) — rows missing either are not usable.

```python
import pandas as pd
import numpy as np
import pmagpy.ipmag as ipmag

# Load the sites table — adjust path as needed
sites = pd.read_csv('sites.txt', sep='\t', skiprows=1)

# Inspect available columns
print(sites.columns.tolist())
print(f"Total rows: {len(sites)}")
```

The key columns needed for the fold test are:

| Column | Meaning |
|---|---|
| `dir_dec` | Declination of the paleomagnetic direction (degrees) |
| `dir_inc` | Inclination of the paleomagnetic direction (degrees) |
| `bed_dip_direction` | Dip direction of bedding (degrees, 0–360) |
| `bed_dip` | Dip of bedding (degrees, 0–90) |

Check that bedding columns are present and populated before proceeding:

```python
required_cols = ['dir_dec', 'dir_inc', 'bed_dip_direction', 'bed_dip']
missing = [c for c in required_cols if c not in sites.columns]
if missing:
    print(f"Missing columns: {missing}")
else:
    # Drop rows with any missing values in required columns
    fold_data = sites[required_cols].dropna()
    print(f"Sites with complete data for fold test: {len(fold_data)} of {len(sites)}")
    print(fold_data.describe())
```

If bedding columns are missing or mostly empty, flag this to the user — the fold test cannot proceed without bedding data. If column names differ from the above, inspect what is available:

```python
# Check for alternative bedding column names
bedding_cols = [c for c in sites.columns if 'dip' in c.lower() or 'bed' in c.lower() or 'strike' in c.lower()]
print("Possible bedding columns:", bedding_cols)
```

Summarise the loaded data before proceeding — number of sites, range of dips, range of declinations/inclinations — so the user can confirm the right dataset has been loaded.

## Step 3: Check prerequisites

Before running the test, check:

1. **Sample size.** A minimum of N = 6 sites is needed for the bootstrap to be meaningful; N ≥ 20 is strongly preferred. With small N, the bootstrap confidence bounds will be wide and the test may be inconclusive regardless of the geological reality.
2. **Range of bedding dips.** Compute the range of dip values. A spread of at least ~30–40° across sites is generally considered the minimum for the fold test to have discriminating power. A single limb, or gently dipping beds throughout, severely limits what the test can conclude.
3. **Geographic coordinates confirmed.** Restate explicitly that the directions are in geographic (in situ) coordinates.
4. **No duplicate bedding orientations for all sites.** If all sites share the same bedding attitude (e.g. all from the same simple monocline with identical dip), note that the fold test will be uninformative.

```python
n = len(fold_data)
dip_range = fold_data['bed_dip'].max() - fold_data['bed_dip'].min()
print(f"N = {n} sites")
print(f"Bedding dip range: {fold_data['bed_dip'].min():.1f}° to {fold_data['bed_dip'].max():.1f}° (range = {dip_range:.1f}°)")
print(f"Dip direction range: {fold_data['bed_dip_direction'].min():.1f}° to {fold_data['bed_dip_direction'].max():.1f}°")
print(f"Declination range: {fold_data['dir_dec'].min():.1f}° to {fold_data['dir_dec'].max():.1f}°")
print(f"Inclination range: {fold_data['dir_inc'].min():.1f}° to {fold_data['dir_inc'].max():.1f}°")
```

### How to report prerequisite results

Report a clear pass/caution/fail for each check. For example:

- Sample size: pass — "N=24 sites with complete direction and bedding data."
- Sample size: caution — "N=9 sites — above the minimum of 6 but below the preferred minimum of 20. Bootstrap confidence bounds may be wide."
- Dip range: pass — "Bedding dips range from 12° to 68° (range = 56°), providing adequate structural variation for a meaningful fold test."
- Dip range: caution — "Bedding dips range from 5° to 22° (range = 17°). This limited structural variation may reduce the fold test's ability to distinguish pre- from post-folding remanence."

### When a prerequisite fails: warn, then ask — do not stop automatically

If a prerequisite fails (e.g. N < 6, or dip range < 20°):

1. **State the issue clearly and specifically** — which check failed, the actual numbers, and what this means for the test.
2. **Ask the user whether they want to proceed anyway**, with results explicitly caveated, rather than stopping or proceeding silently.

Example phrasing:

> "Before proceeding: only N=5 sites have both direction and bedding data, which is below the minimum of 6 recommended for a meaningful bootstrap fold test. The bootstrap confidence bounds may be very wide and the result effectively uninformative. Would you like me to proceed anyway with this caveat clearly noted, or would you prefer to stop here?"

If the user opts to proceed, carry the caveat through all subsequent steps.

## Step 4: State hypotheses and run the test

Always state the hypotheses explicitly before presenting results:

- **H₀ (null hypothesis):** The directions do not become significantly more clustered upon unfolding — clustering is maximised at or near 0% unfolding (in situ), consistent with a post-folding (secondary) remanence, or the test is inconclusive.
- **H₁ (alternative hypothesis):** Clustering is maximised at or near 100% unfolding (full tilt correction), and the 95% confidence bounds enclose 100% unfolding — consistent with a pre-folding (primary) remanence acquired before deformation.

Assemble the data array and run the bootstrap fold test:

```python
# Assemble the [dec, inc, dip_direction, dip] array using the PmagPy helper
data_array = ipmag.make_diddd_array(
    fold_data['dir_dec'].tolist(),
    fold_data['dir_inc'].tolist(),
    fold_data['bed_dip_direction'].tolist(),
    fold_data['bed_dip'].tolist()
)

print(f"Data array shape: {data_array.shape}")
print("First few rows:", data_array[:3])

# Run the bootstrap fold test
# num_sims=1000 is the default and is appropriate for most uses
# random_seed is set for reproducibility — report this value alongside results
ipmag.bootstrap_fold_test(
    data_array,
    num_sims=1000,
    min_untilt=-10,
    max_untilt=120,
    random_seed=42
)
```

**Notes on parameters:**
- `min_untilt=-10, max_untilt=120` are the defaults and are appropriate unless there is specific reason to believe the maximum clustering lies far outside the 0–100% range (e.g. overturned limbs). Report the values used.
- `random_seed=42` (or any fixed integer) ensures reproducibility. Always set and report this value so the result can be reproduced exactly.
- `bedding_error` can be set to a circular standard deviation (degrees) if there is known uncertainty in bedding measurements — leave at default (0) unless the user has a specific reason to adjust it.
- `ninety_nine=True` changes confidence bounds from 95% to 99% — use the default 95% unless the user specifically requests otherwise.

**Important:** `ipmag.bootstrap_fold_test` requires the data array to be assembled with `ipmag.make_diddd_array` — do not construct this array manually or pass it in any other format, per `functions/policy.md`.

## Step 5: Interpret the results

The fold test produces three plots. Guide the user through reading each one.

### Plot 1: Equal area plot — in situ (uncorrected) directions

Shows the paleomagnetic directions in geographic coordinates (0% unfolding). Use this to:
- Confirm the data loaded correctly (expected N, no obvious outliers)
- Note whether directions are scattered or clustered in situ

### Plot 2: Equal area plot — tilt-corrected directions

Shows directions after full tilt correction (100% unfolding). Use this to:
- Visually assess whether directions appear more tightly grouped after correction
- Note the approximate mean direction in tilt-corrected coordinates

### Plot 3: Bootstrap fold test result — the key diagnostic plot

This is the result plot. It shows:
- **Red dashed lines:** eigenvalue trends for individual bootstrap pseudo-samples across the range of % unfolding (from `min_untilt` to `max_untilt`)
- **Green line:** cumulative distribution function (CDF) of the % unfolding at which each pseudo-sample's eigenvalue is maximised
- **Vertical dashed lines:** the 95% confidence bounds enclosing the central 95% of the pseudo-sample maxima

**Decision rule:**

| Outcome | Interpretation |
|---|---|
| 95% confidence bounds enclose 100% unfolding | **Positive fold test** — clustering maximised at full unfolding, consistent with pre-folding (primary) remanence |
| 95% confidence bounds enclose 0% unfolding | **Negative fold test** — clustering maximised in situ, consistent with post-folding (secondary) remanence |
| 95% confidence bounds enclose an intermediate value (e.g. 40–80%) | **Syn-folding remanence** — or possibly a mixture of pre- and post-folding components |
| 95% confidence bounds are very wide (spanning most of the 0–120% range) | **Inconclusive** — insufficient structural variation or sample size to distinguish pre- from post-folding magnetisation |

Ask the user to describe or share the plot if they can, so the confidence bounds can be read accurately. If the user describes the bounds verbally (e.g. "the dashed lines are at about 80% and 115%"), use those values in the interpretation.

### Plain-language interpretation templates

**Positive result (bounds enclose 100%):**

> "The bootstrap fold test gives a positive result: the 95% confidence bounds on the % unfolding at maximum eigenvalue are approximately [X%] to [Y%], which enclose 100% unfolding (N = [n]; num_sims = 1000; random_seed = 42). This is consistent with the paleomagnetic remanence having been acquired before folding, supporting a pre-deformational (primary) age for the magnetisation."

**Negative result (bounds enclose 0%):**

> "The bootstrap fold test gives a negative result: the 95% confidence bounds are approximately [X%] to [Y%], which enclose 0% unfolding rather than 100% (N = [n]; num_sims = 1000; random_seed = 42). Directions are most tightly clustered in situ, consistent with a post-folding (secondary) remanence acquired after the rocks were deformed."

**Syn-folding result (bounds at intermediate %unfolding):**

> "The bootstrap fold test suggests a syn-folding remanence: the 95% confidence bounds are approximately [X%] to [Y%], centred well away from both 0% and 100% unfolding (N = [n]; num_sims = 1000; random_seed = 42). This may indicate that the remanence was acquired during deformation, or that the dataset contains a mixture of pre- and post-folding remanence components."

**Inconclusive result (very wide bounds):**

> "The bootstrap fold test is inconclusive: the 95% confidence bounds span [X%] to [Y%] — a very wide range that does not allow a distinction between pre- and post-folding remanence (N = [n]; num_sims = 1000; random_seed = 42). This is likely due to [insufficient structural variation in the dataset / small sample size — specify which]. Additional sites with greater variation in bedding orientation would improve the test's discriminating power."

### Limitation to note when reporting

The Tauxe and Watson (1994) bootstrap fold test assesses clustering via eigenvalue analysis of the orientation matrix — it measures concentration of directions, not their mean. This means it can give a positive result even if the tilt-corrected mean direction is anomalous (e.g. significantly different from the expected field direction for the study area and age). The fold test result should always be considered alongside the direction itself — if the tilt-corrected mean direction is geologically implausible, a positive fold test result alone is not sufficient to claim a primary remanence.

## Step 6: Suggest a citation-ready summary

If the result is positive and prerequisites were satisfied, offer draft language for a methods/results section, e.g.:

> "A bootstrap fold test (Tauxe and Watson, 1994) was performed on [n] sites from [unit/location] using PmagPy (Tauxe et al., 2016). The 95% confidence bounds on the % unfolding at maximum eigenvalue are [X%] to [Y%], enclosing 100% unfolding, indicating that the remanent magnetisation was acquired before folding. The test was run with 1000 bootstrap samples (random_seed = 42)."

For negative, syn-folding, or inconclusive results, draft language that honestly reflects the outcome — including any prerequisite caveats — rather than overstating the result.

## Step 7: Suggest next steps

After presenting and interpreting results, offer relevant next steps:

- **If positive:** the fold test supports pre-folding (primary) magnetisation. Suggest combining this with other field tests (reversal test, conglomerate test) and a comparison of the tilt-corrected mean direction with the expected field direction for the study area and age.
- **If negative:** suggest investigating potential remagnetisation events — e.g. comparing the in-situ mean direction with directions from known remagnetisation events in the region, or examining whether a secondary overprint is present in the demagnetisation data.
- **If syn-folding or inconclusive:** suggest collecting additional sites with greater structural variation if possible, and considering component separation via demagnetisation analysis to check for mixed components.
- Exporting the session script for reproducibility (see below).

## Reproducibility note

Every code block actually executed in this workflow should be retained in the session's running script in the order executed, so that the full analysis can be exported as a reproducible Python script or notebook at the end of the session. Always record and report `random_seed` alongside fold test results — the bootstrap is stochastic and results will differ between runs unless the seed is fixed. Per `functions/policy.md`, the script should consist of real PmagPy function calls only.
