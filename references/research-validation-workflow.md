# Research Validation Workflow

Use this workflow to validate a proposed research idea with controlled, mechanically verified experiments.

This is the thin execution guide for active runtime work. Load `autonomous-loop-protocol.md` for Phase 0 on every fresh launch, resume boundary, or recovery decision. Once the run is active, keep `runtime-hard-invariants.md` in memory and reopen this file before choosing the next experiment.

## Purpose

Turn a research idea into evidence:

1. state the hypothesis,
2. identify baseline/control,
3. run one controlled experiment,
4. verify mechanically,
5. record support/refutation/inconclusive evidence,
6. decide the next validation question.

The objective is not to maximize a metric at all costs. The objective is to learn whether the idea is supported under the confirmed conditions.

## Before Launch

- Use `interaction-wizard.md` for every new launch.
- Use `session-resume-protocol.md` before deciding whether the run is fresh or resumable.
- Use `environment-awareness.md` before choosing hardware-sensitive work.
- Confirm a launch summary that includes:
  - idea and hypothesis,
  - expected evidence,
  - baseline/control,
  - scope,
  - metric and direction,
  - verify command,
  - leakage guard or reason it is not needed,
  - ablation plan,
  - repeated-run policy when the metric is noisy.

## Runtime Cycle

1. Read the current in-scope context, recent results rows, lessons, and retained state.
2. If no baseline exists yet, measure the baseline/control and initialize `autoresearch-results/results.tsv` plus `autoresearch-results/state.json`.
3. Register the next validation question in one sentence before editing.
4. Make one focused experimental change or one ablation.
5. Create the scoped trial commit when the workspace is safe to isolate.
6. Run verify, then guard and leakage checks.
7. If the repeat policy requires multiple seeds/runs, execute the confirmed repetitions before deciding.
8. Decide `support`, `refute`, or `inconclusive` in the row description. Map the TSV status as:
   - `keep` when evidence supports the hypothesis and passes guards,
   - `discard` when evidence refutes the hypothesis or the change is unsupported,
   - `crash` when verification cannot produce usable evidence,
   - `no-op` when no actual experiment was run.
9. Record the result through `autoresearch_record_iteration.py` using the current clean HEAD after closeout.
10. Only after the result is recorded, choose the next validation question.

## Research Discipline

- Hypothesis registry: every attempted experiment must start from a named hypothesis or ablation question.
- Control/baseline: never compare only against the immediately previous failed trial when a stable baseline/control exists.
- Repeated runs: use repeated seeds/runs when the metric is known or observed to be noisy; do not invent statistical tests when the user did not ask for them.
- Leakage guard: protect dataset splits, test labels, benchmark holdouts, prompt/eval contamination, and train/validation boundaries.
- Negative result logging: null results, regressions, failed ablations, and inconclusive measurements are first-class outputs.
- Ablation discipline: change one variable at a time unless the user approves a factorial design before launch.
- Conclusion boundary: summaries must state what was tested, under which command/data/scope, and what remains unvalidated.

## Stop Conditions

Stop when one of these happens:

- the idea is supported by the confirmed evidence rule,
- the idea is refuted by the confirmed evidence rule,
- the evidence remains inconclusive after the configured budget,
- the user interrupts,
- the configured iteration cap is reached,
- or a true blocker appears.
