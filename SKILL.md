---
name: worktree-session-orchestrator
description: Coordinate multiple Codex worktree/session implementation lanes through source-first implementation, monitoring, verification, PR readiness, review/CI fixes, and maintenance. Use when the user asks to split work across worktrees/sessions and orchestrate them to completion, with reasoning set to low.
---

# Worktree Session Orchestrator

Coordinate several isolated implementation sessions while keeping the parent
session as the source of judgment.

Use this skill when the user asks to coordinate multiple implementation
worktrees or Codex sessions through implementation, verification, PR creation,
review fixes, CI fixes, and readiness maintenance.

Do not use this skill for a single narrow local edit unless the user explicitly
wants multi-session orchestration.

## Purpose and Parent Responsibility

The parent session owns judgment, integration, and lane completion. Separate
Codex child sessions own isolated implementation lanes, including creating or
attaching their own worktrees. Do not treat a parent-local subagent as a
substitute for a separate session when the user asks for worktree/session
orchestration.

Treat current source, tests, live PR state, and CI as authoritative. A lane is
not complete because the child says it is complete, because a PR exists, or
because partial checks are green. A lane is complete only when the parent has
verified that the implementation satisfies the original acceptance criteria,
uses the correct production owner, removes or narrows the intended old path,
and has no material review/CI blockers.

If the user explicitly asks to use a goal, create one before orchestration and
complete it only after every lane is ready, blocked with evidence, intentionally
deferred, or merged if merge was later requested.

The parent should:

- stay in the canonical repository unless inspecting child worktrees
- coordinate separate Codex sessions for implementation lanes
- avoid parent-checkout implementation unless repairing or integrating a lane
- preserve user changes and never force-push unless the batch owns the branch
  and no safer option exists
- keep a compact status table after each monitoring pass
- keep reasoning effort low for parent and children unless the user overrides it

Use subagents only when they materially help with source-first audit, scope
drift review, fake deletion/orphan helper detection, CI log analysis, or risky
integration review. Subagents are advisory; the parent makes the final decision.

## Batch Contract

Read the user's stated goal and the current repository evidence needed to define
the batch. For each lane, record:

- lane name
- branch name
- worktree path
- child thread id or pending worktree id
- acceptance criteria
- expected and out-of-scope surfaces
- verification commands
- PR readiness contract

If the user names an existing session or active PR that overlaps a planned
lane, prefer the user-led session unless the user explicitly says otherwise.
Tell duplicates to stop and do not count duplicate PRs as deliverables.

Prefer Codex thread/worktree tools. If manual worktree creation is needed,
inspect existing worktrees and create a unique clean worktree from the default
branch. Never remove or reuse a dirty path unless it clearly belongs to the
lane.

## Child Session Prompt

Each child prompt must be paste-ready and include:

- repository, exact worktree path, and branch name
- instruction to work only inside that worktree
- task, acceptance criteria, and out-of-scope surfaces
- warning against fake deletion, disconnected helper folders, unused rewrites,
  renamed legacy paths, and compatibility shims that preserve the old route
- expected production wiring and tests
- verification commands
- commit, push, and ready PR requirement
- instruction not to merge
- instruction to use reasoning effort low

Use concrete wording like:

```text
Work only in <worktree-path> on branch <branch-name>. Do not edit the parent
checkout. Implement the task end to end, wire it into production call paths, and
delete obsolete paths rather than renaming or parking them. Do not create a new
folder with replacement functions unless existing production callers are
actually rewired to it and tests prove that path. Commit, push, and open a
ready PR when complete. Do not merge. Use reasoning effort low.
```

## Monitoring and Maintenance Heartbeat

For any multi-worktree batch expected to take more than one parent turn, create
a heartbeat automation attached to the current thread unless the user explicitly
declines. Default cadence: every 5 minutes.

The heartbeat prompt must be self-contained and include:

- target repository
- lane names, child thread ids, worktree paths, and PR numbers if known
- current acceptance criteria
- instruction to inspect live PR state, CI/checks, review threads, and child
  thread progress
- instruction to send corrective prompts to child sessions when needed
- instruction not to merge unless the user explicitly asks in a later turn
- instruction to report only material changes or blockers

Delete the heartbeat when all lanes are ready and no longer need maintenance,
merged, intentionally deferred, or blocked in a way that requires user input.

On each monitoring pass:

- read child thread status, worktree status, PR state, CI, and review threads
- detect duplicate, drifting, stalled, or dirty lanes
- send corrective prompts for blockers, scope drift, missing verification, or
  architecture gaps
- verify any claimed completion before accepting it
- keep a compact status table

Use this parent status shape:

```text
- <lane>: <state>, branch <branch>, PR <url or none>, blocker <none/details>
```

## Completion and PR Readiness Review

When a child says done or opens a PR, perform a source-first integrity review
before accepting it.

Verify:

- branch contains the expected commits
- diff matches the lane scope
- production callers are wired to the new/changed behavior
- old path is actually deleted or unreachable when deletion was the goal
- no name-only deletion, parallel replacement folder, orphan helper, or renamed
  legacy path was used to fake progress
- no brittle keyword/regex/includes semantic bypass was introduced for
  freeform human decisions unless the repository contract explicitly allows it
- tests cover the production entrypoint shape, not only precomputed lower-level
  inputs, AST/string checks, or `vi.fn()` delegation tests
- targeted verification commands pass or failures are explained
- PR is non-draft and has a useful readiness manifest

Use repo-appropriate commands to inspect PR diff, changed files, old/new
symbols, targeted tests, typecheck, boundary lint, live checks, and review
threads. Do not trust stale child summaries or PR bodies when source evidence
conflicts.

If GitHub review comments or CI failures are material, send the relevant child
session a concise correction while it still owns the branch. Include the parent
judgment, not only the raw GitHub comment.

Send a second corrective prompt when a first fix is directionally correct but
still misses the architecture contract. Examples:

- production wiring exists only in examples/tests, not real callers
- test proves a property name or mock call but not the caller path
- CI is green but the old mechanism is still reachable
- a new helper is untracked, uncommitted, or not imported by production code
- review comments are fixed locally but not pushed

If the child is unavailable or the user asks the parent to finish, repair the PR
branch directly in its worktree: inspect evidence first, keep the fix on the
same branch, run the smallest meaningful verification, push, and refresh PR
state.

Do not treat GitHub thread resolution as required unless the user asks for
thread cleanup. Code and CI correctness matter first.

This skill stops at ready PR creation and PR readiness maintenance. Do not merge
PRs as part of this skill. If the user later asks to merge, switch to the
repository merge workflow or a dedicated merge skill, refresh live PR state and
branch rules, and do not let stale heartbeat status substitute for merge
readiness.

Final report: lane status, PR links, verification, blockers, and cleanup
caveats. If a goal was used, mark it complete only after those facts are true.
