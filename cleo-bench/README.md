# Cleo Bench

A benchmark of the legendary **Cleo integrals** from Math StackExchange, plus a harness that has
Claude attempt every one of them closed-book, with full derivations, high-precision numerical
verification, and independent grading.

**Results: [results/REPORT.md](results/REPORT.md)** — scoreboard, per-problem verdicts, and a full
derivation for every problem in [results/solutions/](results/solutions/).

## Background

Between 2013 and 2015, the user [Cleo](https://math.stackexchange.com/users/97378/cleo) posted
answers to some of the hardest definite integrals on Math StackExchange — famously as bare
closed forms with **no derivation whatsoever**, often within hours of the question appearing.
The answers kept checking out numerically, and "how does Cleo do it?" became one of the site's
great mysteries. (In 2025 the account was linked to Vladimir Reshetnikov, who had posed many of
the questions under other accounts; the Cleo account was subsequently suspended, which is why its
reputation now displays as 1.)

Cleo left 39 answers. This repo scrapes all of them — 34 integrals, 1 Euler sum, and 4 other
closed-form problems — into a dataset and asks: *can a frontier model not just name the closed
form, but actually prove it?*

## Dataset

Built by [`scrape.py`](scrape.py) (Python stdlib only) from the official Stack Exchange API:

- `data/cleo_dataset.jsonl` — one record per problem: question id/title/url/tags/asker,
  full question markdown, Cleo's answer markdown, scores, dates, and a coarse `type`
  (`integral` / `series` / `other`).
- `data/problems/<slug>.md` — solver-facing problem statements (**no answers**). One statement
  (q964438) is truncated at an asker edit that referenced Cleo's answer; the flag `sanitized`
  marks it in the dataset.
- `data/answers/<slug>.md` — Cleo's answer for each problem (the ground truth).

Content is CC BY-SA (Stack Exchange); records retain attribution links.

## Benchmark protocol

One workflow run spawns three agents per problem (`claude-fable-5`):

1. **Truth** (effort `high`) — reads Cleo's answer, translates the closed form into an `mpmath`
   expression, evaluates it to 40 digits, and independently checks it against direct
   high-precision numerical evaluation of the integral itself.
2. **Solve** (effort `max`) — *closed book*: gets only the problem statement. No network, no
   reading of answer files. Must produce a complete rigorous derivation in
   `results/solutions/<slug>.md`, plus an `mpmath` expression for its final answer, and must
   verify it numerically (target ≥25 agreeing digits) against direct evaluation of the integral.
   The prompt states that a recalled answer without derivation scores as unsolved.
3. **Grade** (effort `high`) — recomputes both closed forms itself at 50 dps (never trusting
   claimed digits), reads the solver's proof, spot-checks pivotal steps, and returns a verdict:
   `correct` / `equivalent_but_different_form` / `incorrect` / `no_answer`, plus a
   derivation-quality rating. **Solved** = correct value *and* a complete (or nearly complete)
   derivation.

`assemble.py` merges the dataset with the workflow output (`results/results.json`) into
`results/REPORT.md`.

## Reproduce

```bash
python3 scrape.py                                   # rebuild the dataset (4 API calls)
python3 -m venv .venv && .venv/bin/pip install mpmath sympy
# run the benchmark workflow (Claude Code), then:
python3 assemble.py                                 # regenerate results/REPORT.md
```

## Caveat

These problems are famous and their solutions are public, so training-set contamination is
unavoidable. The bench measures whether the model can produce a *complete, numerically verified
derivation* under a no-lookup policy — not whether it can discover the closed forms in a vacuum.
