---
name: worktree-session-orchestrator
description: Coordinate multiple Codex project worktree-thread implementation lanes through source-first implementation, monitoring, verification, PR readiness, review/CI fixes, and maintenance. Use when the user asks to split work across worktrees/sessions and orchestrate them to completion, with reasoning set to low.
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

The parent session owns judgment, integration, child-session creation, and lane
completion. Separate Codex child sessions own implementation inside the Codex
project worktree assigned to that thread. Do not ask a child to create another
`git worktree`, and do not treat a parent-local subagent as a substitute for a
separate session when the user asks for worktree/session orchestration.

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
- coordinate separate Codex project worktree threads for implementation lanes
- avoid parent-checkout implementation unless repairing or integrating a lane
- preserve user changes and never force-push unless the batch owns the branch
  and no safer option exists
- keep a compact status table after each monitoring pass
- keep reasoning effort low for parent and children unless the user overrides it

Use subagents only when they materially help with source-first audit, scope
drift review, fake deletion/orphan helper detection, CI log analysis, or risky
integration review. Subagents are advisory; the parent makes the final decision.

## Codex Worktree Thread Setup

Use Codex app project worktree threads as the normal execution path.

Before creating child sessions:

- load thread tools with `tool_search` if `list_projects`, `create_thread`,
  `read_thread`, `send_message_to_thread`, or `automation_update` are not
  visible
- call `list_projects` and select the saved Codex project for the target
  repository
- stop and report a blocker if the repository is not available as a saved Codex
  project; do not silently fall back to a parent-local subagent
- create each child with `create_thread` using `target.type = "project"`,
  the selected `projectId`, `target.environment.type = "worktree"`, and
  `thinking = "low"`
- let Codex assign the worktree checkout path; do not require a known worktree
  path before the thread is created
- record `threadId` when returned, or `pendingWorktreeId` while worktree setup
  is still pending

The parent creates the worktree-backed child thread. The child reports its
assigned checkout with `pwd` at the start of its first turn, then works only
inside that checkout. Use manual `git worktree add` only if the user explicitly
approves a fallback after Codex project worktree-thread creation is unavailable.

## Batch Contract

Read the user's stated goal and the current repository evidence needed to define
the batch. For each lane, record:

- lane name
- branch name
- child thread id or pending worktree id
- assigned worktree path, initially unknown until the child reports `pwd`
- acceptance criteria
- expected and out-of-scope surfaces
- verification commands
- PR readiness contract
- CI trigger contract, including whether heavy CI is automatic or gated by a
  label, comment, merge queue, or parent instruction

If the user names an existing session or active PR that overlaps a planned
lane, prefer the user-led session unless the user explicitly says otherwise.
Tell duplicates to stop and do not count duplicate PRs as deliverables.

Do not preselect a local worktree path for Codex app worktree threads. If a
child starts in a detached or default branch state, instruct it to create or
switch to the lane branch inside its assigned checkout before editing.

## Child Session Prompt

Each child prompt must be paste-ready and include:

- repository, branch name, and the fact that Codex has already assigned the
  worktree checkout for this thread
- instruction to report `pwd`, `git branch --show-current`, and
  `git status --short` before editing
- instruction to work only inside the assigned checkout and not create another
  worktree
- task, acceptance criteria, and out-of-scope surfaces
- warning against fake deletion, disconnected helper folders, unused rewrites,
  renamed legacy paths, and compatibility shims that preserve the old route
- expected production wiring and tests
- verification commands
- commit, push, and ready PR requirement
- instruction not to run full-repository or expensive CI unless the parent
  explicitly marks the lane `ready-for-ci` or the repository contract says the
  check is required immediately
- instruction not to proactively rebase only because `origin/main` advanced;
  rebase only when the parent marks `base-refresh-needed`, GitHub reports a
  stale-base merge blocker, or current CI/review evidence proves the branch
  cannot become mergeable without a base refresh
- instruction not to merge
- instruction to use reasoning effort low

Use concrete wording like:

```text
Codex has already created or assigned your project worktree for this thread.
Work only inside this assigned checkout. Do not create another git worktree and
do not edit the parent checkout.

First report:
- pwd
- git branch --show-current
- git status --short

Then create or switch to branch <branch-name> inside this assigned checkout if
needed. Implement the task end to end, wire it into production call paths, and
delete obsolete paths rather than renaming or parking them. Do not create a new
folder with replacement functions unless existing production callers are
actually rewired to it and tests prove that path. Commit, push, and open a
ready PR when complete. After opening the PR, stay in maintenance mode: fix
actionable review comments and real latest-head CI failures on the same branch,
but do not run full-repository CI or proactively rebase unless the parent marks
the lane `ready-for-ci` or `base-refresh-needed`. Do not merge. Use reasoning
effort low.
```

## Monitoring and Maintenance Heartbeat

For any multi-worktree batch expected to take more than one parent turn, create
a heartbeat automation attached to the current thread unless the user explicitly
declines. Use `automation_update` with `kind = "heartbeat"`,
`destination = "thread"`, `status = "ACTIVE"`, and a 5-minute cadence. Default
cadence: every 5 minutes.

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
- distinguish review readiness from CI readiness when the repository supports
  gated heavy CI: a ready PR may exist to trigger review while remaining outside
  expensive CI until the parent marks it `ready-for-ci`, applies the required
  label/comment, or queues it for merge
- avoid asking every lane to rebase after each main advancement; mark
  `base-refresh-needed` only when GitHub/rulesets require it, CI proves stale
  base is the blocker, a merged dependency changed the same contract, or the
  lane is one of the next merge candidates
- verify any claimed completion before accepting it
- keep a compact status table

Use these lane states when a batch needs CI-cost control:

- `assigned`: child thread/worktree exists and scope is recorded.
- `implementing`: child is editing inside the assigned checkout.
- `local-verified`: child committed and ran the lane's scoped verification.
- `ready-pr-open`: non-draft PR exists for review; heavy CI may still be gated.
- `parent-reviewed`: parent source review passed for scope, wiring, deletion,
  and tests.
- `ready-for-ci`: parent has chosen this lane for expensive CI, label/comment
  trigger, or merge-queue entry.
- `ci-red-owned`: latest-head CI or review has a material failure owned by this
  lane.
- `merge-candidate`: parent-reviewed with required checks/reviews satisfied or
  deliberately ready for the repository merge workflow.
- `base-refresh-needed`: rebase is required by ruleset, proven stale-base
  failure, merged dependency, or current merge-candidate ordering.
- `blocked`: user or external decision required.

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
- if heavy CI is gated, the PR body or parent status clearly distinguishes
  `ready-pr-open`, `parent-reviewed`, `ready-for-ci`, and `merge-candidate` so a
  review-ready PR is not mistaken for a merge-ready PR

Use repo-appropriate commands to inspect PR diff, changed files, old/new
symbols, targeted tests, typecheck, boundary lint, live checks, and review
threads. Do not trust stale child summaries or PR bodies when source evidence
conflicts.

If GitHub review comments or CI failures are material, send the relevant child
session a concise correction while it still owns the branch. Include the parent
judgment, not only the raw GitHub comment. For repositories with gated heavy CI,
do not treat missing expensive checks on `ready-pr-open` lanes as a child
blocker until the parent has marked the lane `ready-for-ci` or queued it for
merge.

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
