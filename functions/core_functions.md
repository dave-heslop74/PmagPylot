# Core Function Reference

Plain-language descriptions of PmagPy/ipmag functions referenced by `workflows/reversal_test.md`. These descriptions are written for an LLM to understand when and how to use each function, including common mistakes — they supplement, rather than replace, the official PmagPy documentation.

---

## `pmag.fisher_mean(di_block)`

**What it does:** Calculates the Fisher mean direction and associated statistics from a set of directions.

**When to use:** Whenever you need a mean direction and its statistics (κ, α95, R, N) from a group of individual directions — e.g. site mean from sample directions, or population mean for a reversal test.

**Inputs:** A "di_block" — a list of `[declination, inclination]` pairs (see `make_di_block` below).

**Output:** A dictionary including:
- `dec`, `inc` — mean declination and inclination
- `n` — number of directions
- `r` — resultant vector length
- `k` — Fisher precision parameter (κ); higher values indicate tighter clustering
- `alpha95` — radius of the 95% confidence cone around the mean direction, in degrees

**Output interpretation guidance:** α95 below ~5° indicates a well-constrained mean; above ~15° the mean is poorly defined and results derived from it should be treated cautiously.

**Common mistakes:**
- Passing geographic-coordinate directions when tilt-corrected directions are required for the analysis (not relevant to the reversal test itself, but relevant to downstream workflows).
- Treating κ values as directly comparable to each other without considering N — κ depends on both scatter and sample size.

---

## `ipmag.make_di_block(dec_list, inc_list)`

**What it does:** Converts separate lists/columns of declination and inclination values into the `[dec, inc]` paired format ("di_block") expected by most PmagPy statistical functions.

**When to use:** Any time you have declination and inclination as separate pandas columns (as they typically are when loaded from a MagIC table) and need to pass them to a function like `fisher_mean` or `reversal_test_MM1990`.

**Common mistakes:**
- Mismatched lengths between the declination and inclination lists (e.g. after filtering one but not the other) — check that both inputs have the same length before calling.

---

## `ipmag.reversal_test_MM1990(di_block, plot_CDF=True, plot_stereo=True)`

**What it does:** Performs the Watson V test for a common mean direction between two populations (e.g. normal and reversed polarity), following McFadden & McElhinny (1990).

**When to use:** The primary function for the reversal test workflow when prerequisites (see `workflows/reversal_test.md`, Step 3) are satisfied.

**Inputs:**
- `di_block` — combined list of `[dec, inc]` pairs from **both** populations (the function internally separates and compares them; in some implementations the two populations may need to be passed separately — check the installed PmagPy version's signature and adjust accordingly).
- `plot_CDF` — whether to produce a cumulative distribution plot of the bootstrapped angle differences
- `plot_stereo` — whether to produce a stereonet plot of both populations

**Output:** Watson's V statistic, the critical value V_crit, the angle between the two population means, and the critical angle — used together to determine the McFadden & McElhinny grade (see workflow file for the grade table).

**Common mistakes:**
- Forgetting to flip the reversed population to its antipode before comparison — depending on the PmagPy version, this may be handled internally by the function or may need to be done explicitly. Check the function's behaviour and state clearly to the user which is happening.
- Reporting V and V_crit without also reporting the angle between means and the resulting grade — both pieces of information are needed for a complete interpretation (see workflow Step 5).

---

## `ipmag.common_mean_bootstrap(di_block_1, di_block_2)`

**What it does:** A non-parametric bootstrap test for whether two populations of directions share a common mean, without assuming a Fisher distribution.

**When to use:** When prerequisites for the Watson V test are not met — e.g. one or both populations fail a Fisher-distribution check, or have very different κ values.

**Inputs:** Two separate di_blocks (normal and reversed populations) — note this differs from `reversal_test_MM1990`, which may take a combined block depending on version.

**Output interpretation:** The bootstrap test typically produces confidence regions for each population's mean direction; if these regions overlap, this is consistent with a common mean. The function may produce a plot showing this overlap directly — describe what the plot shows in plain language for the user.

**Common mistakes:**
- Running this test with very small N (below ~10 per population) and over-interpreting the result — bootstrap methods need reasonably sized samples to produce stable confidence regions. Flag this to the user if N is small.

---

## `ipmag.common_mean_parametric(di_block_1, di_block_2)`

**What it does:** An alternative parametric test for a common mean direction between two populations.

**When to use:** As an optional cross-check alongside the Watson V test (Option 1), particularly if the user wants additional confidence in a borderline result. Not the default recommendation.

**Common mistakes:**
- Presenting this as equivalent in standing to the Watson V test in a publication context — it is less commonly used and reviewers may be less familiar with it. If used, explain why it was included (e.g. "as a cross-check").

---

## A note on PmagPy versions

Function signatures (argument names, whether populations are passed combined or separately, return value structure) can vary between PmagPy versions. If the user's environment produces an error that suggests a signature mismatch (e.g. unexpected argument, wrong number of return values), say so plainly, suggest checking the installed version with `import pmagpy; print(pmagpy.__version__)`, and suggest consulting the PmagPy documentation or source for that version rather than guessing repeatedly.
