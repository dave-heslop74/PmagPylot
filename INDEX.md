# Knowledge Base Index

This file tells an LLM what knowledge modules are available and when to load them. Read this file first. Only fetch the additional files that are relevant to the user's current request — there is no need to load everything for every session.

## How to use this index

1. Read the user's request.
2. Identify which module(s) below are relevant.
3. Fetch those specific files (raw GitHub URLs) before responding.
4. If a request doesn't map to any module here, say so honestly rather than guessing — and proceed using general paleomagnetic knowledge with appropriate caution.

## Available modules

### Data loading

- **`conventions/data_loading.md`**
  How to retrieve a dataset from the MagIC database given a contribution ID, and how the resulting tables are structured. Load this whenever the user provides a MagIC ID or asks to load data.

### Workflows

- **`workflows/reversal_test.md`**
  The full workflow for testing whether normal and reversed polarity populations share a common mean direction (the "reversal test" / "common mean test"). Covers test options, prerequisites, hypothesis statements, and interpretation. Load this whenever the user mentions a reversal test, common mean test, or asks to compare normal/reversed directions.

### Function reference

- **`functions/core_functions.md`**
  Plain-language descriptions of the PmagPy/ipmag functions referenced by the workflows above, including common mistakes and output interpretation. Load this alongside a workflow file when you need details on a specific function call.

## Modules not yet available

The following are referenced by the long-term project plan but do not exist yet in this demo repository. If a user asks about these, say that this knowledge base does not yet cover them:

- Fold tests
- Tilt corrections / coordinate system conversions
- Paleomagnetic pole calculation
- Zijderveld diagram interpretation and PCA component selection
- AF/thermal demagnetization workflows

## Versioning note

This index corresponds to the demo version of the knowledge base (reversal test workflow only). Future versions will expand coverage module by module.
