# PmagPy Assistant Knowledge Base

This repository is a community-maintained knowledge base that allows a large language model (LLM) to act as a natural-language assistant for paleomagnetic data analysis using [PmagPy](https://github.com/PmagPy/PmagPy).

It is **not code** — it is a set of plain-text documents (Markdown and YAML) that describe:

- Common analytical workflows and the order steps should happen in
- Prerequisites and assumptions that should be checked before running a test
- How to interpret results in plain language
- Descriptions of PmagPy functions written for an LLM audience
- Data loading and formatting conventions (e.g. MagIC database access)

## How to use this knowledge base

At the start of a session with an LLM (e.g. Claude, ChatGPT, Gemini), point it at this repository and ask it to read the relevant files before helping with your analysis. For example:

> "Please fetch and read the files at
> `https://raw.githubusercontent.com/<org>/pmagpy-assistant-knowledge/main/INDEX.md`
> and follow the index to load the reversal test workflow. Then help me run a
> reversal test on MagIC contribution 16663."

The LLM will use the `INDEX.md` file to decide which other files are relevant and fetch those too.

## Versioning and reproducibility

For published analyses, we recommend specifying a release tag or commit hash rather than `main`, so that the knowledge base version used is recorded alongside the PmagPy version. For example:

> "...read the files at
> `https://raw.githubusercontent.com/<org>/pmagpy-assistant-knowledge/v0.1.0/INDEX.md`..."

## Contributing

This knowledge base is intended to be community-driven. Contributions are welcome via pull request:

- **`workflows/`** and **`functions/`** — core content, reviewed by maintainers for accuracy
- **`community/`** — practical notes, edge cases, and lab-specific tips with a lower bar for acceptance

You do not need to know how to code to contribute — these are plain text files describing paleomagnetic analysis practice. If you can describe a workflow, an assumption, or a common mistake in words, you can contribute.

## Status

This is an early demonstration version. It currently covers:

- One full workflow: the reversal/common mean test (`workflows/reversal_test.md`)
- A function-use policy (`functions/policy.md`) requiring real PmagPy function calls rather than improvised calculations
- An auto-generated reference of the full `pmagpy.ipmag` API plus a curated `pmagpy.pmag` subset (`functions/reference_index.md`), generated against PmagPy 4.3.15
- Hand-curated, verified documentation for the statistics/test functions used by the reversal test workflow (`functions/statistics_and_tests.md`)
- Real worked examples from development/testing, including a documented disagreement between two statistical tests on the same dataset (`community/edge_cases.md`)

See `INDEX.md` for the full module list and what is not yet covered.
