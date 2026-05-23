# Hypothesis Perspectives

Structured multi-lens reasoning applied before committing to a validation hypothesis. Not multi-agent -- this is a thinking framework within a single agent.

## When to Apply

**Mandatory:**
- On every hypothesis during the first 5 iterations (exploring the problem space).
- Immediately after every REFINE or PIVOT decision (perspectives are part of the re-ideation).

**Optional:**
- When the last 3 iterations were all discards but REFINE has not yet triggered.
- When entering a new strategy family for the first time.

**Skip:**
- For obvious, mechanical measurement setup steps.
- When the hypothesis is a direct continuation of a successful validation strategy (same idea, next ablation).

## The Four Lenses

### 1. Optimist

Ask: "What experiment would produce the clearest evidence right now?"

- Focus on evidence clarity, not just metric gain.
- Prefer experiments that isolate the idea's mechanism.
- Look for controls or ablations that can falsify the idea.

### 2. Skeptic

Ask: "Why might this hypothesis fail?"

- Cross-check with the results log: has a similar approach already been tried and discarded?
- Identify assumptions that could be wrong.
- Consider side effects that could trigger guard or leakage failure.
- Check if the hypothesis depends on conditions that may not hold.

### 3. Historian

Ask: "What do past results and lessons tell me?"

- Consult `research-validation-results/lessons.md` for relevant entries.
- Review the results log for patterns:
  - Which validation strategies produced conclusive evidence?
  - Which files, configs, or datasets were sensitive to changes?
  - What is the typical noise or delta for successful iterations?
- If this is the first run with no history, note "no prior data" and move on.

### 4. Minimalist

Ask: "Is there a smaller ablation that tests the same hypothesis?"

- Can the same evidence be obtained with fewer file changes?
- Can the change be smaller in scope while still testing the core idea?
- Is there a way to achieve 80% of the benefit with 20% of the complexity?
- Would a simpler version be easier to revert if it fails?

## Decision Process

After applying all four lenses:

1. If all lenses agree on the hypothesis, proceed.
2. If the Skeptic raises a concrete concern backed by evidence (prior failure in results log, known side effect from lessons), address the concern before proceeding. Evidence-backed skepticism overrides optimism.
3. If the Historian suggests a better-tested alternative, prefer it unless the Optimist's case is compelling and untried.
4. If the Minimalist offers a simpler ablation that tests the same core idea, prefer the simpler version.
5. **Tie-breaking:** A tie is when 2 lenses favor and 2 oppose (or equivalent ambiguity). In a true tie, the Minimalist wins -- smaller experiments are cheaper to discard. If no Minimalist alternative exists, prefer the untried approach over the retried one.

## Output Format

When perspectives are applied, record the reasoning briefly in the commit message or log description:

```
experiment: [hypothesis] (perspectives: optimist=clear-evidence, skeptic=no leakage risk, historian=new approach, minimalist=single-variable ablation)
```

Do not add perspectives reasoning to the TSV log -- keep it in commit messages only to avoid log bloat.

## Integration Points

- **autonomous-loop-protocol.md (Phase 3):** Apply perspectives before selecting the validation question.
- **lessons-protocol.md:** Historian lens reads from the lessons file.
- **pivot-protocol.md:** Always apply perspectives after a REFINE or PIVOT.
- **Serial execution:** Active runs choose one validation question at a time; apply perspectives before each sequential hypothesis selection when the protocol calls for them.
