# Function Use Policy

This is a behavioral rule for the assistant and applies to **every workflow** in this knowledge base. It exists because of a documented failure mode: when an LLM computes a statistic by recalling/deriving a formula itself rather than calling the tested PmagPy implementation, the result can disagree with the real function's output — sometimes substantially — and the disagreement is not a finding about the data, it is an artifact of the improvisation.

## The rule

1. **Before performing any paleomagnetic calculation, search `functions/reference_index.md`** for an existing PmagPy function that performs (or closely approximates) the requested operation.

2. **If a suitable function exists, call it via code execution.** Do not reimplement its logic manually, even if you are confident you know the underlying formula or algorithm. This applies to statistical tests, Fisher/Kent statistics, coordinate transforms, bootstrap procedures — anything for which a PmagPy function exists.

3. **If no suitable PmagPy function exists**, you may offer to implement the calculation directly (e.g. from a published formula or paper). When doing so:
   - State explicitly that PmagPy does not have a built-in function for this operation
   - State that the implementation is being constructed for this session, has not been validated against the tested PmagPy codebase, and may contain errors
   - Where possible, cite the source (paper, equation) the implementation is based on
   - Do not present the result with the same confidence as a result from a tested PmagPy function

4. **If a PmagPy function exists but its signature, behaviour, or return values are unclear** (e.g. from the one-line summary in `reference_index.md`), use `help(module.function_name)` or `inspect.signature()` in code execution to check before calling it — do not guess at argument names or ordering.

5. **If two different functions could plausibly answer the same question and give different results** (e.g. a parametric vs. non-parametric test), report both results explicitly along with what each test assumes, rather than picking one and presenting it as "the" answer. See the relevant workflow file for guidance on interpreting disagreements between tests.

## Why this matters for reproducibility

A session script that calls `ipmag.reversal_test_bootstrap_H23(...)` is reproducible — anyone with the same PmagPy version and input data gets the same result. A session script containing a hand-rolled reimplementation of Watson's V is not equivalent to "running PmagPy" even if it produces a plausible-looking number, and should never be presented to the user as if it were.

## A real example of why this rule exists

During development of this knowledge base, a manual (analytically-approximated) implementation of the Watson V test gave a "Grade C / fail to reject H₀" result for a test dataset. When the same data was run through the real `ipmag.reversal_test_bootstrap_H23()` function, the result was "reject H₀" (p ≈ 0.01) — a different conclusion. Investigating the discrepancy was a worthwhile scientific exercise (see `community/edge_cases.md`), but it should not have been possible for the manual calculation to be presented as a PmagPy result in the first place. This policy exists to prevent that.
