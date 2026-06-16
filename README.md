# PmagPylot

**PmagPylot** is a community-maintained knowledge base that allows a large language model (LLM) to act as a natural-language assistant for paleomagnetic data analysis using [PmagPy](https://github.com/PmagPy/PmagPy).

It is **not code** — it is a set of plain-text documents (Markdown) that describe:

- Common analytical workflows and the order steps should happen in
- Prerequisites and assumptions that should be checked before running a test
- How to interpret results in plain language
- A function-use policy and reference describing which PmagPy functions to call (and how) for a given task
- Data loading and formatting conventions (e.g. MagIC database access)

## Platform compatibility

**PmagPylot currently requires [Claude](https://claude.ai) (claude.ai) for full functionality.**

Full use of this knowledge base requires two capabilities that Claude provides but most other LLM platforms do not currently support reliably:

- **Code/bash execution** — needed to clone this repository and to run PmagPy analyses directly in the conversation
- **GitHub access** — needed to clone this repository at the start of a session

Other platforms (ChatGPT, Gemini, etc.) can be used to read through workflow guidance and generate code, but cannot clone the repository or execute PmagPy analyses within the conversation. We expect to extend platform support as execution environments on other platforms mature — a pre-packaged zip-based approach for platforms without GitHub access is planned for a future release.

## How to use this knowledge base

At the start of a new Claude session, paste the following prompt:

```
Please clone this repository using a code/bash execution tool:
git clone https://github.com/dave-heslop74/PmagPylot.git
Also ensure PmagPy is installed and importable
(pip install --break-system-packages pmagpy if needed).
Then read INDEX.md from the local clone and follow it to load
whatever else is relevant. I'd like your help with a
paleomagnetic data analysis.
```

Claude will clone the repository, check that PmagPy is available, read `INDEX.md`, and load whichever workflow and reference files are relevant to your request — there is no need to provide any other links up front.

### What should happen next

Following `INDEX.md`, the assistant should:

1. Briefly state what it can currently help with, based on what's actually in this knowledge base (currently: the reversal/common mean test workflow — see Status below; this will grow as more workflow files are added)
2. Give a few concrete example prompts you could try
3. Ask you to download the relevant MagIC contribution(s) from [earthref.org/MagIC](https://www2.earthref.org/MagIC) (as a zip file) and attach them to the conversation, if not already provided

If your request isn't covered by the current knowledge base, the assistant will say so plainly rather than improvising a workflow.

## Versioning and reproducibility

For published analyses, we recommend specifying a release tag or commit hash rather than `main`, so that the knowledge base version used is recorded alongside the PmagPy version. For example, replace the clone command above with:

```
git clone https://github.com/dave-heslop74/PmagPylot.git && cd PmagPylot && git checkout v0.1.0
```

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
