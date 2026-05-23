# Results Logging

This is the detailed reference for TSV/state semantics. During normal loop execution, treat `autoresearch_record_iteration.py` as the authoritative closeout step instead of reopening this file.

## Workspace-Owned Results Directory

Default user-visible directory:

```text
<workspace_root>/generic-supervised-results/
```

Fixed files:

```text
results.tsv
state.json
lessons.md
context.json
```

### `context.json` Schema

`context.json` is the canonical run context written by `autoresearch_workspace.py`. It replaces the former `autoresearch-hook-context.json` and serves as the single source of truth for resume and status helpers to locate the active run's artifacts.

```json
{
  "version": 2,
  "active": true,
  "session_mode": null,
  "workspace_root": "/abs/path/to/workspace",
  "artifact_root": "/abs/path/to/workspace/generic-supervised-results",
  "primary_repo": "/abs/path/to/repo",
  "repo_targets": [
    {"path": "/abs/path/to/repo", "scope": "src/**/*.ts", "role": "primary"},
    {"path": "/abs/path/to/companion", "scope": "lib/**/*.py", "role": "companion"}
  ],
  "verify_cwd": "workspace_root",
  "results_path": "/abs/path/to/workspace/generic-supervised-results/results.tsv",
  "state_path": "/abs/path/to/workspace/generic-supervised-results/state.json",
  "launch_path": null,
  "runtime_path": null,
  "log_path": null,
  "updated_at": "2026-04-15T12:00:00Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `version` | `int` | Schema version, currently `2` |
| `active` | `bool` | Whether this run context is active |
| `session_mode` | `string \| null` | Compatibility marker for helper state; do not expose to users |
| `workspace_root` | `string` | Absolute path to the workspace root |
| `artifact_root` | `string` | Absolute path to `generic-supervised-results/` |
| `primary_repo` | `string` | Absolute path to the primary git repo |
| `repo_targets` | `array` | List of managed repos with path, scope, and role |
| `verify_cwd` | `string \| null` | `"workspace_root"` or `"primary_repo"` |
| `results_path` | `string` | Absolute path to `results.tsv` |
| `state_path` | `string` | Absolute path to `state.json` |
| `launch_path` | `string \| null` | Reserved for legacy runtime state; normally `null` |
| `runtime_path` | `string \| null` | Reserved for legacy runtime state; normally `null` |
| `log_path` | `string \| null` | Reserved for legacy runtime state; normally `null` |
| `updated_at` | `string` | ISO 8601 UTC timestamp |

Each managed repo also stores a repo-local pointer at `.generic-supervised-skill/pointer.json` that references back to the workspace-owned `context.json`.

Add a direction comment at the top:

```text
# metric_direction: higher
```

or

```text
# metric_direction: lower
```

## Header Comments

The first comment line declares the metric direction. Additional comment lines may include:

```text
# environment: cpu=8 ram=16384MB gpu=A100(40GB) python=3.11 container=docker
# metric_direction: lower
# mode: loop
# run_tag: any-types-v2
# web_search: enabled
```

## Generic Schema

```tsv
iteration	commit	metric	delta	guard	status	description
```

## Columns

| Column | Meaning |
|--------|---------|
| `iteration` | Integer main iteration counter starting at `0` for the baseline. Active runs write only integer rows. Legacy rows from older versions may use suffix notation (`5a`, `5b`, `5c`) and are ignored for retained-state replay. |
| `commit` | Short hash for the primary repo's clean HEAD after the iteration is closed out. For `keep`, this is the trial commit. For reverted `discard` or `crash` rows, this is the rollback/restored HEAD. Use `-` only for meta rows that did not test a committed trial (for example `pivot`, `search`, or a strategy-only `refine`) |
| `metric` | Parsed metric value for that row's attempt or recalibration |
| `delta` | `metric - retained_metric_before_row` |
| `guard` | `pass`, `fail`, or `-` |
| `status` | See Status Values below |
| `description` | One-sentence explanation of the iteration. Structured keep/stop-gating labels may prefix the sentence as `[labels: foo, bar] ...` |

For multi-repo runs, the TSV `commit` column still records the **primary repo** closeout commit. Per-repo commit provenance for companion repos lives in `state.json` (`state.last_repo_commits` and `state.last_trial_repo_commits`) so the primary audit trail stays compact while the JSON snapshot preserves cross-repo detail.

## Metrics And Acceptance Contract

The metrics model is intentionally small:

- `verify_format = scalar | metrics_json`
- `primary_metric_key`
- `acceptance_criteria`
- `required_keep_criteria`

`acceptance_criteria` and `required_keep_criteria` are lists of criterion objects:

```json
[
  {"metric_key": "accuracy", "operator": ">=", "target": 0.9}
]
```

Do not use legacy `metric` / `op` / `value` fields, an `all` wrapper, or the `!=` operator.

When `verify_format=scalar`, the verify command must emit a single numeric metric as its final non-empty output line. Do not heuristically scrape banner text, earlier lines, or arbitrary regex matches during the loop. If the command is noisy, tighten the verify command during setup so the final line is mechanically parseable.

When `verify_format=metrics_json`, the verify command must print a JSON object as its final non-empty output line. That JSON object is the metrics map used by the helpers. It must include `primary_metric_key` plus every metric referenced by `acceptance_criteria` and `required_keep_criteria`. Helpers must not synthesize missing metrics from the scalar primary metric in this mode.

`results.tsv` records only the primary metric. Structured metrics and acceptance states live in `state.json`.

## Structured Labels For Keep / Stop Gating

Some goals need more than a numeric threshold. Example: "Only retain improvements from the production path, and stop only when latency <= 120 ms and the retained keep uses the required production path and real backend."

For those runs:

- persist `config.required_keep_labels` when retention itself has a structural requirement
- persist `config.required_stop_labels` in JSON config/state
- record structured iteration labels with `autoresearch_record_iteration.py --label ...`
- let the helper write a canonical TSV prefix like:

```text
[labels: production-path, real-backend] optimized query path preserved real backend behavior
```

Would-be `keep` rows that miss `required_keep_labels` are mechanically downgraded to `discard` before they can update retained state.

The supervisor only treats a retained result as terminal when the configured final gates are satisfied:

- if `acceptance_criteria` is configured, the retained result satisfies it,
- if `stop_condition` is configured, the retained result satisfies it,
- the retained keep labels cover every `required_stop_labels` entry.

This keeps causal or implementation-specific success criteria machine-checkable instead of leaving them in free-form prose.

## Status Values

| Status | Meaning |
|--------|---------|
| `baseline` | Initial measurement before any changes |
| `keep` | Change improved the metric and passed guard |
| `discard` | Change did not improve or failed guard |
| `crash` | Verification crashed or produced an error |
| `no-op` | No actual diff was produced |
| `blocked` | Hard blocker encountered, loop stopped |
| `refine` | Strategy adjustment within current approach (see `pivot-protocol.md`) |
| `pivot` | Strategy abandoned, fundamentally new approach (see `pivot-protocol.md`) |
| `search` | Web search performed for external knowledge (see `web-search-protocol.md`) |
| `drift` | Metric drifted from expected value during session resume |

## Example

```tsv
# metric_direction: lower
iteration	commit	metric	delta	guard	status	description
0	a1b2c3d	14	0	-	baseline	current pytest failure count
1	b2c3d4e	9	-5	pass	keep	reduce fixture startup overhead
2	c3d4e5f	11	+2	-	discard	expand retries in API client
3	d4e5f6a	0	0	-	crash	refactor parser with bad import
4	e5f6a7b	9	0	fail	discard	inline auth cache but break regression guard
```

## Legacy Suffix Rows

Active runs must write one authoritative integer row per completed iteration. Older result logs may contain suffix rows such as `5a`, `5b`, or `5c`; keep these as historical audit detail, but do not create new suffix rows during current-session serial execution.

Only integer rows (`0`, `1`, `2`, `5`) define the retained state.

## Helper Scripts

Prefer the bundled helper scripts for stateful artifact updates:

These helper scripts live in the skill bundle. Do not confuse them with the target repo's own `scripts/` directory.

Define `<skill-root>` as the directory that contains the loaded `SKILL.md`. In the common repo-local install this is usually `.agents/skills/generic-supervised-skill`, so the exact command becomes `python3 .agents/skills/generic-supervised-skill/scripts/...`.

- `python3 <skill-root>/scripts/autoresearch_init_run.py --repo <primary_repo> --workspace-root <workspace_root> ...`
  Initializes `generic-supervised-results/results.tsv` and `generic-supervised-results/state.json` together from the baseline measurement, writes canonical `context.json`, and writes repo-local pointers for every managed repo. Interactive supervised runs use the helper default session marker for compatibility. Multi-repo runs may add repeated `--repo-commit PATH=COMMIT` flags to persist companion-repo baseline provenance in JSON state. Runs with structural success criteria may add repeated `--required-keep-label LABEL` flags to protect retained state and repeated `--required-stop-label LABEL` flags so the supervisor only stops when the retained keep also carries those labels.
- `python3 <skill-root>/scripts/autoresearch_set_session_mode.py --repo <repo> ...`
  Legacy internal helper for synchronizing old interactive state. Normal supervised skill flow should not expose or call it.
- `python3 <skill-root>/scripts/autoresearch_record_iteration.py ...`
  Appends one authoritative main iteration row and updates JSON state atomically. Multi-repo runs may add repeated `--repo-commit PATH=COMMIT` flags to update companion-repo commit provenance while the TSV `commit` column continues to track the primary repo. Repeated `--label LABEL` flags record structured keep/stop-gating labels on the attempted row and retained state.
- `python3 <skill-root>/scripts/autoresearch_resume_check.py --repo <repo>`
  Reconstructs retained state from the TSV and decides `full_resume`, `mini_wizard`, `tsv_fallback`, or `fresh_start`.
- `python3 <skill-root>/scripts/autoresearch_supervisor_status.py --repo <repo>`
  Computes whether the supervised run should continue, stop, or ask for human help after a finished turn.

## Rules

- Create the log only after the baseline metric is known.
- Record every completed experiment before starting the next one.
- In normal loop execution, do that closeout through the bundled helper scripts rather than by hand.
- Append after every iteration, including crashes, no-ops, refines, pivots, and searches.
- Never commit the Results directory.
- Treat `generic-supervised-results/` and repo-local pointers as autoresearch-owned artifacts: leave them unstaged and ignore them when checking experiment scope.
- Re-read the latest entries before choosing the next idea.
- The standalone health-check helper reports warnings/blockers as JSON. Append a TSV row only when the runtime explicitly decides to log a blocker or recovery event.

## Cross-Validation with JSON State

`generic-supervised-results/state.json` is the primary recovery source for session resume (see `references/session-resume-protocol.md`). The TSV log and the JSON state file serve complementary roles:

| Aspect | `generic-supervised-results/results.tsv` | `generic-supervised-results/state.json` |
|--------|----------------------|--------------------------|
| **Purpose** | Full audit trail of every iteration | Compact snapshot for fast resume |
| **Content** | One main row per iteration, plus legacy suffix rows if imported from older runs | Aggregated counters and config |
| **Recovery role** | Fallback when JSON is missing | Primary recovery source |
| **Cross-validation** | Reconstruct retained state from integer main rows | Must match the reconstructed retained state |

### Consistency Rules

- **Main iteration match:** `state.iteration` must equal the highest integer iteration label in the TSV.
- **Retained metric match:** `state.current_metric` must equal the retained metric after replaying the integer main rows. After a `discard`, the TSV row records the attempted metric, but `state.current_metric` stays at the last kept metric.
- **Last trial match:** `state.last_trial_metric` must equal the metric on the latest integer main row.
- **Multi-repo provenance:** when `state.last_repo_commits` or `state.last_trial_repo_commits` are present, they are auxiliary JSON-only provenance keyed by repo path. They are not reconstructed from the TSV and therefore do not participate in TSV/JSON consistency blocking.
- **Legacy suffix-row tolerance:** Rows such as `5a`, `5b`, and `5c` are ignored for `state.iteration` matching. They provide audit detail only.

During session resume, `python3 <skill-root>/scripts/autoresearch_resume_check.py --repo <repo>` reconstructs the retained state from the TSV and compares it with `generic-supervised-results/state.json`. Any mismatch triggers a mini-wizard rather than a silent full resume.
