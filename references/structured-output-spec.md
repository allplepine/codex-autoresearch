# Structured Output Specification

Every `codex-autoresearch` research-validation run must produce predictable user-facing output and predictable artifacts.

## Status Values

All runs share the TSV statuses defined in `references/results-logging.md`:

| Status | Research Meaning |
|--------|------------------|
| `baseline` | Baseline/control measurement |
| `keep` | Evidence supports the registered hypothesis and passes guards |
| `discard` | Evidence refutes the hypothesis, is unsupported, or fails guards |
| `crash` | Verification crashed or produced unusable evidence |
| `no-op` | No actual experiment was run |
| `blocked` | A blocker prevents valid validation |
| `refine` | Validation strategy refined within the same idea |
| `pivot` | Validation strategy changed because evidence remained inconclusive |
| `search` | External information gathered to form a new validation hypothesis |
| `drift` | Metric drifted from expected value during session resume |

Use the row description to record `support`, `refute`, or `inconclusive`.

## Common Sections

Before launch:

1. `Setup`
2. `Validation Plan`
3. `Baseline/Control`

During work:

1. `Hypothesis`
2. `Experiment`
3. `Evidence`
4. `Decision`

At completion:

1. `Conclusion`
2. `Evidence Summary`
3. `Artifacts`
4. `Unvalidated Questions`

## Iteration Line

Use this shape during validation:

```text
[iteration N] hypothesis -> experiment -> evidence -> support/refute/inconclusive
```

Examples:

```text
[iteration 1] contrastive loss should improve macro F1 -> loss-only ablation -> +1.8 F1, guard pass -> support
[iteration 2] stronger augmentation should help -> augmentation-only ablation -> -0.6 F1 -> refute
[iteration 3] retrieval temperature may matter -> temp sweep crashed on OOM -> crash
```

## Research Output

Required completion summary:

- idea,
- hypothesis tested,
- baseline/control,
- metric and verification command,
- best/current evidence,
- support/refute/inconclusive decision,
- negative results count,
- leakage guard outcome,
- ablations completed,
- repeat policy actually used,
- remaining unvalidated questions,
- artifact paths.

Artifacts:

- `autoresearch-results/results.tsv`
- `autoresearch-results/state.json`
- `autoresearch-results/context.json`
- `autoresearch-results/lessons.md` if lessons were extracted

Optional human-readable research closeout:

```text
autoresearch-results/research/{YYMMDD}-{HHMM}-{slug}/
  validation-plan.md
  evidence-summary.md
  negative-results.md
  conclusion.md
```

Create the optional closeout only when useful or requested; the TSV/state artifacts remain authoritative.

## Logging Rules

- TSV headers must be written exactly once.
- Always record the baseline/control before treatment experiments.
- Record negative and inconclusive outcomes, not just supportive metric changes.
- Row descriptions should mention the hypothesis or ablation being tested.
- Workspace-owned artifact metadata should use canonical paths. `context.json` and state config fields store absolute paths so resume and status helpers can resolve the active run without cwd guessing.
- Final summaries should reference every artifact created.
- Active runs use normal iteration lines only. Preserve legacy prefixes only when summarizing pre-existing historical rows.
