# Community Edge Cases and Notes

This file collects practical observations, edge cases, and notes from real use of this knowledge base. Contributions here have a lower bar than `workflows/` or `functions/` — if you observed something interesting or unexpected while using the assistant, it can go here as a starting point, even if not yet fully resolved into formal workflow guidance.

---

## `ipmag.download_magic_from_id()` and sandboxed network access

**Observation:** `ipmag.download_magic_from_id()` is the correct, tested PmagPy function for retrieving a contribution by ID (preferred per `functions/policy.md` over a hand-rolled `requests.get()`). However, in claude.ai's bash/code execution sandbox, it fails with `success=False, result="Failed to download: Forbidden"` — because, like any direct call to `api.earthref.org`, it is blocked by the sandbox's network egress allowlist.

**Why this matters:** This is not a defect in the function, and not something a different PmagPy function or a manual `requests` call would fix — both paths reach the same blocked host. It's a useful "stop" signal: if `download_magic_from_id` fails with "Forbidden" (as opposed to e.g. a 404 for a genuinely missing contribution ID), the cause is almost certainly the execution environment's network policy, not the contribution ID or the function call itself. At that point, switch to the upload-zip alternative in `conventions/data_loading.md` rather than retrying API variations.

**In environments with normal network access** (local installs, Tier 1/2 deployments per `README.md`), `download_magic_from_id` is expected to work normally and should be the default data-loading method.

---

## Watson V (MM1990) and Heslop bootstrap (H23) disagreement — Hector Formation example

**Dataset:** MagIC contribution 19938 (Hector Formation, Mojave Block; data from MacFadden et al. 1990, as compiled in MagIC). 48 site mean directions, split 24/24 by inclination sign (no `dir_polarity` column was present in this dataset).

**Observation:** Two different tests for a common mean direction gave different conclusions on this dataset:

- **`reversal_test_MM1990`** (analytical/manual calculation during development — see caveat below): angle between means ≈ 13.4°, critical angle ≈ 21.4° → fail to reject H₀, **Grade C** ("marginal — interpret with caution")
- **`reversal_test_bootstrap_H23`** (real PmagPy function, `num_sims=10000`, `random_seed=42`): Lmin = 21.98, Lmin_c = 13.66, p ≈ 0.012 → **reject H₀**

These point in different directions: MM1990 says "marginal pass," H23 says "reject at p≈0.01."

**Important caveat on this specific comparison:** The MM1990-side numbers above came from a manual, analytically-approximated implementation during early development of this knowledge base — *not* a call to the real `ipmag.reversal_test_MM1990()`. Per `functions/policy.md`, this should not have happened, and the comparison should be **redone using the real `reversal_test_MM1990` function** before drawing firm conclusions about whether this disagreement reflects a genuine MM1990-vs-H23 difference or an error in the manual calculation. This entry is retained as a worked example of (a) why the function-use policy exists, and (b) the kind of cross-test comparison that's valuable to document once done correctly.

**Context from the literature:** This is the same Hector Formation dataset used as a worked example in Heslop et al. (2023) itself, originally analysed by Tauxe et al. (1991). It is plausible that this dataset was chosen in the literature specifically because it illustrates an interesting/borderline case — worth checking the original paper's discussion of this example directly.

**What a user/assistant should do if this disagreement is reproduced with verified function calls:**
1. Report both results in full (see `functions/statistics_and_tests.md`, "When tests disagree")
2. Note that the H23 test's null-hypothesis framework (with p-value) is generally preferred over MM1990 alone for borderline cases
3. Consider running `common_mean_bayes` as a third perspective
4. Examine individual site directions for outliers that might be driving the discrepancy — a small number of sites near the polarity-split boundary could affect the inclination-sign-based split disproportionately
5. If no `dir_polarity` column was available and inclination-sign splitting was used, consider whether any sites are mis-assigned (see the polarity assignment note below)

---

## Polarity assignment: recorded `dir_polarity` vs inclination sign

**Observation:** For MagIC contribution 16663 (Yuntaiguan Formation, South China, Devonian), the recorded `dir_polarity` column and a naive inclination-sign split (`dir_inc > 0` = normal) **disagreed for 100% of sites (34/34)**. The recorded polarity showed 31 reversed / 3 normal; the inclination-sign heuristic would have given 31 normal / 3 reversed — backwards.

**Why:** At this site's paleolatitude and age, the expected field directions (for either polarity) have shallow inclinations and declinations far from 0°/180° — the simple "positive inclination = normal" heuristic implicitly assumes a reference field similar to today's high-latitude Northern Hemisphere field, which does not hold generally.

**Implication for the workflow:** `conventions/data_loading.md` and `workflows/reversal_test.md` Step 1 already caution that recorded `dir_polarity` should be preferred over inclination-sign splitting when available. This real example is strong evidence for why — the two methods can disagree completely, not just at the margins. When `dir_polarity` is **not** available (as in contribution 19938), inclination-sign splitting is a fallback, but results should be presented with this caveat, and any borderline test outcomes (as above) should prompt a check of whether individual sites' polarity assignments are plausible.

---

## Small populations and test applicability

**Observation:** For MagIC contribution 16663, the recorded normal-polarity population had only N = 3 sites (vs N = 31 reversed). This is below the Watson V minimum (N ≥ 5) *and* the bootstrap test's recommended minimum (N ≥ 10) — no test in the current workflow is well-suited to this comparison.

**What happened:** Per the (now-updated) workflow guidance, this should be flagged to the user with a clear warning, and the user should be asked whether to proceed anyway (with results caveated as unreliable) or stop. This is a real example of a dataset where, after loading and basic checks, the honest answer is "this isn't a good candidate for a reversal test as configured" — which is itself a useful and correct output for the assistant to produce, rather than forcing a result.
