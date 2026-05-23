---
name: codex-autoresearch
description: "Research idea validation for Codex CLI. Use when the user proposes a new research, ML, algorithm, prompt, data-processing, benchmark, or experimental idea and wants Codex to turn it into an executable validation run with hypotheses, baselines/controls, ablations, leakage guards, repeated runs when needed, negative-result logging, and supervised serial execution in the current Codex session. Do not use for ordinary one-shot coding help, generic bug fixing, deployment, or casual Q&A."
---

# codex-autoresearch

Autonomous research validation. Hypothesis -> Baseline/control -> Experiment -> Verify -> Evidence -> Repeat.

## When Activated

1. Classify the request as `research` or `plan`, and parse any inline config from the prompt.
2. Load `references/core-principles.md` and `references/structured-output-spec.md`. For active validation runs (`research`), also load `references/runtime-hard-invariants.md`.
3. Load only the additional references the current situation needs:
   - `references/session-resume-protocol.md` for every interactive launch or existing-run control path, before deciding fresh vs resumable
   - `references/environment-awareness.md` before choosing hardware-sensitive work
   - `references/interaction-wizard.md` for every new interactive research launch before execution begins
   - `references/results-logging.md` only when debugging TSV/state semantics or helper behavior directly
4. Load the selected mode workflow reference plus only the detailed cross-cutting protocols that actually apply (`lessons`, `pivot`, `health-check`, `web-search`, `hypothesis-perspectives`). Active runs are serial; do not load or use the parallel experiments protocol during normal execution.
5. Use the bundled helper scripts when stateful artifacts are involved. Resolve them relative to the loaded skill bundle root (`<skill-root>/scripts/...`), not the target repo root. In the common repo-local install this means commands such as `python3 .agents/skills/codex-autoresearch/scripts/autoresearch_init_run.py --repo <primary_repo> --workspace-root <workspace_root> ...`. New-run helpers (`autoresearch_init_run.py`) require both `--repo <primary_repo>` and `--workspace-root <workspace_root>`. Existing-run helpers (`autoresearch_resume_check.py`, `autoresearch_resume_prompt.py`, `autoresearch_supervisor_status.py`, `autoresearch_health_check.py`) require `--repo <primary_repo>` and resolve the workspace-owned Results directory from the repo-local pointer plus canonical context. `autoresearch_launch_gate.py --repo <primary_repo>` is the pre-wizard gate: it returns `fresh` for a clean repo with no prior artifacts and otherwise uses the same pointer/context recovery path.
6. Execute the selected workflow exactly as written and produce the required structured output and artifacts.

## Core Loop

1. Register the hypothesis and expected evidence before changing code.
2. Identify baseline/control and leakage guards.
3. Establish the baseline measurement.
4. Make one focused experimental change or ablation.
5. Verify with a command.
6. Decide whether the evidence supports, refutes, or is inconclusive for the hypothesis.
7. Log positive and negative results.
8. Repeat only when the next experiment answers a concrete validation question.

## Modes

| Mode | Purpose | Primary Reference |
|------|---------|-------------------|
| `research` | Validate a research idea with controlled experiments | `references/research-validation-workflow.md` |
| `plan` | Convert a vague idea into a launch-ready validation plan | `references/plan-workflow.md` |

Use `Mode: <name>` in the prompt to force a specific subworkflow.

## Required Config

For a research validation run, the following fields are needed internally. Codex infers them from the user's natural language input and repo context, then fills gaps through guided conversation:

- `Idea`
- `Hypothesis`
- `Expected evidence`
- `Baseline/control`
- `Scope`
- `Metric`
- `Direction`
- `Verify`
- `Leakage guard`

Optional but recommended:

- `Guard`
- `Ablations`
- `Repeated runs / seeds`
- `Iterations`
- `Run tag`
- `Stop condition`

For every new interactive run, use the wizard contract in `references/interaction-wizard.md`.

## Interactive Supervised Runs

- Use `$codex-autoresearch` for interactive research-validation launches and follow-up controls.
- For a new interactive run, scan the repo, ask the confirmation questions, and start a single supervised run in the current Codex session after the user explicitly approves execution with `go`.
- Keep the operator-facing session and the active validation run in the current Codex thread. Do not delegate to another agent or spawn worker agents. The current session owns hypothesis registration, helper-script calls, commits, verification, negative-result logging, milestone summaries, and conclusion summaries.
- Carry forward the confirmed idea, hypothesis, expected evidence, baseline/control, scope, metric, direction, verify/guard commands, leakage guard, ablations, repeated-run policy, workspace root, primary repo, companion repos, rollback policy, and runtime checklist. Keep running until a validation stop condition, blocker, iteration cap, or user interrupt.
- Ignore external-agent availability; this skill always executes the validation protocol directly in the current session.
- When model-visible goal tools are available, use the official Codex goal only as the parent thread's continuation anchor: after launch approval, call `get_goal`; reuse a matching non-complete current goal, or call `create_goal` with the confirmed objective when no goal exists. If an existing goal cannot be reused, surface it in the confirmation summary before launch and do not create a second one. Mark the goal complete with `update_goal` only when the autoresearch stop condition is actually satisfied.
- Interactive continuation uses the same current-session supervised path.
- Treat the repo where the run starts as the **primary repo**. Single-repo runs are the default. If the task truly spans multiple codebases, declare **companion repos** explicitly and give each repo its own scope instead of stuffing absolute paths into one mixed scope string.
- For a new interactive run, default the `workspace_root` from the launch context: if Codex started inside a git repo, use that repo root; otherwise use the current launch directory. Do not silently widen to a parent workspace just because sibling repos or old artifacts exist. Only widen when the user explicitly confirms a broader multi-repo workspace, and show the resulting `Results directory` in the confirmation summary.
- For every interactive launch that proceeds past the session-resume gate, check `python3 <skill-root>/scripts/autoresearch_hooks_ctl.py status` and then follow the readiness flow in `references/interaction-wizard.md`. Capture the first `startup_tip_needed` value from that status; if it is true, include one product-facing launch tip in the confirmation summary. If setup is missing, stale, disabled, or untrusted, run `python3 <skill-root>/scripts/autoresearch_hooks_ctl.py install` before clarification continues. Treat setup details as internal preparation unless a setup failure blocks launch. Use model-visible goal tools when they are actually available.
- For `status`, `stop`, or `resume` requests, stay on the same skill entry and inspect the workspace-owned artifacts plus any active supervised run.

