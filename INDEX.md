# Knowledge Base Index

This file tells an LLM what knowledge modules are available and when to load them. Read this file first. Only fetch the additional files that are relevant to the user's current request — there is no need to load everything for every session.

## How to use this index

1. Read the user's request.
2. **Always load `functions/policy.md` first, before anything else.** It defines a behavioral rule that applies to the entire session: search for and use real PmagPy functions rather than improvising calculations. This rule is load-bearing and must be followed regardless of which workflow is active.
3. Identify which workflow/convention module(s) below are relevant and fetch those files.
4. When a specific calculation is needed, consult `functions/reference_index.md` to find the relevant PmagPy function before writing any code, and `functions/statistics_and_tests.md` for richer guidance on functions used by the reversal test workflow specifically.
5. If a request doesn't map to any module here, say so honestly rather than guessing — and proceed using general paleomagnetic knowledge with appropriate caution, following the function-use policy.

## Always load

- **`functions/policy.md`**
  Behavioral rule: search for and use real PmagPy functions before performing any calculation manually. Applies to every workflow and every session.

## Available modules

### Data loading

- **`conventions/data_loading.md`**
  How to retrieve a dataset from the MagIC database given a contribution ID (including the practical workaround of downloading a zip from the MagIC web interface and uploading it directly, if API access is unavailable in the current environment), and how the resulting tables are structured, including polarity conventions. Load this whenever the user provides a MagIC ID/file or asks to load data.

### Workflows

- **`workflows/reversal_test.md`**
  The full workflow for testing whether normal and reversed polarity populations share a common mean direction (the "reversal test" / "common mean test"). Covers test options, prerequisites, hypothesis statements, and interpretation. Load this whenever the user mentions a reversal test, common mean test, or asks to compare normal/reversed directions.

### Function reference

- **`functions/reference_index.md`**
  Auto-generated catalog of all functions in `pmagpy.ipmag` (full) and a curated subset of `pmagpy.pmag`, with signatures and one-line summaries, derived by introspecting an installed PmagPy package (generated against PmagPy 4.3.15). Use this to search for a relevant function before performing any calculation. Organized by category (statistical tests, coordinate transforms, MagIC I/O, poles/VGPs, plotting, paleointensity, maps, simulation, general utilities).

- **`functions/statistics_and_tests.md`**
  Tier 1 (hand-curated, richly annotated) guidance for the statistical test functions used by the reversal test workflow — `make_di_block`, `fisher_mean`, `reversal_test_MM1990`, `common_mean_watson`, `common_mean_bootstrap`, `reversal_test_bootstrap_H23`, `common_mean_bayes`. Includes verified signatures, return value conventions (note: result conventions differ between MM1990 and H23 functions), common mistakes, and guidance on what to do when tests disagree.

### Community notes

- **`community/edge_cases.md`**
  Real worked examples and practical observations from using this knowledge base, including: a documented case where MM1990 and H23 bootstrap tests gave different conclusions on the same dataset; a real dataset where recorded `dir_polarity` and inclination-sign polarity splitting disagreed for 100% of sites; and an example of a dataset too small for any available reversal test. Useful context for interpreting similar situations.

## Modules not yet available

The following are referenced by the long-term project plan but do not exist yet in this demo repository. If a user asks about these, say that this knowledge base does not yet cover them:

- Fold tests (note: `ipmag.bootstrap_fold_test` exists in `functions/reference_index.md` but has no Tier 1 workflow file yet)
- Tilt corrections / coordinate system conversions
- Paleomagnetic pole calculation
- Zijderveld diagram interpretation and PCA component selection
- AF/thermal demagnetization workflows

## Versioning note

This index corresponds to the demo version of the knowledge base (reversal test workflow plus a full ipmag function reference). `functions/reference_index.md` was generated against PmagPy 4.3.15 — if a different version is installed, function signatures may differ; see `functions/policy.md` for what to do in that case.
