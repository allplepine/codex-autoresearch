# Core Principles

These principles define the skill.

## 1. Constraint Enables Autonomy

Autonomous validation works when scope is small enough to fully understand.

- Prefer a bounded file set.
- Prefer a single primary metric plus explicit guards.
- Prefer a fixed iteration cost.

## 2. Humans Set Direction, Agents Execute

The user defines the idea. Codex turns it into testable hypotheses and executes inside the declared boundaries.

## 3. Metrics Must Be Mechanical

If a command cannot produce evidence for support/refutation, the run is not ready.

Good metrics:

- validation F1/accuracy/loss
- benchmark score or throughput
- retrieval recall / MRR / NDCG
- test failure count as a guard
- bundle size
- response latency
- validation metric such as `val_bpb`

Bad metrics:

- "looks better"
- "seems publishable"
- "probably faster"

## 4. Fast Verification Wins

Use the fastest trustworthy check. Slow verification destroys iteration speed.

Prefer:

- targeted tests over full suites
- incremental builds over full rebuilds
- narrow benchmarks over end-to-end manual testing

## 5. One Change Per Iteration

Atomic experiments create causality. If the result changes, the agent knows why.

## 6. Git Is Memory

Kept experiments stay in history. Failed experiments are rolled back using the pre-approved experiment rollback strategy. The results log (`research-validation-results/results.tsv`) records every experiment -- kept or discarded -- as the true audit trail.

## 7. Evidence Beats Optimization

Do not chase metric gains that do not answer the registered hypothesis. Null results, regressions, leakage failures, and inconclusive measurements are valid outputs when they are honestly logged.

## 8. Honest Limits

If permissions, tooling, flakiness, or missing context make the run unsafe, stop and say so. Planning cleanly is better than guessing.