## Hard Rules

1. **Ask before act for new interactive launches.** For `research`, scan the repo, run the session-resume launch gate, and ask at least one repo-grounded confirmation round before the run starts. Load and follow `references/interaction-wizard.md` for every new interactive launch.
2. **Use the direct supervised path after launch approval.** In interactive modes, once the user says "go" (or equivalent: "start", "launch", or any clear approval), keep the current session available for both monitoring and execution. Do not delegate the validation run to another agent.
3. **Never ask after the user approves the run.** Once the user has approved `go`, do not pause mid-run to ask anything -- not for clarification, not for confirmation, not for permission. If ambiguity appears during the validation run, apply best practices and keep going. The user may be asleep.
4. Read all in-scope files before the first write.
5. One focused experimental variable per iteration unless a factorial design is explicitly approved before launch.
6. Mechanical verification only.
7. After launch approval, scoped per-iteration trial commits are part of the approved run; do not ask separately before creating them. Create a trial commit before verification only when every managed repo's worktree stays within that repo's declared scope or autoresearch-owned artifacts, remove generated verify/guard byproducts, apply the approved keep/discard closeout, then record the current clean HEAD commit(s). The current-session runner must honor the same scope-aware gate before every trial commit.
8. Never stage or revert unrelated user changes.
9. Keep run artifacts uncommitted and never stage them.
10. Use the rollback strategy approved during setup. In a dedicated experiment branch/worktree with pre-launch approval, `git reset --hard HEAD~1` is allowed; otherwise use `git revert --no-edit HEAD`.
11. Do not chase metric gains without evidence for the registered hypothesis. Treat unsupported gains, regressions, null results, crashes, and inconclusive measurements as valid research outcomes and log them.
12. Unlimited runs by default unless the user explicitly asks for `Iterations: N`.
13. Do not ask "should I continue?". Once launched, keep the supervised run active until interrupted or a hard blocker / configured terminal condition appears (see `references/autonomous-loop-protocol.md` Stop Conditions for the full definition).
14. During active execution, keep `references/runtime-hard-invariants.md` as the primary runtime checklist. Core persistent artifacts are `autoresearch-results/results.tsv`, `autoresearch-results/state.json`, `autoresearch-results/context.json`, and `autoresearch-results/lessons.md`.
15. When results are repeatedly inconclusive, use the PIVOT/REFINE escalation ladder from `references/pivot-protocol.md` to change validation strategy rather than brute-force optimizing.
16. Prefer the bundled helper scripts over hand-editing `autoresearch-results/results.tsv`, `autoresearch-results/state.json`, `autoresearch-results/context.json`, or other run artifacts. Always call them via the skill-bundle path (`<skill-root>/scripts/...`); never call bare `scripts/autoresearch_*.py` from the target repo root unless the skill bundle itself is actually installed there.
17. After any context compaction event (the CLI warns about thread length and compaction), re-read `references/runtime-hard-invariants.md`, `references/core-principles.md`, and the selected mode workflow from disk before the next iteration. Do not rely on memory of those documents after compaction.
18. Every 10 iterations, perform the Protocol Fingerprint Check defined in `references/runtime-hard-invariants.md`. Use Phase 8.7 of `references/autonomous-loop-protocol.md` only for the detailed re-anchoring procedure. If any item fails, re-read all loaded runtime docs from disk before continuing.

## Structured Output

Every mode should follow `references/structured-output-spec.md`.

Minimum requirement:

- for interactive and user-facing modes, print a setup summary before the validation run starts,
- for interactive and user-facing modes, print progress updates during the validation run,
- for interactive and user-facing modes, print a completion summary at the end,
- write the mode-specific output files when the workflow defines an output directory.

## Quick Start

```text
$codex-autoresearch
I have a new retrieval reranking idea and want to validate whether it improves recall without leaking labels
```

```text
$codex-autoresearch
Test whether adding a contrastive loss improves validation F1 on this training script
```

```text
$codex-autoresearch
Run an ablation to see whether the new data augmentation actually helps
```

Codex scans the repo, asks targeted questions to clarify the hypothesis, controls, metrics, guards, and repeat policy, then starts a supervised validation run after you approve. You never need to write key-value config.

## References

- `references/core-principles.md`
- `references/runtime-hard-invariants.md`
- `references/research-validation-workflow.md`
- `references/autonomous-loop-protocol.md`
- `references/interaction-wizard.md`
- `references/structured-output-spec.md`
- `references/plan-workflow.md`
- `references/results-logging.md`
- `references/lessons-protocol.md`
- `references/pivot-protocol.md`
- `references/web-search-protocol.md`
- `references/environment-awareness.md`
- `references/session-resume-protocol.md`
- `references/health-check-protocol.md`
- `references/hypothesis-perspectives.md`
