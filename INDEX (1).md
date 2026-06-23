# Knowledge Base Index — PmagPylot

This file tells an LLM what knowledge modules are available and when to load them. Read this file first.

**Step 0 — getting the knowledge base files (do this before reading anything else below):**

This knowledge base consists of multiple files in a GitHub repository. **As your very first action upon being pointed at this file, before fetching or reading any other file referenced in this index, attempt to clone the repository using a code/bash execution tool:**

```
git clone https://github.com/dave-heslop74/PmagPylot.git
```

**Also as part of this first step**, check that PmagPy is installed and importable, and install it if not:

```bash
python -c "import pmagpy.ipmag" 2>/dev/null || pip install --quiet --break-system-packages pmagpy
```

This should happen before any analysis steps, regardless of whether the user has mentioned PmagPy explicitly — every workflow in this knowledge base depends on it.

- **If the clone succeeds:** read `functions/policy.md` from the local clone first (see "Always load" below), then other files from the clone as relevant. There is no need to fetch anything over the web for the rest of this session — proceed directly with local file reads.
- **If a code/bash execution tool is not available in this session, or the clone command fails for an environment reason (e.g. network/tooling unavailable):** fall back to `curl -sL <url>` per file via code execution if possible (every file is given as a full URL below), and only if no code execution is possible at all, fall back to a `web_fetch`-style tool on the full URLs given below.

**Do not start with `web_fetch` and only fall back to cloning if `web_fetch` fails.** Some `web_fetch`-style tools restrict fetching to URLs that have already appeared in a prior search or fetch result — this means `INDEX.md` itself (provided directly by the user) may fetch successfully, while every *sibling* file referenced from it fails on the first attempt, typically with an error like "This URL was not in any prior search or fetch result." If this error is encountered anyway (e.g. because `web_fetch` was tried first despite the guidance above, or because cloning failed partway through):

- **This is expected** and is not a problem with this knowledge base, the requested file, or the URL — all URLs here are written out in full and are correct.
- **Do not** report this to the user as a blocker or a reason the knowledge base can't be used.
- **Do not** retry the same `web_fetch` call, try variant URLs, search the web for the file, or attempt to fetch the GitHub repo's HTML page instead of the raw file.
- **Immediately run the `git clone` command above** via a code/bash execution tool — this resolves the issue completely.
- Only if a code/bash execution tool genuinely is not available in this session should this limitation be mentioned to the user, e.g.: "I don't have code execution available in this session, so I can't reliably load this knowledge base's files — could you paste the contents of [file] directly, or let me know if code execution can be enabled?"

In all cases: only read files referenced explicitly, written out in full, in this document or in another file already obtained. Do not construct, guess, or modify paths or URLs.

Only read the additional files that are relevant to the user's current request — there is no need to read everything for every session, even after cloning.

## How to use this index

1. **If this is the start of a new session** (i.e. this knowledge base hasn't been loaded yet in the conversation), follow the "Opening message" guidance below before doing anything else.
2. Read the user's request.
3. **Always load `functions/policy.md` first, before anything else** — from the local clone obtained in Step 0 (`functions/policy.md`). This defines a behavioral rule that applies to the entire session: search for and use real PmagPy functions rather than improvising calculations. This rule is load-bearing and must be followed regardless of which workflow is active.
4. Identify which workflow/convention module(s) below are relevant and fetch those files using the full URLs given.
5. When a specific calculation is needed, consult `functions/reference_index.md` to find the relevant PmagPy function before writing any code, and `functions/statistics_and_tests.md` for richer guidance on functions used by the reversal test workflow specifically (URLs below).
6. If a request doesn't map to any module here, say so honestly rather than guessing — and proceed using general paleomagnetic knowledge with appropriate caution, following the function-use policy.

## Opening message

When a session starts (the user has just pointed the assistant at this index, with or without an accompanying request), the assistant can introduce what this knowledge base offers:

1. **Briefly state what it can currently help with**, based on the "Available modules" section below — do not list every function or file, just describe the analyses available in plain terms (currently: the reversal/common mean test and the conglomerate test, both including test selection, prerequisite checking, and interpretation). As more workflow files are added to this knowledge base over time, this description should reflect whatever is actually available at the time, not a fixed list.
2. **Give 2-3 concrete example prompts** the user could try, e.g. "run a reversal test on my site mean directions", "I have normal and reversed polarity sites — are they antipodal?", or "run a conglomerate test on my clast directions".
3. **Ask the user to provide their MagIC data**, if they haven't already: prompt them to download the relevant contribution(s) as a zip from [earthref.org/MagIC](https://www2.earthref.org/MagIC) and upload/attach the file(s) to the conversation. See `conventions/data_loading.md` (URL below) for how this is handled once received, and for the preferred `ipmag.download_magic_from_id()` method in environments where direct API access works.
4. If the user's original message already included a specific request and/or an uploaded file, this introduction can be folded into the response to that request rather than presented as a separate first step — but it should still appear in some form (what's available, example prompts, and a prompt for data if not yet provided).

