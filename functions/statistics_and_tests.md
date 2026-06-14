# Statistics and Tests — Function Reference (Tier 1)

This file provides rich, hand-curated guidance for the PmagPy functions used by `workflows/reversal_test.md`. Unlike `functions/reference_index.md` (an auto-generated catalog of the full package), the descriptions here are written specifically to support that workflow, including common mistakes and output interpretation.

**All signatures and return values below were verified against an installed PmagPy package (version 4.3.15) using `inspect.signature()` and `inspect.getdoc()` — not reproduced from memory or external documentation.** If your installed PmagPy version differs, check `help(ipmag.function_name)` before relying on the details below, and see `functions/policy.md`.

---

## `ipmag.make_di_block(dec, inc, unit_vector=True)`

**What it does:** Converts separate lists of declination and inclination values into the nested `[dec, inc, 1.0]` ("di_block") format expected by most PmagPy statistical functions.

**When to use:** Any time you have declination/inclination as separate columns (as they typically are when loaded from a MagIC `sites` table) and need to pass them to a statistical function.

**Returns:** A nested list, e.g. `[[180.3, 12.1, 1.0], [179.2, 13.7, 1.0], ...]` if `unit_vector=True` (the default), or `[[dec, inc], ...]` if `unit_vector=False`.

**Common mistakes:**
- Mismatched lengths between the declination and inclination lists (e.g. after filtering one but not the other) — check both have the same length first.
- Some downstream functions (e.g. `common_mean_watson`, `common_mean_bootstrap_H23`) expect `[dec, inc]` pairs rather than `[dec, inc, 1.0]` triples — check the target function's expected input format. If in doubt, `unit_vector=False` gives the simpler `[dec, inc]` form.

---

## `ipmag.fisher_mean(dec=None, inc=None, di_block=None)`

**What it does:** Calculates the Fisher mean direction and associated statistics from a set of directions.

**When to use:** Whenever you need a mean direction and its statistics (κ, α95, R, N) from a group of individual directions — e.g. a population mean for the reversal test prerequisite checks.

**Inputs:** Either separate `dec`/`inc` lists, or a `di_block`.

**Returns:** A dictionary with keys `dec`, `inc`, `n`, `r`, `k`, `alpha95`, `csd`.

**Output interpretation:**
- `alpha95` below ~5° indicates a well-constrained mean; above ~15° the mean is poorly defined and results derived from it should be treated cautiously.
- `k` (κ) is the Fisher precision parameter — higher values indicate tighter clustering, but depend on both scatter and N, so are not directly comparable between populations of very different size without care.

**Common mistakes:**
- Passing both `dec`/`inc` lists *and* `di_block` is unnecessary — only one input method is needed, and `di_block` takes precedence if both are given.

---

## `ipmag.reversal_test_MM1990(dec=None, inc=None, di_block=None, plot_CDF=False, plot_stereo=False, save=False, save_folder='.', fmt='svg', random_seed=None)`

**What it does:** The Watson V test for a common mean direction following McFadden & McElhinny (1990), using Monte Carlo simulation. **This function handles polarity separation and antipodal flipping internally** — it is a wrapper that splits the input data into two polarities using `pmag.flip()` and flips the reversed population to its antipode before testing.

**When to use:** Primary recommendation for the reversal test workflow when prerequisites (workflow Step 3) are satisfied, **and the user has not already manually separated normal/reversed populations** — pass the combined dataset and let this function do the splitting.

**Inputs:** A single combined `di_block` (or `dec`/`inc` lists) containing **both** polarities together. Do not pre-split into normal/reversed for this function — that's its job.

**Returns:** A tuple `(result, angle, critical_angle, classification)`:
- `result` (int): 1 = pass (cannot reject common mean), 0 = fail
- `angle` (float): angle between the Fisher means of the two populations (after flipping)
- `critical_angle` (float): the critical angle for the test
- `classification` (str): McFadden & McElhinny (1990) grade — `'A'`, `'B'`, `'C'`, `'indeterminate'` for a positive test, or `''` for a negative test

**Output interpretation:** See `workflows/reversal_test.md` Step 5 for the grade table and interpretation templates. Report `angle`, `critical_angle`, and `classification` together — `result` alone (pass/fail) does not convey the grade.

