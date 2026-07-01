# Worktree Session Orchestrator

Codex skill for coordinating multi-lane implementation work across Codex app
project worktree threads.

## Install

Copy this directory into your Codex skills directory:

```sh
mkdir -p ~/.codex/skills
cp -R worktree-session-orchestrator ~/.codex/skills/
```

Then restart Codex or reload skills so `worktree-session-orchestrator` is
available.

## Scope

Use this skill when a parent Codex session needs to split repository work into
separate Codex worktree-backed child threads, monitor implementation lanes,
verify source and CI evidence, and maintain PR readiness.

It is intentionally a coordination skill. It does not merge PRs by default.
