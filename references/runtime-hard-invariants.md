# Runtime Hard Invariants

Use this file as the primary execution checklist during active runs. Keep it short in memory. Treat the other protocol files as detailed reference material unless a specific situation requires them.

## Shared Runtime Checklist

1. Register the hypothesis and leakage guard before changing code.
2. Measure the baseline/control before initializing run artifacts.
3. Initialize artifacts immediately after the baseline/control is known.
4. Treat every completed experiment as unfinished until it is logged before the next one starts.
5. Log negative and inconclusive results, not only supported hypotheses.
6. Do not emit placeholder progress/status messages when there is no new experiment, no new verification result, and no new blocker.
7. Use bundled helper scripts for authoritative TSV/JSON updates, keep/stop gating, and row/state semantics.
8. All normal run artifacts are workspace-owned under `research-validation-results/`: `results.tsv`, `state.json`, `context.json`, and `lessons.md`.
9. Stop only on hypothesis supported/refuted, evidence budget exhausted, manual stop, configured iteration cap, a true blocker, or the documented soft-blocker handoff after strategy exhaustion.
10. After any context compaction event, re-read `core-principles.md`, this file, and `research-validation-workflow.md` before the next iteration.
11. Every 10 iterations, run the Protocol Fingerprint Check. If any item fails, re-read the loaded runtime docs before continuing.

## Protocol Fingerprint Check

Verify you can still recall:

- hypothesis and leakage guard before edits,
- baseline/control before init,
- log every completed experiment before the next one starts,
- negative and inconclusive results are valid outcomes,
- helper scripts own authoritative TSV/JSON updates and keep/stop gating,
- artifact paths come from `workspace_root` + `research-validation-results/` and the repo-local pointer, never from repo-root artifact guessing,
- the current stop conditions for this run,
- the current rollback strategy in use,
- the active pivot/refine escalation thresholds when they matter,
- the research workflow's conclusion boundary.

## Closeout Order

For research validation execution, the closeout order is:

1. finish the registered experiment,
2. create the scoped trial commit(s),
3. run verify and guard,
4. remove generated verify/guard byproducts such as cache files,
5. decide support/refute/inconclusive/crash and apply approved rollback for non-kept trials,
6. record the current clean HEAD commit(s) through the helper,
7. only then choose the next validation question.

Do not treat logging as optional bookkeeping.
