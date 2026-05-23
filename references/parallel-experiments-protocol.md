# Parallel Experiments Protocol

Parallel experiment execution is disabled for this skill.

Active `codex-autoresearch` runs are serial and execute in the current Codex session. Do not spawn workers, delegate to other agents, create worker worktrees for concurrent validation, or ask the user to approve a parallel mode.

If several hypotheses need validation, test them sequentially through the normal runtime cycle:

1. Register one validation question.
2. Make one focused change.
3. Verify mechanically.
4. Log the result.
5. Decide the next validation question from the evidence.

This file remains only as a compatibility placeholder for older references and helper artifacts.
