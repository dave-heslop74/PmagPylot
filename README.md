# PmagPylot

**PmagPylot** is a community-maintained knowledge base that allows a large language model (LLM) to act as a natural-language assistant for paleomagnetic data analysis using [PmagPy](https://github.com/PmagPy/PmagPy).

It is **not code** — it is a set of plain-text documents (Markdown) that describe:

- Common analytical workflows and the order steps should happen in
- Prerequisites and assumptions that should be checked before running a test
- How to interpret results in plain language
- A function-use policy and reference describing which PmagPy functions to call (and how) for a given task
- Data loading and formatting conventions (e.g. MagIC database access)

## How to use this knowledge base

At the start of a session with an LLM (e.g. Claude, ChatGPT, Gemini), point it at this repository's index file and ask it to read and follow it:

> "Please clone this repository using a code/bash execution tool:
> `git clone https://github.com/dave-heslop74/PmagPylot.git`. Also ensure
> PmagPy is installed and importable (e.g.
> `pip install --quiet --break-system-packages pmagpy` if needed), suppressing
> routine installer output so it isn't shown to me. Then read `INDEX.md` from
> the local clone and follow it to load whatever else is relevant. I'd like
> your help with a paleomagnetic data analysis."

The LLM will use `INDEX.md` to decide which other files are relevant and fetch those too — there is no need to provide any other links up front.

### What should happen next

Following `INDEX.md`, the assistant should respond by introducing itself as **PmagPylot** and:

1. Briefly state what it can currently help with, based on what's actually in this knowledge base (currently: the reversal/common mean test workflow — see Status below; this will grow as more workflow files are added)
2. Give a few concrete example prompts the user could try
3. Ask the user to download the relevant MagIC contribution(s) from [earthref.org/MagIC](https://www2.earthref.org/MagIC) (as a zip file) and upload/attach them to the conversation, if not already provided — see `conventions/data_loading.md` for why this is the primary data-loading method in chat environments, and how it's handled once uploaded

If a user's request isn't covered by the current knowledge base, the assistant should say so plainly (per `INDEX.md`, "Modules not yet available") rather than improvising a workflow.

## Versioning and reproducibility

For published analyses, we recommend specifying a release tag or commit hash rather than `main`, so that the knowledge base version used is recorded alongside the PmagPy version. For example:

> "...clone the repository and check out the `v0.1.0` tag before reading
> `INDEX.md`: `git clone https://github.com/dave-heslop74/PmagPylot.git && cd
> PmagPylot && git checkout v0.1.0`..."

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
- Real worked examples from development/testing, including a documented disagreement between two statistical tests on the same dataset, and notes on data-loading constraints in sandboxed environments (`community/edge_cases.md`)

See `INDEX.md` for the full module list and what is not yet covered.
