# Interaction Wizard Contract

This file defines how Codex should collect missing information for research-validation runs.

When this file mentions `<skill-root>`, it means the directory containing the loaded `SKILL.md`.

## Goal

The user may provide only a rough research idea. Codex should scan the repo, infer the experimental setup, and ask a short confirmation round before launch. The user should not need to know internal field names.

## Global Rules

1. Accept natural language ideas such as "test whether this augmentation helps" or "validate this reranking idea".
2. Scan the repo before asking anything: read training/eval scripts, configs, datasets/splits metadata, benchmark scripts, metrics, and relevant model/code paths.
3. Ask at least one repo-grounded confirmation round before launch.
4. Keep clarification to 1-3 rounds unless a real blocker remains.
5. Always confirm hypothesis, baseline/control, metric, verify command, leakage guard, ablation boundary, and repeat policy.
6. Do not require statistical decision rules. Use repeats/seeds only when the metric is noisy, the repo already supports them, or the user asks.
7. Present a structured confirmation summary before launching.
8. End the confirmation summary with a clear call to action that explicitly authorizes a supervised current-session run.
9. After launch approval, start from the same skill entrypoint. The current Codex session performs the validation run directly and reports concise milestone summaries.
10. After the launch gate allows a fresh or confirmed launch, check `python3 <skill-root>/scripts/autoresearch_hooks_ctl.py status`. If setup is missing, stale, disabled, or untrusted, run `python3 <skill-root>/scripts/autoresearch_hooks_ctl.py install` before clarification continues. Treat setup details as internal preparation unless a setup failure blocks launch.

## Clarification Protocol

### Step 1: Scan

Identify:

- code path touched by the idea,
- existing baseline/control command,
- evaluation command and metric output,
- dataset split or benchmark boundary,
- seed/repeat support,
- risks of leakage or benchmark contamination,
- whether the idea needs an ablation rather than a direct replacement.

After the primary repo is known, run:

```bash
python3 <skill-root>/scripts/autoresearch_launch_gate.py --repo <primary_repo>
```

Follow `session-resume-protocol.md` before deciding fresh vs resumable.

### Step 2: Guided Questions

Ask only what is blocking launch. Good research-validation questions include:

- "I see `eval.py` reports macro F1 on `data/val.json`. Should the baseline be the current model on that split?"
- "Your idea changes both retrieval scoring and prompt format. Should I ablate retrieval first, prompt second, or approve a combined test?"
- "I found seed support via `--seed`. Should I run 3 seeds for noisy validation, or one run as a smoke validation?"
- "The benchmark has a held-out test file. Should I guard against touching it and use validation only?"
- "If the metric improves but the ablation suggests the mechanism is wrong, should I keep investigating rather than mark the idea supported?"

Rules:

- Never ask for novelty judgment as part of the mechanical run.
- Do not silently choose the test set as the decision target when a validation split exists.
- When repeated runs are expensive, default to one baseline and one treatment run unless noise is already evident.
- If leakage risk cannot be controlled, do not launch; report the blocker.

### Step 3: Confirmation Summary

Use the user's language. Keep it compact.

```text
**Confirmed**
- Idea: add contrastive loss to improve validation F1
- Hypothesis: contrastive loss improves macro F1 without using validation labels in training
- Baseline/control: current training script on `configs/base.yaml`
- Results directory: `./research-validation-results/`
- Metric: validation macro F1, direction: higher
- Verify: `python train.py --config configs/base.yaml && python eval.py --split val`
- Leakage guard: do not edit `data/val*` or test labels; train split only
- Ablation: contrastive loss only; no data augmentation changes
- Repeat policy: 3 seeds if a single run changes F1 by < 1 point, otherwise one run

**Need to confirm**
- Any stricter guard beyond the validation split boundary?

**Runtime checklist**
- Register hypothesis before each experiment.
- Baseline/control first, then initialize results/state.
- Log support, refutation, and inconclusive/negative results.

**Next step**
- Reply "go" to authorize the supervised run to start in this session, or tell me what to change.
```

Format rules:

1. Always show the Results directory.
2. Always show the leakage guard or explicitly say why none is needed.
3. Always show ablation boundary and repeat policy.
4. Do not show raw internal field names unless the user already used them.
5. Keep "Need to confirm" for genuine blockers only.

## Launch Handoff

When the user replies with launch approval:

1. Ensure setup check is complete.
2. Align the official Codex goal when model-visible goal tools are available.
3. Continue in the current Codex session with the confirmed idea, hypothesis, baseline/control, metric, direction, verify command, leakage guard, ablation boundary, repeat policy, workspace paths, selected references, and runtime checklist.
4. Use the helper scripts for `results.tsv`, `state.json`, and `context.json`, and log negative and inconclusive outcomes.
5. Do not spawn workers or delegate the validation run. The current session performs the run directly and reports concise milestone summaries.

## Internal Field Mapping

The wizard internally maps the conversation to:

- Idea: the user's proposed research idea.
- Hypothesis: what should be true if the idea works.
- Expected evidence: what observed outcome would support or refute the hypothesis.
- Baseline/control: the current system, previous method, config, or no-treatment condition.
- Scope: files/configs/data-processing paths allowed to change.
- Metric: primary validation metric.
- Direction: higher/lower.
- Verify: command that produces the metric.
- Leakage guard: files, splits, labels, benchmark data, or prompt/eval materials that must not be touched.
- Ablation: the single variable being tested, or approved factorial design.
- Repeat policy: number of seeds/runs or condition that triggers repeats.
- Guard: pass/fail regression check.
- Iterations: validation budget.

## Validation Rules

Before launch, silently validate:

- scope resolves to real files,
- baseline/control can be measured,
- verify command is runnable and emits a parseable metric,
- leakage guard is enforceable,
- ablation boundary is specific enough,
- repeat policy is feasible in the environment,
- guard command is pass/fail only and already passes at baseline.

If validation fails, explain the blocker in plain language and suggest the smallest fix.

## Mini-Wizard

When `session-resume-protocol.md` detects a prior run with valid `state.json` but inconsistent TSV:

1. Show prior run tag, iteration count, best/current metric, and last status.
2. Ask exactly one question:
   - resume from JSON state, or
   - start fresh and archive old artifacts.
3. If resuming, show a condensed confirmation summary from JSON config.
4. The user replies "go" and the supervised validation run resumes immediately in the current session.
