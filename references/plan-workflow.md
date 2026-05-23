# Plan Workflow

Use this mode when the user has a research idea but not yet a launch-ready validation plan.

## Purpose

Convert a vague idea into a concrete validation plan that can be launched as `Mode: research`.

## Trigger

- `$research-validation-skill Mode: plan`
- "help me validate this idea"
- "turn this research idea into an experiment"
- "what should I run to test this?"

## Planning Steps

1. Scan the repo for existing training/eval scripts, configs, metrics, data splits, and benchmark entrypoints.
2. State the hypothesis in one sentence.
3. Identify the baseline/control.
4. Choose the primary metric and direction.
5. Propose the verify command and guard command.
6. Define the leakage guard.
7. Define the ablation boundary.
8. Define the repeat policy only when needed.
9. Estimate runtime/resource risks.
10. Produce a launch-ready summary.

## Output

Use concise sections:

- Idea
- Hypothesis
- Baseline/control
- Metric
- Verify
- Leakage guard
- Ablation
- Repeat policy
- Launch summary

Do not edit code in plan mode unless the user explicitly says to launch.

If the user says `launch`, switch to `Mode: research` with the confirmed plan and follow `interaction-wizard.md`.