Keep this concise — a short paragraph plus a few examples, not an exhaustive manual.

## Always load

- **`functions/policy.md`**
  `https://raw.githubusercontent.com/dave-heslop74/PmagPylot/main/functions/policy.md`
  Behavioral rule: search for and use real PmagPy functions before performing any calculation manually. Applies to every workflow and every session.

## Available modules

### Data loading

- **`conventions/data_loading.md`**
  `https://raw.githubusercontent.com/dave-heslop74/PmagPylot/main/conventions/data_loading.md`
  How to retrieve a dataset from the MagIC database given a contribution ID (including the practical workaround of downloading a zip from the MagIC web interface and uploading it directly, if API access is unavailable in the current environment), and how the resulting tables are structured, including polarity conventions. Load this whenever the user provides a MagIC ID/file or asks to load data.

### Workflows

- **`workflows/reversal_test.md`**
  `https://raw.githubusercontent.com/dave-heslop74/PmagPylot/main/workflows/reversal_test.md`
  The full workflow for testing whether normal and reversed polarity populations share a common mean direction (the "reversal test" / "common mean test"). Covers test options, prerequisites, hypothesis statements, and interpretation. Load this whenever the user mentions a reversal test, common mean test, or asks to compare normal/reversed directions.

- **`workflows/conglomerate_test.md`**
  `https://raw.githubusercontent.com/dave-heslop74/PmagPylot/main/workflows/conglomerate_test.md`
  The full workflow for the Watson (1956) conglomerate test — a test of whether clast directions from a conglomerate or breccia are randomly distributed on the sphere, as expected if the remanent magnetisation is pre-depositional. Covers geological context checks, prerequisites (sample size, clast independence), hypothesis statements, running `ipmag.conglomerate_test_Watson()`, and interpretation. Load this whenever the user mentions a conglomerate test, clast directions, or asks to test whether directions are randomly distributed.

### Function reference

- **`functions/reference_index.md`**
  `https://raw.githubusercontent.com/dave-heslop74/PmagPylot/main/functions/reference_index.md`
  Auto-generated catalog of all functions in `pmagpy.ipmag` (full) and a curated subset of `pmagpy.pmag`, with signatures and one-line summaries, derived by introspecting an installed PmagPy package (generated against PmagPy 4.3.15). Use this to search for a relevant function before performing any calculation. Organized by category (statistical tests, coordinate transforms, MagIC I/O, poles/VGPs, plotting, paleointensity, maps, simulation, general utilities).

- **`functions/statistics_and_tests.md`**
  `https://raw.githubusercontent.com/dave-heslop74/PmagPylot/main/functions/statistics_and_tests.md`
  Tier 1 (hand-curated, richly annotated) guidance for the statistical test functions used by the reversal test workflow — `make_di_block`, `fisher_mean`, `reversal_test_MM1990`, `common_mean_watson`, `common_mean_bootstrap`, `reversal_test_bootstrap_H23`, `common_mean_bayes`. Includes verified signatures, return value conventions (note: result conventions differ between MM1990 and H23 functions), common mistakes, and guidance on what to do when tests disagree. Also consult when computing Fisher statistics upstream of other workflows (e.g. `pmag.fisher_mean` for R and n in the conglomerate test).

### Community notes

- **`community/edge_cases.md`**
  `https://raw.githubusercontent.com/dave-heslop74/PmagPylot/main/community/edge_cases.md`
  Real worked examples and practical observations from using this knowledge base, including: a documented case where MM1990 and H23 bootstrap tests gave different conclusions on the same dataset; a real dataset where recorded `dir_polarity` and inclination-sign polarity splitting disagreed for 100% of sites; an example of a dataset too small for any available reversal test; and notes on data-download functions in sandboxed environments. Useful context for interpreting similar situations.

## Modules not yet available

The following are referenced by the long-term project plan but do not exist yet in this repository. If a user asks about these, say that this knowledge base does not yet cover them:

- Fold tests (note: `ipmag.bootstrap_fold_test` exists in `functions/reference_index.md` but has no Tier 1 workflow file yet)
- Tilt corrections / coordinate system conversions
- Paleomagnetic pole calculation
- Zijderveld diagram interpretation and PCA component selection
- AF/thermal demagnetization workflows

## Versioning note

This index currently covers two workflows (reversal/common mean test; conglomerate test) plus a full ipmag function reference. `functions/reference_index.md` was generated against PmagPy 4.3.15 — if a different version is installed, function signatures may differ; see `functions/policy.md` for what to do in that case.

If using a versioned/tagged release rather than `main`, all URLs above should have `main` replaced with the relevant tag or commit hash (e.g. `.../PmagPylot/v0.1.0/...`).