**Common mistakes:**
- Pre-splitting and flipping the data yourself, then passing it to this function — it expects to do this itself. If you have already separated by polarity (e.g. using a recorded `dir_polarity` column rather than the function's automatic split), use `ipmag.common_mean_watson()` instead (below), which takes two pre-separated populations directly.
- Reporting only `result` (pass/fail) without `angle`/`critical_angle`/`classification` — incomplete per the workflow.

---

## `ipmag.common_mean_watson(Data1, Data2, NumSims=5000, print_result=True, plot=False, save=False, save_folder='.', fmt='svg', random_seed=None)`

**What it does:** The Watson V test for a common mean direction between **two pre-separated** populations (does not do its own polarity splitting or antipodal flipping).

**When to use:** When you have already separated normal and reversed populations yourself — e.g. using a recorded `dir_polarity` column (see `conventions/data_loading.md`) rather than relying on automatic inclination-sign splitting — and need to compare them. **You must flip one population to its antipode yourself before calling this function** if testing a reversal (normal vs reversed) rather than a general common-mean comparison.

**Inputs:** `Data1`, `Data2` — each a `di_block` of `[dec, inc]` pairs.

**Returns:** Tuple `(result, angle, critical_angle, classification)` — same structure as `reversal_test_MM1990`.

**Common mistakes:**
- Forgetting to flip one population to its antipode before calling this function for a *reversal* test specifically (as opposed to a general common-mean test, e.g. comparing two normal-polarity sites where no flip is needed).

---

## `ipmag.common_mean_bootstrap(Data1, Data2, NumSims=1000, ..., random_seed=None)`

**What it does:** The Tauxe (2010) bootstrap test for a common mean — generates bootstrap distributions of the X, Y, Z Cartesian components of each population's mean and checks whether their 95% confidence bounds overlap.

**When to use:** As a non-parametric alternative when Watson V prerequisites (Fisher distribution, comparable κ) are not met. Per the original literature, recommended minimum N ≈ 10 per population (though see Heslop et al. 2023 for a more rigorous treatment of small-N behaviour, via `common_mean_bootstrap_H23` below).

**Inputs:** Two `di_block`s of `[dec, inc]` pairs. (Can also accept a single direction as `Data2` to test whether it falls within `Data1`'s confidence region — see docstring for details.)

**Returns:** `1` if the test passes (bounds overlap, consistent with common mean), `0` if it fails.

**Important limitation:** As noted in Heslop et al. (2023), "the bootstrap test for a common mean paleomagnetic direction does not consider a null hypothesis and can yield outcomes that cannot be interpreted in terms of a statistical significance level" in this original (Tauxe 2010) form. There is no p-value — only a pass/fail based on confidence interval overlap. For a result with an associated significance level, prefer `common_mean_bootstrap_H23` / `reversal_test_bootstrap_H23`.

---

## `ipmag.reversal_test_bootstrap_H23(dec=None, inc=None, di_block=None, num_sims=10000, alpha=0.05, plot=True, save=False, save_folder='.', fmt='svg', verbose=True, random_seed=None)`

**What it does:** The Heslop et al. (2023) bootstrap reversal test — extends the Tauxe (2010) bootstrap approach with a proper null-hypothesis-significance-testing framework, yielding a p-value. Internally calls `common_mean_bootstrap_H23` after flipping the second population.

**When to use:** Recommended non-parametric alternative when Watson V prerequisites are not met, or as a cross-check alongside Watson V even when prerequisites are met — **this is now the preferred bootstrap approach** over the plain `common_mean_bootstrap`, since it provides an interpretable p-value.

**Inputs:** A single combined `di_block` (or `dec`/`inc` lists) containing both polarities, **or** call `common_mean_bootstrap_H23` directly with two pre-separated populations and `reversal=True`.

**Returns:** Tuple `(result, Lmin, Lmin_c, p)`:
- `result` (int): 0 if H₀ (common mean) is rejected, 1 if not rejected — **note this is the opposite convention to `reversal_test_MM1990`'s `result`, where 1 = pass.** Check which function you're using before interpreting `result`.
- `Lmin` (float): the test statistic
- `Lmin_c` (float): the critical value
- `p` (float): p-value

**Output interpretation:** `Lmin > Lmin_c` (equivalently `p < alpha`, `result == 0`) → reject H₀, populations do not share a common mean. Report `Lmin`, `Lmin_c`, and `p` together.

**Common mistakes:**
- Confusing the `result` convention with `reversal_test_MM1990` — in this function, `result == 0` means *reject* H₀ (a "negative" reversal test), whereas in `reversal_test_MM1990`, `result == 0` means *fail*. Always report `p` alongside `result` to avoid ambiguity.
- For very small populations (e.g. N < ~10), bootstrap-based p-values may be less stable — note sample size when reporting results.

---

## `ipmag.common_mean_bayes(Data1, Data2, reversal_test=False)`

**What it does:** Bayesian test for a common mean direction following Heslop & Roberts (2018), returning a Bayes factor and posterior probability rather than a frequentist p-value.

**When to use:** As an additional perspective alongside Watson V / bootstrap results, particularly useful for small datasets where frequentist tests may be unstable (see `community/edge_cases.md` for a worked example of using this alongside `reversal_test_bootstrap_H23`). Set `reversal_test=True` to have the function handle antipodal flipping of `Data2` internally.

**Inputs:** Two `di_block`s of `[dec, inc]` pairs.

**Returns:** `BF0` (Bayes factor), `P` (posterior probability of common mean), `support` (qualitative category, e.g. "strong support").

**Note:** A Bayes factor / posterior probability is a different kind of quantity from a p-value and should not be directly equated with one in interpretation — see the Bayesian statistics literature for guidance on interpreting `support` categories (e.g. Jeffreys' scale).

---

## When tests disagree

It is possible — and was observed during development of this knowledge base — for `reversal_test_MM1990` (parametric) and `reversal_test_bootstrap_H23` (non-parametric, with p-value) to give different conclusions on the same dataset, particularly for borderline (e.g. Grade C) results. When this happens:

1. Report **both** results in full, with their respective statistics
2. Do not silently prefer one — explain what each test assumes (Fisher distribution for MM1990; fewer assumptions for H23) and that disagreement is itself informative
3. Consider running `common_mean_bayes` as a third perspective
4. See `community/edge_cases.md` for a documented real-world example of this situation
