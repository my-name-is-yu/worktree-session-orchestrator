---
name: worktree-session-orchestrator
description: Coordinate multiple Codex project worktree threads through implementation, source-first parent review, PR readiness, and maintenance. Use when the user wants parallel worktree/session lanes completed with cost-aware GPT-5.6 routing; do not use for ordinary single-lane edits.
---

# Worktree Session Orchestrator

Use the active project context for repository conventions, architecture, tests,
and scope. Do not restate that context here or duplicate it in child prompts;
carry only the lane-specific delta.

## Operating Contract

The parent owns lane design, thread creation, integration judgment, and final
verification. Children implement only inside their Codex-assigned project
worktrees. Source, tests, current PR state, and latest-head CI outrank summaries.

- Keep the parent in the canonical checkout except when inspecting or repairing
  a lane.
- Use separate Codex project worktree threads, not parent-local subagents, for
  implementation lanes. Never ask a child to create another worktree.
- Preserve user changes. Do not force-push unless the batch owns the branch and
  no safer path exists.
- Do not merge. If the user later requests merging, switch to the repository's
  merge workflow or a dedicated merge skill.
- Create a goal only when explicitly requested; complete it only when every lane
  is ready, intentionally deferred, or blocked with evidence.
- Use subagents only as advisory reviewers when an independent audit materially
  helps; the parent retains judgment.

## Model and Prompt Policy

Use GPT-5.6 by default:

| Assignment | Default use |
| --- | --- |
| Luna low | Deterministic edits, inventory, formatting, log reduction, status, heartbeat |
| Terra low | Default implementation, scoped investigation, tests, review fixes |
| Sol low/medium | Parent judgment; ambiguous, cross-cutting, high-risk, or repeatedly failing lanes |

Do not select GPT-5.5 unless a workflow is pinned to it, reproducibility requires
it, or GPT-5.6 is unavailable. Treat GPT-5.4 as a pinned-workflow compatibility
choice and Spark as an optional Pro text-only preview, not a dependency.

Escalate `Luna low -> Terra low -> Terra medium -> Sol low/medium`. Before
escalating, repair missing success criteria, prerequisites, tool routing,
evidence, verification, or stop rules. Escalate from observed failure or risk,
not task duration.

Follow GPT-5.6 prompt guidance:

- State outcome, success criteria, constraints, authority, validation, output,
  and stop rules; let the model choose the implementation path.
- Include only lane-specific context. Reference project instructions and source
  surfaces instead of copying them.
- Use absolute language only for invariants. Omit generic brevity, thoroughness,
  step-by-step, and repeated permission language.
- Parallelize independent reads; keep dependent decisions sequential. Try one
  or two meaningful fallbacks after empty or suspicious results.
- Request a short opening update and sparse phase-change updates, not routine
  command narration. Stop when the success criteria are evidenced.

## Create the Batch

1. Read the user goal, project instructions, relevant source, and overlapping
   sessions or PRs. Split only genuinely separable ownership boundaries.
2. Load thread tools if needed, call `list_projects`, and select the saved Codex
   project. If unavailable, stop; use manual worktrees only with user approval.
3. Create each child with `target.type = "project"`, the selected `projectId`,
   `target.environment.type = "worktree"`, and the chosen model/reasoning.
   Record `threadId` or `pendingWorktreeId`; let Codex assign the path.
4. Track each lane as:

```text
<lane>: <state>, <model>/<effort>, branch <branch>, thread <id>, worktree <path|pending>, PR <url|none>, blocker <none|details>
```

For each lane retain only: goal, success criteria, scope boundary, verification,
PR contract, and any CI gate. Prefer an existing user-led session over a
duplicate lane.

## Child Prompt

Use this compact contract and omit empty fields:

```text
Role: Own <lane> implementation and same-branch PR readiness in <repository>.
Model: <model>, reasoning <effort>.
Project context: Follow the repository instructions and current source already
available in this project. The lane-specific contract below is the delta.

Goal: <outcome>.

Success:
- <observable production result>
- <required removal or migration result>
- <verification>
- commit, push, and open/update a non-draft PR with a readiness summary

Boundaries:
- Work only in this thread's assigned worktree on <branch>; do not create a
  worktree or edit the parent checkout.
- In scope: <scope>. Out of scope: <boundary>.
- Do not preserve replaced behavior through aliases, shims, renamed paths, or
  disconnected helpers unless the project contract requires it.
- Do not merge, run gated heavy CI, or proactively rebase unless the parent
  marks `ready-for-ci` or `base-refresh-needed`, or project rules require it.

Authority: Inspect, edit, test, commit, push, and maintain this PR without asking
again. Stop for unrelated external writes, destructive action beyond requested
tracked-file changes, or material scope expansion.

Execution: First report `pwd`, current branch, and short status; create or switch
to <branch> if needed. Inspect the real production owner/caller before editing.
Run <verification>. Use sparse updates.

Closeout: Return commit SHA, PR URL, verification evidence, and blocker. Stop
when Success is evidenced. After PR creation, fix only actionable review or
latest-head failures owned by this lane; otherwise remain idle.
```

## Monitor and Verify

For a batch expected to span multiple parent turns, create a 5-minute thread
heartbeat unless the user declines: use `automation_update` with
`kind = "heartbeat"`, `destination = "thread"`, and `status = "ACTIVE"`. Keep
its prompt self-contained with the repository, lane table, criteria, live
PR/CI/review checks, and authority to send corrective prompts. Use Luna low when
available. Report only material changes; delete the heartbeat when every lane
is ready, deferred, merged by a later workflow, or blocked on user input.

Use this state flow:

```text
assigned -> implementing -> local-verified -> ready-pr-open -> parent-reviewed -> ready-for-ci -> merge-candidate
exceptions: ci-red-owned | base-refresh-needed | blocked
```

- A ready PR is not automatically CI-ready or merge-ready.
- Mark `base-refresh-needed` only for a ruleset blocker, proven stale-base
  failure, merged dependency, or imminent merge-order need.
- Correct children with: observed gap and evidence, required outcome, fix
  boundary, validation, and stop condition. Do not resend the original prompt.

Before accepting a lane, the parent verifies the actual branch and diff, scope,
production wiring, removal of the old path, absence of orphan or compatibility
surfaces, production-entrypoint tests, targeted verification, non-draft PR, and
current review/check state. Repair the same branch directly only when the child
is unavailable or the user asks the parent to finish.

## Finish

This skill ends at verified PR readiness and maintenance; it does not merge.
Lead with the batch outcome, then give the lane table, PR links, verification,
material blockers or caveats, and next action. Mark an explicit goal complete
only when those facts are true.
